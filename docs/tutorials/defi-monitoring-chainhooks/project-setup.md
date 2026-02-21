# Project Setup

## Project Structure
```
defi-sentinel/
├── src/
│   ├── server.ts
│   ├── predicates/
│   │   ├── dex-swaps.ts
│   │   ├── whales.ts
│   │   └── liquidity.ts
│   ├── handlers/
│   │   ├── swap-handler.ts
│   │   ├── whale-handler.ts
│   │   └── liquidity-handler.ts
│   └── types.ts
├── package.json
└── tsconfig.json
```

## Initialize the Project
```bash
mkdir defi-sentinel && cd defi-sentinel
npm init -y
npm install fastify dotenv
npm install -D typescript @types/node ts-node
```

## Environment Variables

Create `.env`:
```bash
HIRO_API_KEY=your_api_key_here
WEBHOOK_URL=https://your-app.example.com
PORT=3000
WHALE_THRESHOLD_STX=100000
```

## Basic Fastify Server

Create `src/server.ts`:
```typescript
import Fastify from 'fastify'
import { swapHandler } from './handlers/swap-handler'
import { whaleHandler } from './handlers/whale-handler'
import { liquidityHandler } from './handlers/liquidity-handler'

const fastify = Fastify({ logger: true })

fastify.get('/health', async () => {
  return { status: 'ok', timestamp: new Date().toISOString() }
})

fastify.post('/webhook', async (request, reply) => {
  const payload = request.body as any
  const uuid = payload.chainhook?.uuid

  try {
    if (uuid?.startsWith('dex-swap')) await swapHandler(payload)
    else if (uuid?.startsWith('whale-alert')) await whaleHandler(payload)
    else if (uuid?.startsWith('liquidity')) await liquidityHandler(payload)
    return reply.status(200).send({ received: true })
  } catch (error) {
    fastify.log.error(error)
    return reply.status(500).send({ error: 'Handler failed' })
  }
})

const start = async () => {
  await fastify.listen({ port: parseInt(process.env.PORT || '3000'), host: '0.0.0.0' })
}

start()
```
