# Real-Time DeFi Monitoring with Chainhooks

Learn how to build a real-time DeFi monitoring application on Stacks using Chainhooks. This tutorial walks you through setting up event-driven blockchain listeners to track DEX swaps, whale movements, and liquidity events.

## What You'll Build

A production-ready DeFi sentinel application that:

- Monitors DEX swaps across Velar, ALEX, and Arkadiko in real time
- Sends whale alerts for STX transfers over 100,000 STX
- Tracks liquidity pool events and TVL changes
- Exposes a webhook server to receive and process Chainhook payloads

## Prerequisites

- Basic knowledge of TypeScript/Node.js
- Familiarity with Stacks and Clarity smart contracts
- [Hiro Platform account](https://platform.hiro.so) (for managed Chainhooks)
- Node.js v18+ installed

## Tutorial Sections

1. [Introduction to Chainhooks](introduction.md)
2. [Project Setup](project-setup.md)
3. [Creating Predicates](creating-predicates.md)
4. [Processing Payloads](processing-payloads.md)
5. [Running the Monitor](running-the-monitor.md)

## Reference Implementation

A working example of this tutorial is available at:
[https://github.com/serayd61/stacks-defi-sentinel](https://github.com/serayd61/stacks-defi-sentinel)
