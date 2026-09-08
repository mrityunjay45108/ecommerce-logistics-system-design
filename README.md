# 🚀 Enterprise E-Commerce & Logistics System Design

[![System Design](https://img.shields.io/badge/Architecture-Distributed%20Microservices-blue.svg)](https://github.com/mrityunjay45108/ecommerce-logistics-system-design)
[![Pattern](https://img.shields.io/badge/Pattern-Transactional%20Outbox-purple.svg)](https://github.com/mrityunjay45108/ecommerce-logistics-system-design)
[![Message Broker](https://img.shields.io/badge/Broker-Apache%20Kafka-orange.svg)](https://github.com/mrityunjay45108/ecommerce-logistics-system-design)
[![Tech Stack](https://img.shields.io/badge/Tech-NestJS%20%7C%20React%20%7C%20Prisma%20%7C%20Redis-red.svg)](https://github.com/mrityunjay45108/ecommerce-logistics-system-design)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> A production-grade **E-Commerce & Logistics Architecture** showcasing the **Transactional Outbox Pattern**, **Idempotent Kafka Consumer**, **Decoupled Bounded Contexts (Order ➔ Shipment)**, and a **Pluggable Courier Strategy Adapter (Shiprocket / Delhivery)**.

---

## 📌 Architecture Highlights

### 1️⃣ Transactional Outbox Pattern (Zero Dual-Write Loss)
- **Problem**: Writing to PostgreSQL and publishing to Apache Kafka in a standard API handler risks data inconsistency if the network fails or Kafka is unreachable.
- **Solution**: The `Order`, `Shipment Intent`, and `Outbox Event` records are committed together in **one atomic ACID PostgreSQL transaction**.
- **CDC / Poller**: An Outbox Poller (Debezium CDC / Worker) streams unread events to Kafka, guaranteeing **At-Least-Once Delivery**.

```
┌─────────────────────────────────────────────────────────────┐
│                   E-COMMERCE SERVICE                        │
│             [POST /api/v1/orders/checkout]                  │
│                            │                                │
│                            ▼                                │
│          ┌───────────────────────────────────┐              │
│          │    DB TRANSACTION (ACID Commit)   │              │
│          │  📦 Order                         │              │
│          │  📄 Shipment Intent               │              │
│          │  📤 Outbox Event (UNPROCESSED)    │              │
│          └─────────────────┬─────────────────┘              │
└────────────────────────────┼────────────────────────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  OUTBOX POLLER  │ (Debezium CDC / Poller)
                    └────────┬────────┘
                             │
                             ▼
        ╔═══════════════════════════════════════════╗
        ║       APACHE KAFKA MESSAGE BROKER         ║
        ║        Topic: courier.shipment.events     ║
        ╚════════════════════╤══════════════════════╝
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    COURIER SERVICE                          │
│        Kafka Consumer (courier-fulfillment-group)           │
│                            │                                │
│                            ▼                                │
│          ┌───────────────────────────────────┐              │
│          │    INBOX / IDEMPOTENCY LAYER      │              │
│          │    (KafkaProcessedEvent Table)    │              │
│          └─────────────────┬─────────────────┘              │
│                            │                                │
│                            ▼                                │
│          ┌───────────────────────────────────┐              │
│          │  COURIER ADAPTER (Strategy)       │              │
│          │  [Shiprocket] [Delhivery] [Other] │              │
│          └─────────────────┬─────────────────┘              │
│                            │                                │
│                            ▼                                │
│          ┌───────────────────────────────────┐              │
│          │     TRACKING STATE MACHINE        │              │
│          │  PICKED_UP ➔ IN_TRANSIT ➔         │              │
│          │  OUT_FOR_DELIVERY ➔ DELIVERED     │              │
│          └───────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

---

### 2️⃣ Decoupled Bounded Contexts: Order ➔ Shipment

```
          E-COMMERCE DOMAIN                      LOGISTICS DOMAIN
       ┌─────────────────────┐               ┌─────────────────────┐
       │     📦 ORDER        │               │    🚚 SHIPMENT      │
       ├─────────────────────┤               ├─────────────────────┤
       │ 🔑 Order ID         │               │ 🔑 Shipment ID      │
       │ 📦 Items            │               │ 🏷️ AWB Number       │
       │ 💰 Amount           │               │ 📡 Tracking Status  │
       │ 💳 Payment          │               │ 🚚 Courier Partner  │
       │ 👤 Customer         │               │ ⚖️ Weight           │
       │ 📍 Address          │               │ 📐 Dimensions       │
       └──────────┬──────────┘               │ 🔄 Status           │
                  │                          └──────────▲──────────┘
                  │                                     │
                  └──────────── ⚡ creates ──────────────┘
                       (OrderPaidEvent ➔ BullMQ)
```

- **1 : N Fulfillment**: A single Order can spawn multiple Shipments if items originate from different fulfillment centers.
- **Courier API Resilience**: Courier API latencies or vendor downtime never block customer checkout.
- **Two-Way Webhook Sync**: Real-time carrier webhooks update the Shipment tracking status, which dynamically transitions the Order state to `DELIVERED`.

---

### 3️⃣ End-to-End Technology Stack

| Layer | Technologies | Key Responsibility |
| :--- | :--- | :--- |
| **Frontend** | React 18, Vite, Tailwind CSS, TanStack Query | Client SPA, optimistic UI updates, cart state |
| **API Gateway** | NestJS (TypeScript) | Rate limiting (Throttler), ValidationPipe, JWT Guards |
| **Persistence** | PostgreSQL + Prisma ORM | ACID compliance, type-safe migrations, strict foreign keys |
| **Message Broker** | Apache Kafka | Event streaming (`courier.shipment.events`) |
| **In-Memory & Cache** | Redis | Session state, distributed locking (Redlock), BullMQ tasks |
| **Carrier Integrations** | Courier Adapter Strategy | Pluggable Shiprocket, Delhivery, Bluedart APIs |

---

## 🖥️ Interactive Architecture Studio

This repository includes a single-file, interactive studio (`index.html`) ready to view in any browser:
- 📸 **Retina 2x Export**: One-click PNG download for LinkedIn presentations and system design reviews.
- 📋 **LinkedIn Post Copy**: Formatted technical breakdown ready to post.
- 📐 **Preset Aspect Ratios**: Full width, 4:5 mobile portrait, and 16:9 landscape.
- 🗂️ **Interactive Tabs**:
  - `Outbox & Courier Flow`
  - `Order ➔ Shipment`
  - `NestJS Stack Flow`
  - `Prisma Models`
  - `Request Lifecycle`

### How to Open:
```bash
# Windows (cmd or PowerShell)
start index.html
start day4-excalidraw.html
start webhook-reliability-excalidraw.html
start day6-idempotency-excalidraw.html

# macOS
open index.html
open day4-excalidraw.html
open webhook-reliability-excalidraw.html
open day6-idempotency-excalidraw.html

# Linux
xdg-open index.html
xdg-open day4-excalidraw.html
xdg-open webhook-reliability-excalidraw.html
xdg-open day6-idempotency-excalidraw.html
```

---

## 🎨 Day 4 Excalidraw Edition (`day4-excalidraw.html`)
Includes an authentic hand-drawn Excalidraw aesthetic figure designed specifically for **LinkedIn Day 4/30 Architecture Series**:
- Hand-drawn Virgil/Kalam typography & rough sketch shapes.
- Complete Order to Courier Logistics pipeline (E-Commerce ➔ Kafka ➔ Inbox Check ➔ State Machine ➔ Courier Partners).
- Light / Dark Excalidraw theme toggle.
- 2.5x Retina PNG export button.
- 1-click Day 4 LinkedIn caption copy.

---

## ⚡ Webhook Reliability Architecture (`webhook-reliability-excalidraw.html`)
An authentic hand-drawn Excalidraw architectural diagram based on real-world fintech and logistics webhook ingestion systems:
- **Core 6-Stage Webhook Pipeline**:
  1. **Verify Signature**: HMAC-SHA256 hash matching, timing-attack safe comparison, timestamp skew checks.
  2. **Validate & Parse**: JSON schema validation, payload sanitization, strict type checks.
  3. **Idempotency & Deduplication**: Redis distributed locks + PostgreSQL unique event ID checks to prevent duplicate processing on retries.
  4. **State Machine & Event Reconciliation**: Strict monotonic state transitions (e.g. prevent `DELIVERED` ➔ `OUT_FOR_DELIVERY` out-of-order race conditions).
  5. **Transactional Update**: Atomic DB commit with Transactional Outbox pattern (`webhook_events` + domain entities).
  6. **Publish to Broker & 200 OK ACK**: Asynchronous Kafka event dispatch and fast sub-250ms `200 OK` response to third-party providers.
- **7 Reliability Pillars**: Cryptographic verification, Deduplication, Fast ACK, Monotonic State Machines, Transactional Outbox, Dead Letter Queue (DLQ), and Audit Logging.
- **Provider & Consumer Ecosystem**: Ingestion from Payment Gateways (Razorpay, Stripe) & Logistics (Shiprocket, Delhivery), consuming across Order, Fulfillment, and Analytics microservices.
- **Interactive Tools**: 🌓 Dark/Light Mode, 📸 2.5x Retina PNG Export, and 📋 1-Click LinkedIn Caption Copy.

---

## 🔑 Day 6 Excalidraw Edition: Idempotency in Production Systems (`day6-idempotency-excalidraw.html`)
An authentic hand-drawn Excalidraw architectural diagram based on real-world mission-critical payment and order processing:
- **Core Principle**: *"Same Request ≠ Duplicate Operation — Build for retries, not just success."*
- **6-Stage Production Pipeline**:
  1. **Client**: Issues HTTP requests with a unique `Idempotency-Key` header (e.g. `POST /payments/create`).
  2. **API Gateway / Controller**: Extracts and validates the key, evaluates rate limits, and forwards to the microservice.
  3. **Idempotency Check (Redis + PostgreSQL Dual Store)**:
     - `Redis`: Ultra-fast sub-millisecond cache lookup & distributed lock.
     - `PostgreSQL`: Authoritative source of truth.
     - `Key Found (YES)`: Instantly return stored response without re-executing business logic.
     - `Key Not Found (NO)`: Safely proceed with domain execution.
  4. **Business Service**: Executes payment/order logic, validates state transitions, and calculates domain state.
  5. **Database Transaction**: Atomic ACID commit saving business entities, idempotency records, tracking events, and the Transactional Outbox event in **one single SQL transaction**.
  6. **Outbox + Kafka**: Asynchronously streams events to Kafka topic (`courier.shipment.events` / `order.paid.events`), preventing the dual-write hazard.
- **Consumer Inbox Pattern (Step 7)**:
  - Microservice consumers (`E-Commerce`, `Logistics`, `Notifications`, `Analytics`) maintain their own `Inbox/ProcessedEvents` table (`UNIQUE(consumerGroup, eventId)`) to ensure exactly-once side effects.
- **Key Database Constraints**:
  - `UNIQUE(provider, idempotencyKey)`
  - `UNIQUE(sellerId, externalOrderId)`
  - `UNIQUE(paymentProvider, transactionId)`
  - `UNIQUE(eventId, consumerGroup)`
- **The Big Lesson**:
  > "Idempotency is not just a Redis key. Exactly-once behavior is achieved through idempotent processing, not by assuming the network will deliver exactly once."
- **Interactive Tools**: 🌓 Light/Dark mode, 📸 2.5x Retina PNG Export, and 📋 1-Click LinkedIn Post Caption Copy.

---

## 👤 Author

**Mrityunjay Kumar**  
- GitHub: [@mrityunjay45108](https://github.com/mrityunjay45108)  
- Focus: Full Stack & Distributed Systems Architecture

---

## 📄 License
MIT License. Feel free to use this architecture reference for your projects and interviews!
