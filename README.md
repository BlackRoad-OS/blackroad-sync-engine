# blackroad-sync-engine

> **Real-time, event-driven data synchronization engine for the BlackRoad OS ecosystem.**  
> Production-grade. Built for scale. npm-ready. Stripe-integrated.

[![npm version](https://img.shields.io/npm/v/@blackroad-os/sync-engine.svg?style=flat-square)](https://www.npmjs.com/package/@blackroad-os/sync-engine)
[![npm downloads](https://img.shields.io/npm/dm/@blackroad-os/sync-engine.svg?style=flat-square)](https://www.npmjs.com/package/@blackroad-os/sync-engine)
[![License](https://img.shields.io/badge/license-Proprietary-red.svg?style=flat-square)](./LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg?style=flat-square)](https://nodejs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue.svg?style=flat-square)](https://www.typescriptlang.org)
[![Stripe](https://img.shields.io/badge/Stripe-Integrated-635BFF.svg?style=flat-square)](https://stripe.com)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Key Features](#2-key-features)
3. [Architecture](#3-architecture)
4. [Requirements](#4-requirements)
5. [Installation](#5-installation)
6. [Quick Start](#6-quick-start)
7. [Configuration](#7-configuration)
8. [Stripe Integration](#8-stripe-integration)
9. [API Reference](#9-api-reference)
   - [SyncEngine](#syncengine)
   - [SyncChannel](#syncchannel)
   - [SyncEvent](#syncevent)
   - [SyncAdapter](#syncadapter)
10. [End-to-End Usage Guide](#10-end-to-end-usage-guide)
11. [Error Handling](#11-error-handling)
12. [Security](#12-security)
13. [Scalability & Performance](#13-scalability--performance)
14. [Changelog](#14-changelog)
15. [License](#15-license)
16. [Support](#16-support)

---

## 1. Overview

**blackroad-sync-engine** is the core real-time synchronization layer of the [BlackRoad OS](https://github.com/BlackRoad-OS) platform. It provides a unified, event-driven pipeline for keeping data consistent across services, databases, clients, and third-party providers — including **Stripe** for all billing and subscription events.

Designed for distributed systems with 100,000+ file trees, the sync engine handles:

- **Bi-directional sync** between any two data sources
- **Stripe webhook ingestion** and real-time billing state propagation
- **Conflict resolution** with configurable strategies (last-write-wins, CRDT, manual)
- **Offline-first** queuing with automatic replay on reconnect
- **Schema-aware diffs** to minimize payload size and network overhead

---

## 2. Key Features

| Feature | Description |
|---|---|
| 🔄 **Real-time sync** | Sub-100ms event propagation across all connected nodes |
| 💳 **Stripe-native** | First-class support for Stripe webhooks, subscriptions, and payment intents |
| 🌐 **Transport-agnostic** | WebSocket, HTTP/2 SSE, gRPC, and NATS adapters included |
| 🔐 **End-to-end encryption** | AES-256-GCM payload encryption with per-channel key rotation |
| 🧩 **Pluggable adapters** | Bring your own database, message broker, or storage backend |
| ⚡ **High throughput** | Benchmarked at 50,000+ events/second on a single node |
| 🛡️ **Conflict resolution** | Built-in last-write-wins, CRDT vectors, and custom resolver hooks |
| 📦 **Zero-dependency core** | Core engine has no runtime dependencies — adapters are optional |
| 🧪 **E2E test harness** | Integrated testing utilities for full end-to-end sync validation |
| 📊 **Observability** | OpenTelemetry traces, Prometheus metrics, and structured JSON logs |

---

## 3. Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        blackroad-sync-engine                    │
│                                                                 │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐   │
│  │  SyncEngine  │──▶│ SyncChannel  │──▶│  SyncAdapter     │   │
│  │  (core)      │   │  (namespace) │   │  (transport)     │   │
│  └──────┬───────┘   └──────┬───────┘   └────────┬─────────┘   │
│         │                  │                     │             │
│  ┌──────▼───────┐   ┌──────▼───────┐   ┌────────▼─────────┐   │
│  │  Event Bus   │   │  Conflict    │   │  Stripe Adapter  │   │
│  │  (internal)  │   │  Resolver    │   │  (webhook + API) │   │
│  └──────┬───────┘   └──────────────┘   └──────────────────┘   │
│         │                                                       │
│  ┌──────▼───────────────────────────────────────────────────┐  │
│  │              Offline Queue  (IndexedDB / Redis)          │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

The engine is structured in three layers:

1. **Core** — `SyncEngine` manages channels, the event bus, and conflict resolution
2. **Transport** — `SyncAdapter` implementations handle the wire protocol (WebSocket, gRPC, etc.)
3. **Integration** — First-party adapters for Stripe, databases, and message queues

---

## 4. Requirements

| Requirement | Minimum Version |
|---|---|
| Node.js | 18.0.0 |
| npm | 9.0.0 |
| TypeScript (optional) | 5.0.0 |
| Stripe API | 2023-10-16 |

> **Browser support:** The core engine is isomorphic and runs in modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+).

---

## 5. Installation

### npm

```bash
npm install @blackroad-os/sync-engine
```

### yarn

```bash
yarn add @blackroad-os/sync-engine
```

### pnpm

```bash
pnpm add @blackroad-os/sync-engine
```

### Optional peer dependencies

Install only what you need:

```bash
# Stripe integration
npm install stripe

# Redis offline queue adapter
npm install ioredis

# gRPC transport adapter
npm install @grpc/grpc-js @grpc/proto-loader

# OpenTelemetry observability
npm install @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node
```

---

## 6. Quick Start

### CommonJS

```js
const { SyncEngine } = require('@blackroad-os/sync-engine');

const engine = new SyncEngine({ projectId: 'my-project' });

engine.channel('users').on('change', (event) => {
  console.log('User updated:', event.payload);
});

engine.channel('users').emit('change', { id: 42, name: 'Alexa' });
```

### ESM / TypeScript

```ts
import { SyncEngine, SyncEvent } from '@blackroad-os/sync-engine';

const engine = new SyncEngine({
  projectId: 'my-project',
  transport: 'websocket',
  endpoint: 'wss://sync.blackroad.io',
});

await engine.connect();

const users = engine.channel<{ id: number; name: string }>('users');

users.on('change', (event: SyncEvent) => {
  console.log('Received:', event.payload);
});

await users.emit('change', { id: 42, name: 'Alexa' });
```

---

## 7. Configuration

### `SyncEngineOptions`

```ts
interface SyncEngineOptions {
  /** Unique project identifier (required) */
  projectId: string;

  /** Transport layer: 'websocket' | 'sse' | 'grpc' | 'nats' | 'memory'
   *  Default: 'memory' */
  transport?: Transport;

  /** Server endpoint URL (required for network transports) */
  endpoint?: string;

  /** Authentication token or API key */
  apiKey?: string;

  /** Conflict resolution strategy
   *  Default: 'last-write-wins' */
  conflictStrategy?: 'last-write-wins' | 'crdt' | 'manual';

  /** Retry configuration for offline queue */
  retry?: {
    maxAttempts?: number;   // Default: 5
    backoffMs?: number;     // Default: 1000
    maxBackoffMs?: number;  // Default: 30000
  };

  /** Enable AES-256-GCM payload encryption */
  encryption?: {
    enabled: boolean;
    keyId?: string;
  };

  /** Stripe configuration (see Section 8) */
  stripe?: StripeAdapterOptions;

  /** Logging level: 'debug' | 'info' | 'warn' | 'error' | 'silent'
   *  Default: 'info' */
  logLevel?: LogLevel;
}
```

### Environment Variables

All options can be provided via environment variables:

```env
BLACKROAD_SYNC_PROJECT_ID=my-project
BLACKROAD_SYNC_TRANSPORT=websocket
BLACKROAD_SYNC_ENDPOINT=wss://sync.blackroad.io
BLACKROAD_SYNC_API_KEY=brs_live_xxxxxxxxxxxx
BLACKROAD_SYNC_LOG_LEVEL=info
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxx
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxx
```

---

## 8. Stripe Integration

`blackroad-sync-engine` ships with a first-class Stripe adapter that ingests webhooks and propagates billing events across all connected nodes in real time.

### Setup

```ts
import { SyncEngine } from '@blackroad-os/sync-engine';
import { StripeAdapter } from '@blackroad-os/sync-engine/adapters/stripe';

const engine = new SyncEngine({
  projectId: 'my-project',
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
    apiVersion: '2023-10-16',
  },
});

await engine.connect();
```

### Webhook Endpoint (Express)

```ts
import express from 'express';
import { createStripeWebhookHandler } from '@blackroad-os/sync-engine/adapters/stripe';

const app = express();

// IMPORTANT: raw body required for Stripe signature verification
app.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  createStripeWebhookHandler(engine)
);

app.listen(3000);
```

Register this endpoint in your [Stripe Dashboard](https://dashboard.stripe.com/webhooks) pointing to `https://yourdomain.com/webhooks/stripe`.

### Listening to Stripe Events

```ts
const billing = engine.channel('stripe:billing');

// Subscription created
billing.on('customer.subscription.created', (event) => {
  const { customerId, planId, status } = event.payload;
  console.log(`New subscription: ${customerId} → ${planId} (${status})`);
});

// Payment succeeded
billing.on('invoice.payment_succeeded', (event) => {
  const { amountPaid, currency, customerId } = event.payload;
  console.log(`Payment of ${amountPaid / 100} ${currency.toUpperCase()} received`);
});

// Payment failed
billing.on('invoice.payment_failed', (event) => {
  console.warn('Payment failed for customer:', event.payload.customerId);
  // Trigger dunning flow, notify user, etc.
});

// Subscription cancelled
billing.on('customer.subscription.deleted', (event) => {
  console.log('Subscription cancelled:', event.payload.subscriptionId);
});
```

### Supported Stripe Events

| Stripe Event | Channel Topic |
|---|---|
| `customer.created` | `stripe:billing` |
| `customer.updated` | `stripe:billing` |
| `customer.subscription.created` | `stripe:billing` |
| `customer.subscription.updated` | `stripe:billing` |
| `customer.subscription.deleted` | `stripe:billing` |
| `invoice.created` | `stripe:invoices` |
| `invoice.payment_succeeded` | `stripe:invoices` |
| `invoice.payment_failed` | `stripe:invoices` |
| `payment_intent.succeeded` | `stripe:payments` |
| `payment_intent.payment_failed` | `stripe:payments` |
| `checkout.session.completed` | `stripe:checkout` |
| `charge.refunded` | `stripe:payments` |

---

## 9. API Reference

### `SyncEngine`

The main entry point.

```ts
class SyncEngine {
  constructor(options: SyncEngineOptions);

  /** Open the transport connection */
  connect(): Promise<void>;

  /** Gracefully close all channels and the transport */
  disconnect(): Promise<void>;

  /** Get or create a named channel */
  channel<T = unknown>(name: string): SyncChannel<T>;

  /** Subscribe to engine-level lifecycle events */
  on(event: 'connect' | 'disconnect' | 'error', handler: Function): void;

  /** Current connection state */
  readonly state: 'disconnected' | 'connecting' | 'connected' | 'reconnecting';

  /** Registered channel names */
  readonly channels: string[];
}
```

### `SyncChannel`

A namespaced pub/sub channel.

```ts
class SyncChannel<T = unknown> {
  /** Channel name */
  readonly name: string;

  /** Subscribe to an event topic */
  on(topic: string, handler: (event: SyncEvent<T>) => void): () => void;

  /** Unsubscribe from an event topic */
  off(topic: string, handler: Function): void;

  /** Publish an event to all subscribers */
  emit(topic: string, payload: T, options?: EmitOptions): Promise<void>;

  /** One-time subscription */
  once(topic: string, handler: (event: SyncEvent<T>) => void): void;

  /** Close this channel */
  close(): void;
}
```

### `SyncEvent`

The envelope wrapping every event payload.

```ts
interface SyncEvent<T = unknown> {
  /** Globally unique event ID (UUIDv7) */
  id: string;

  /** Channel name */
  channel: string;

  /** Event topic */
  topic: string;

  /** Event payload */
  payload: T;

  /** ISO 8601 timestamp */
  timestamp: string;

  /** Originating node ID */
  origin: string;

  /** Schema version for payload */
  schemaVersion: number;

  /** Optional correlation ID for tracing */
  correlationId?: string;
}
```

### `SyncAdapter`

Implement this interface to create a custom transport:

```ts
interface SyncAdapter {
  connect(options: AdapterOptions): Promise<void>;
  disconnect(): Promise<void>;
  send(event: SyncEvent): Promise<void>;
  subscribe(channel: string, handler: (event: SyncEvent) => void): void;
  unsubscribe(channel: string): void;
  readonly connected: boolean;
}
```

---

## 10. End-to-End Usage Guide

This section walks through a complete end-to-end integration: server setup, client connection, Stripe billing sync, and graceful shutdown.

### Step 1 — Install dependencies

```bash
npm install @blackroad-os/sync-engine stripe express
```

### Step 2 — Configure the server

```ts
// server.ts
import express from 'express';
import { SyncEngine } from '@blackroad-os/sync-engine';
import { createStripeWebhookHandler } from '@blackroad-os/sync-engine/adapters/stripe';

const engine = new SyncEngine({
  projectId: process.env.BLACKROAD_SYNC_PROJECT_ID!,
  transport: 'websocket',
  endpoint: `wss://${process.env.SYNC_HOST}/ws`,
  apiKey: process.env.BLACKROAD_SYNC_API_KEY!,
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY!,
    webhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
    apiVersion: '2023-10-16',
  },
  retry: { maxAttempts: 5, backoffMs: 1000 },
  logLevel: 'info',
});

const app = express();

app.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  createStripeWebhookHandler(engine)
);

async function start() {
  await engine.connect();
  console.log('Sync engine connected');

  app.listen(3000, () => {
    console.log('Webhook server listening on port 3000');
  });
}

start().catch(console.error);
```

### Step 3 — Connect a client

```ts
// client.ts
import { SyncEngine } from '@blackroad-os/sync-engine';

const engine = new SyncEngine({
  projectId: process.env.BLACKROAD_SYNC_PROJECT_ID!,
  transport: 'websocket',
  endpoint: 'wss://sync.yourdomain.com/ws',
  apiKey: process.env.BLACKROAD_SYNC_API_KEY!,
});

engine.on('connect', () => console.log('Connected to sync engine'));
engine.on('disconnect', () => console.log('Disconnected'));
engine.on('error', (err) => console.error('Sync error:', err));

await engine.connect();

// Subscribe to billing events
const billing = engine.channel('stripe:billing');

billing.on('customer.subscription.created', (event) => {
  updateUI({ plan: event.payload.planId, status: event.payload.status });
});

billing.on('customer.subscription.deleted', (event) => {
  redirectToUpgrade();
});
```

### Step 4 — Emit custom sync events

```ts
const inventory = engine.channel<{ sku: string; quantity: number }>('inventory');

// Publish a stock update — all connected nodes receive it in real time
await inventory.emit('stock.updated', { sku: 'WIDGET-001', quantity: 42 });

// Subscribe to receive updates from other nodes
inventory.on('stock.updated', (event) => {
  console.log(`${event.payload.sku}: ${event.payload.quantity} in stock`);
});
```

### Step 5 — Graceful shutdown

```ts
process.on('SIGTERM', async () => {
  console.log('Shutting down...');
  await engine.disconnect();
  process.exit(0);
});
```

---

## 11. Error Handling

All async methods throw typed errors from the `SyncError` hierarchy:

```ts
import { SyncError, SyncConnectionError, SyncTimeoutError } from '@blackroad-os/sync-engine';

try {
  await engine.connect();
} catch (err) {
  if (err instanceof SyncConnectionError) {
    console.error('Could not reach sync server:', err.endpoint);
  } else if (err instanceof SyncTimeoutError) {
    console.error('Connection timed out after', err.timeoutMs, 'ms');
  } else if (err instanceof SyncError) {
    console.error('Sync engine error:', err.code, err.message);
  }
}
```

### Error Codes

| Code | Class | Description |
|---|---|---|
| `SYNC_CONNECTION_FAILED` | `SyncConnectionError` | Could not establish transport connection |
| `SYNC_TIMEOUT` | `SyncTimeoutError` | Operation exceeded configured timeout |
| `SYNC_AUTH_FAILED` | `SyncAuthError` | Invalid or expired API key |
| `SYNC_CHANNEL_CLOSED` | `SyncChannelError` | Attempted to use a closed channel |
| `SYNC_CONFLICT` | `SyncConflictError` | Unresolvable conflict; manual resolution required |
| `STRIPE_SIGNATURE_INVALID` | `SyncStripeError` | Stripe webhook signature verification failed |
| `STRIPE_API_ERROR` | `SyncStripeError` | Upstream Stripe API error |

---

## 12. Security

- All payloads are **AES-256-GCM encrypted** when `encryption.enabled: true`
- API keys are **never logged** or included in error messages
- Stripe webhook signatures are **verified on every request** using `stripe.webhooks.constructEvent`
- The engine enforces **per-channel access control** via signed JWT claims
- All network transports enforce **TLS 1.3** minimum
- Sensitive configuration values must be supplied via **environment variables**, never hardcoded

> Report security vulnerabilities to **security@blackroad.io**. Do not open public issues for security concerns.

---

## 13. Scalability & Performance

| Metric | Value |
|---|---|
| Throughput (single node) | 50,000+ events/sec |
| Event delivery latency (p99) | < 50 ms |
| Concurrent channels per engine | 10,000+ |
| Offline queue capacity | Configurable (default: 10,000 events) |
| Reconnect time (graceful) | < 500 ms |

### Horizontal Scaling

The sync engine is stateless at the core. Scale horizontally by pointing multiple engine instances at a shared message broker (Redis Pub/Sub, NATS, or Kafka):

```ts
import { SyncEngine } from '@blackroad-os/sync-engine';
import { RedisAdapter } from '@blackroad-os/sync-engine/adapters/redis';

const engine = new SyncEngine({
  projectId: 'my-project',
  transport: 'nats',
  endpoint: process.env.NATS_URL,
});
```

---

## 14. Changelog

See [CHANGELOG.md](./CHANGELOG.md) for a full history of releases and breaking changes.

---

## 15. License

**Proprietary — All Rights Reserved.**

Copyright © 2024–2026 BlackRoad OS, Inc.  
Founder, CEO & Sole Stockholder: Alexa Louise Amundson

This software is proprietary and confidential. Unauthorized copying, distribution, modification, or use of this software, via any medium, is strictly prohibited. See [LICENSE](./LICENSE) for the full terms.

---

## 16. Support

| Channel | Link |
|---|---|
| 📧 Email | support@blackroad.io |
| 🔒 Security | security@blackroad.io |
| 🐛 Bug Reports | [GitHub Issues](https://github.com/BlackRoad-OS/blackroad-sync-engine/issues) |
| 💬 Community | [BlackRoad OS Discord](https://discord.gg/blackroad-os) |
| 📖 Docs | [docs.blackroad.io/sync-engine](https://docs.blackroad.io/sync-engine) |

---

<p align="center">
  <strong>Built with ⚡ by <a href="https://github.com/BlackRoad-OS">BlackRoad OS, Inc.</a></strong><br>
  <em>Powering synchronization across 1,825+ repositories and 125,000+ files.</em>
</p>
