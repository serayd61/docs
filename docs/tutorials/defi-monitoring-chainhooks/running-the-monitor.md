# Running the Monitor

## Local Development with ngrok
```bash
# Terminal 1
npm run dev

# Terminal 2
ngrok http 3000
```

Update `.env` with your ngrok URL, then register predicates:
```bash
npx ts-node src/register-predicates.ts
```

## Verify Registration
```bash
curl -H "x-hiro-api-key: $HIRO_API_KEY" https://api.hiro.so/chainhooks/v1
```

## Test with a Manual Payload
```bash
curl -X POST http://localhost:3000/webhook \
  -H "Content-Type: application/json" \
  -d '{
    "apply": [{
      "block_identifier": { "index": 155432, "hash": "0xabc123" },
      "timestamp": 1710000000,
      "transactions": [{
        "transaction_identifier": { "hash": "0xtx123" },
        "operations": [{
          "operation_identifier": { "index": 0 },
          "type": "credit",
          "status": "success",
          "account": { "address": "SP2J6ZY48GV1EZ5V2V5RB9MP66SW86PYKKNRV9EJ7" },
          "amount": { "value": 150000000000, "currency": { "symbol": "STX", "decimals": 6 } }
        }],
        "metadata": { "success": true, "sender": "SP1HTBVD3JG9C05J7HBJTHGR0GGW7KXW28M5JS8QE", "fee": 1000, "kind": { "type": "ContractCall" } }
      }],
      "metadata": {}
    }],
    "rollback": [],
    "chainhook": { "uuid": "whale-alert-stx-v1", "is_streaming_blocks": true }
  }'
```

Expected output:
```
🐋 WHALE | 150,000 STX | SP1HTBVD... → SP2J6ZY4... | Block 155432
```

## Delete a Predicate
```bash
curl -X DELETE \
  -H "x-hiro-api-key: $HIRO_API_KEY" \
  https://api.hiro.so/chainhooks/v1/dex-swap-velar-v1
```

## Next Steps

- Persist data in PostgreSQL or Redis
- Push live events to a frontend dashboard via WebSocket
- Send Telegram/Discord alerts to your community
- Detect price impact from large swaps

## Resources

- [Chainhooks Documentation](https://docs.hiro.so/chainhooks)
- [Hiro Platform](https://platform.hiro.so)
- [Reference Implementation](https://github.com/serayd61/stacks-defi-sentinel)
