# Creating Predicates

Predicates define what on-chain activity Chainhooks should watch for.

## DEX Swap Predicate

Create `src/predicates/dex-swaps.ts`:
```typescript
const WEBHOOK_URL = process.env.WEBHOOK_URL!
const AUTH_TOKEN = process.env.AUTH_TOKEN || 'defi-sentinel-secret'

const DEX_CONTRACTS = [
  {
    uuid: 'dex-swap-velar-v1',
    name: 'Velar',
    contract: 'SP1Y5YSTAHZ88XYK1VPDH24GY0HPX5J4JECTMY4A1.univ2-router',
    method: 'swap-exact-tokens-for-tokens'
  },
  {
    uuid: 'dex-swap-alex-v1',
    name: 'ALEX',
    contract: 'SP102V8P0F7JX67ARQ77WEA3D3CFB5XW39REDT0AM.amm-swap-pool-v1-1',
    method: 'swap-helper'
  },
  {
    uuid: 'dex-swap-arkadiko-v1',
    name: 'Arkadiko',
    contract: 'SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.arkadiko-swap-v2-1',
    method: 'swap-x-for-y'
  }
]

export function buildDexPredicates() {
  return DEX_CONTRACTS.map(dex => ({
    uuid: dex.uuid,
    name: `DeFi Sentinel - ${dex.name} Swap Monitor`,
    version: 1,
    chain: 'stacks',
    networks: {
      mainnet: {
        start_block: 150000,
        predicate: {
          scope: 'contract_call',
          contract_identifier: dex.contract,
          method: dex.method
        },
        action: {
          http_post: {
            url: `${WEBHOOK_URL}/webhook`,
            authorization_header: `Bearer ${AUTH_TOKEN}`
          }
        }
      }
    }
  }))
}
```

## Whale Alert Predicate

Create `src/predicates/whales.ts`:
```typescript
export function buildWhalePredicates() {
  return [{
    uuid: 'whale-alert-stx-v1',
    name: 'DeFi Sentinel - STX Whale Alert',
    version: 1,
    chain: 'stacks',
    networks: {
      mainnet: {
        start_block: 150000,
        predicate: { scope: 'stx_event', actions: ['transfer'] },
        action: {
          http_post: {
            url: `${process.env.WEBHOOK_URL}/webhook`,
            authorization_header: `Bearer ${process.env.AUTH_TOKEN}`
          }
        }
      }
    }
  }]
}
```

## Register Predicates

Create `src/register-predicates.ts`:
```typescript
import { buildDexPredicates } from './predicates/dex-swaps'
import { buildWhalePredicates } from './predicates/whales'

async function registerPredicates() {
  const allPredicates = [...buildDexPredicates(), ...buildWhalePredicates()]

  for (const predicate of allPredicates) {
    const response = await fetch('https://api.hiro.so/chainhooks/v1', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-hiro-api-key': process.env.HIRO_API_KEY!
      },
      body: JSON.stringify(predicate)
    })

    if (!response.ok) {
      console.error(`Failed: ${predicate.uuid}`, await response.text())
      continue
    }
    console.log(`✅ Registered: ${predicate.uuid}`)
  }
}

registerPredicates().catch(console.error)
```

Run:
```bash
npx ts-node src/register-predicates.ts
```
