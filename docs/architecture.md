# BotLR Architecture

WhatsApp order-taking bot for small sellers. ~100 orders/day.

## Architecture Overview

BotLR uses **Hexagonal Architecture** (Ports & Adapters) to keep business logic isolated from frameworks and infrastructure.

**Why Hexagonal?** The bot must support multiple WhatsApp providers (360dialog, Twilio) without changing core logic. The architecture defines clear interfaces (Ports) that the domain code depends on, with concrete implementations (Adapters) plugged in at the edges. This makes the domain testable without FastAPI, PostgreSQL, or Redis, and allows swapping providers by implementing a single interface.

**Layers:**
- **Driving Adapters** (inbound): WhatsApp webhook handler, Dashboard REST API, SSE endpoint
- **Ports** (interfaces): `MessagePort`, `OrderPort`, `CatalogPort`, `SSEPort`, `MessagingProvider`, `StorageProvider`
- **Application/Domain**: BotEngine (FSM), OrderService, CatalogService
- **Driven Adapters** (outbound): 360dialog/Twilio providers, PostgreSQL repositories, Redis cache

## System Context Diagram

```mermaid
graph TB
    C[Customer<br/>WhatsApp] -->|Sends message| WP[WhatsApp API<br/>360dialog / Twilio]
    WP -->|Webhook| B[BotLR Backend<br/>FastAPI + PostgreSQL + Redis]
    B -->|REST + SSE| D[Seller Dashboard<br/>Next.js]
    B -->|API calls| WP
    C -->|Receives message| WP
```

## Component Diagram

```mermaid
graph TB
    subgraph Backend
        WH[Webhook Router] --> MA[MessagingAdapter]
        API[API Router] --> OS[OrderService]
        API --> CS[CatalogService]
        API --> SSE[SSE Service]
        MA --> BE[BotEngine / FSM]
        BE --> CS
        BE --> OS
        OS --> SSE
    end
    subgraph Frontend
        DASH[Dashboard<br/>Next.js App Router]
    end
    subgraph Infrastructure
        PG[(PostgreSQL)]
        RD[(Redis)]
    end
    subgraph Providers
        W360[360dialog]
        WAT[Twilio]
    end
    MA --> W360
    MA --> WAT
    CS --> PG
    CS --> RD
    OS --> PG
    SSE --> RD
    DASH --> SSE
    DASH --> API
```

## Design Patterns

| Pattern | Where Applied | Why |
|---------|--------------|-----|
| **Strategy** | `MessagingProvider` interface with `WhatsApp360dialogProvider` and `TwilioProvider` | Swap providers without touching domain code |
| **State** | `ConversationFSM` for bot flow | Explicit transitions prevent invalid jumps (e.g., `greeting` → `order_confirmed`) |
| **Repository** | `OrderRepository`, `ProductRepository`, `ConversationRepository` | Domain remains persistence-agnostic; test with in-memory fakes |
| **Observer** | `SSEService` subscribes to `OrderService` events | Decouples event source from consumers (dashboard, logging) |
| **Factory** | `MessagingProviderFactory` resolves provider per seller | Each seller may use a different provider; creation logic centralized |
| **Dependency Injection** | FastAPI `Depends` for services and repositories | Endpoints are pure functions; test with mocked dependencies |

## Data Model

```mermaid
erDiagram
    SELLER {
        uuid id PK
        varchar email UK
        varchar phone UK
        varchar business_name
        jsonb provider_config
        boolean is_active
    }
    CUSTOMER {
        uuid id PK
        uuid seller_id FK
        varchar phone_number
        varchar name
    }
    PRODUCT {
        uuid id PK
        uuid seller_id FK
        varchar name
        decimal price
        int stock
        varchar category
        boolean is_available
    }
    ORDER {
        uuid id PK
        uuid seller_id FK
        uuid customer_id FK
        varchar status
        decimal total_price
        timestamp created_at
        timestamp updated_at
    }
    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
    }
    STATUS_HISTORY {
        uuid id PK
        uuid order_id FK
        varchar status
        varchar actor
        text reason
        timestamp created_at
    }
    CONVERSATION {
        uuid id PK
        uuid seller_id FK
        uuid customer_id FK
        varchar current_state
        jsonb cart_data
        timestamp last_activity_at
    }
    MESSAGE {
        uuid id PK
        uuid conversation_id FK
        varchar direction
        text content
        timestamp created_at
    }
    SELLER ||--o{ CUSTOMER : has
    SELLER ||--o{ PRODUCT : owns
    SELLER ||--o{ ORDER : receives
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    ORDER ||--o{ STATUS_HISTORY : tracks
    PRODUCT ||--o{ ORDER_ITEM : referenced_by
    CONVERSATION ||--o{ MESSAGE : contains
```

## Sequence Diagrams

### Customer Places Order

```mermaid
sequenceDiagram
    participant C as Customer
    participant W as WhatsApp
    participant WH as Webhook
    participant MA as MessagingAdapter
    participant BE as BotEngine
    participant FSM as FSM
    participant OS as OrderService
    participant PG as PostgreSQL
    participant SSE as SSE
    participant D as Dashboard

    C->>W: "Hello"
    W->>WH: webhook
    WH->>MA: normalize & verify
    MA->>BE: process message
    BE->>FSM: get state
    FSM-->>BE: greeting
    BE->>MA: send catalog
    MA->>W: dispatch
    W->>C: menu

    C->>W: "1" (product)
    W->>WH: webhook
    MA->>BE: process
    BE->>FSM: update state
    BE->>MA: "How many?"
    MA->>W: dispatch
    W->>C: quantity prompt

    C->>W: "2" (quantity)
    W->>WH: webhook
    MA->>BE: process
    BE->>FSM: update state
    BE->>MA: "Confirm?"
    MA->>W: dispatch
    W->>C: summary

    C->>W: "yes"
    W->>WH: webhook
    MA->>BE: process
    BE->>FSM: order_confirmed
    BE->>OS: create order
    OS->>PG: INSERT orders, items, status_history
    OS->>SSE: publish
    SSE->>D: new order
    OS->>MA: notify customer
    MA->>W: dispatch
    W->>C: "Order #123 confirmed!"
```

### Seller Confirms Order

```mermaid
sequenceDiagram
    participant S as Seller
    participant D as Dashboard
    participant API as API
    participant OS as OrderService
    participant PG as PostgreSQL
    participant SSE as SSE
    participant MA as MessagingAdapter
    participant W as WhatsApp
    participant C as Customer

    S->>D: Click "Confirm"
    D->>API: PATCH /orders/{id}
    API->>OS: update status
    OS->>PG: SELECT + validate transition
    OS->>PG: UPDATE status + INSERT history
    OS->>SSE: publish
    SSE->>D: update card
    OS->>MA: notify customer
    MA->>W: dispatch
    W->>C: "Order confirmed!"
    API-->>D: 200 OK
```

## Architecture Decision Records

| Decision | Options Considered | Chosen | Why |
|----------|-------------------|--------|-----|
| Architecture | MVC, Layered, Hexagonal | Hexagonal | Provider swapping, testability, framework independence |
| Backend Framework | Django, Flask, FastAPI | FastAPI | Native async, auto OpenAPI, DI via `Depends` |
| Frontend Framework | CRA, Vue/Nuxt, Next.js | Next.js App Router | SSR, server components, strong TS support |
| Real-time Transport | WebSockets, Long Polling, SSE | SSE | Unidirectional push; native HTTP/2; simpler than WebSockets |
| Primary Database | MongoDB, SQLite, MySQL, PostgreSQL | PostgreSQL | ACID orders, JSONB for flexible metadata, mature ecosystem |
| Cache / State | Memcached, Redis | Redis | Cache, Pub/Sub, and conversation state in one store |
| Multi-tenancy | Schema-per-tenant, DB-per-tenant, row-level | Row-level (`seller_id`) | Simple ops, single migration path, sufficient for 50 sellers |
| WhatsApp Providers | Meta Cloud API, on-premise, unofficial | 360dialog / Twilio | Official APIs, no infra overhead, abstracted via adapter |

## Multi-Tenancy

Row-level isolation via `seller_id` foreign keys on every entity. Each query is scoped by `seller_id`; JWT tokens embed the `seller_id` claim; Redis keys are prefixed with `seller:{seller_id}:`. Webhooks are routed to the correct seller by phone number or API key before processing. This supports 50 sellers on a single instance with minimal operational overhead.

## Security

- **Auth**: JWT (HS256) with 30-minute expiry; `seller_id` claim enforced on every endpoint
- **Webhooks**: Provider signature verification (Twilio `X-Twilio-Signature`, 360dialog API key)
- **Input**: Pydantic validation on all inbound requests; parameterized queries only
- **Images**: Max 5 MB, JPEG/PNG/WebP whitelist, file type validation via magic numbers
- **Secrets**: Provider API keys encrypted at rest (AES-256); never logged
- **Rate limits**: Webhooks 10 req/sec global; API 100 req/min per seller; login 5 failures/15 min per IP
- **Audit**: All order status changes to immutable `status_history`; structured JSON logging

---

*End of Architecture Documentation*
