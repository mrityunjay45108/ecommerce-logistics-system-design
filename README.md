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
# Windows
start index.html
start day4-excalidraw.html

# macOS
open index.html
open day4-excalidraw.html

# Linux
xdg-open index.html
xdg-open day4-excalidraw.html
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

## 👤 Author

**Mrityunjay Kumar**  
- GitHub: [@mrityunjay45108](https://github.com/mrityunjay45108)  
- Focus: Full Stack & Distributed Systems Architecture

---

## 📄 License
MIT License. Feel free to use this architecture reference for your projects and interviews!
