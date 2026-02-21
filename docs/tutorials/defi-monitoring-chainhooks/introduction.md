# Introduction to Chainhooks

Chainhooks is an event-driven framework built by Hiro that lets you react to on-chain activity on Stacks and Bitcoin in real time. Instead of polling the blockchain repeatedly, you define **predicates** that describe the events you care about, and Chainhooks delivers matching transactions directly to your webhook.

## Why Chainhooks Instead of Polling?

| Approach | Latency | Resource Usage | Complexity |
|---|---|---|---|
| Polling (RPC) | High (interval-based) | High (constant requests) | Low |
| Chainhooks | Low (event-driven) | Low (push-based) | Low |

## How Chainhooks Works

1. You define a **predicate** — a JSON rule describing which transactions to watch
2. The Chainhooks engine scans every new block
3. When a match is found, the engine POSTs a payload to your webhook URL
4. Your server processes the payload and acts on it

## Types of Predicates

### `contract-call`
Triggers when a specific contract function is called.
```json
{
  "type": "contract-call",
  "contract_identifier": "SP2C2YFP12AJZB4MABJBAJ55XECVS7E4PMMZ89YZR.arkadiko-swap-v2-1",
  "method": "swap-x-for-y"
}
```

### `print-event`
Triggers when a contract emits a `print` event matching a topic.
```json
{
  "type": "print-event",
  "contract_identifier": "SP1Y5YSTAHZ88XYK1VPDH24GY0HPX5J4JECTMY4A1.univ2-core",
  "topic": "swap"
}
```

### `stx-event`
Triggers on STX transfers, useful for whale tracking.
```json
{
  "type": "stx-event",
  "actions": ["transfer"]
}
```

## Deployment Options

- **Hiro Platform (managed)** — Register predicates via the [Hiro Platform dashboard](https://platform.hiro.so). No infrastructure required.
- **Self-hosted** — Run `chainhooks` locally using the open-source CLI.

In this tutorial we use the **Hiro Platform** approach.
