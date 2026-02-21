# Processing Payloads

## Payload Structure
```json
{
  "apply": [
    {
      "block_identifier": { "index": 155432, "hash": "0xabc..." },
      "timestamp": 1710000000,
      "transactions": [ ... ]
    }
  ],
  "rollback": [],
  "chainhook": {
    "uuid": "dex-swap-velar-v1",
    "is_streaming_blocks": true
  }
}
```

- **`apply`** — blocks with transactions that matched your predicate
- **`rollback`** — blocks rolled back due to chain reorg
- **`chainhook.uuid`** — which predicate triggered this payload

## Swap Handler

Create `src/handlers/swap-handler.ts`:
```typescript
export async function swapHandler(payload: any): Promise<void> {
  for (const block of payload.apply) {
    for (const tx of block.transactions) {
      if (!tx.metadata.success) continue

      const events = tx.metadata?.receipt?.events || []
      for (const event of events) {
        const value = event.data?.value
        if (value?.topic !== 'swap') continue

        console.log(
          `🔄 Swap | ${(value.dx / 1_000_000).toFixed(2)} ${value.token_x} → ` +
          `${(value.dy / 1_000_000).toFixed(2)} ${value.token_y} | ` +
          `Block ${block.block_identifier.index}`
        )
      }
    }
  }

  for (const block of payload.rollback) {
    for (const tx of block.transactions) {
      console.warn(`⚠️ Reorg - rolling back: ${tx.transaction_identifier.hash}`)
    }
  }
}
```

## Whale Handler

Create `src/handlers/whale-handler.ts`:
```typescript
const THRESHOLD = parseInt(process.env.WHALE_THRESHOLD_STX || '100000') * 1_000_000

export async function whaleHandler(payload: any): Promise<void> {
  for (const block of payload.apply) {
    for (const tx of block.transactions) {
      if (!tx.metadata.success) continue

      for (const op of tx.operations) {
        if (op.type !== 'credit' || !op.amount) continue

        const amount = Math.abs(op.amount.value)
        if (amount < THRESHOLD) continue

        console.log(
          `🐋 WHALE | ${(amount / 1_000_000).toLocaleString()} STX | ` +
          `${tx.metadata.sender.slice(0, 8)}... → ${op.account.address.slice(0, 8)}... | ` +
          `Block ${block.block_identifier.index}`
        )
      }
    }
  }
}
```

## Liquidity Handler

Create `src/handlers/liquidity-handler.ts`:
```typescript
export async function liquidityHandler(payload: any): Promise<void> {
  for (const block of payload.apply) {
    for (const tx of block.transactions) {
      if (!tx.metadata.success) continue

      const events = tx.metadata?.receipt?.events || []
      for (const event of events) {
        const value = event.data?.value
        if (!value) continue

        if (value.topic === 'mint') {
          console.log(`💧 LP ADD | ${tx.metadata.sender.slice(0, 8)}... | Block ${block.block_identifier.index}`)
        }
        if (value.topic === 'burn') {
          console.log(`💧 LP REMOVE | ${tx.metadata.sender.slice(0, 8)}... | Block ${block.block_identifier.index}`)
        }
      }
    }
  }
}
```

> **Warning:** Always handle the `rollback` array. Ignoring reorgs can lead to stale or incorrect data.
