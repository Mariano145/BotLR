# BotLR Requirements

WhatsApp-based conversational bot for small sellers. Automates order-taking; customers chat via WhatsApp, sellers manage orders via a real-time web dashboard.

---

## Functional Requirements

### Bot

| ID | Requirement | Priority |
|----|-------------|----------|
| BOT-001 | Initialize FSM on first message or after 24h idle | Must |
| BOT-002 | Greet customer and present catalog | Must |
| BOT-003 | Allow product selection by name, number, or image | Must |
| BOT-004 | Prompt for quantity and validate input | Must |
| BOT-005 | Present order summary and ask for confirmation | Must |
| BOT-006 | Submit order on confirmation; cancel on "no" | Must |
| BOT-007 | Answer "status" / "estado" with current order status | Must |
| BOT-008 | Reset to greeting after 30 min idle | Must |
| BOT-009 | Gracefully reject unsupported message types (voice, documents) | Must |
| BOT-010 | Send product images with catalog | Must |
| BOT-011 | Support Spanish and English per seller config | Should |

### Messaging

| ID | Requirement | Priority |
|----|-------------|----------|
| MSG-001 | Unified provider-agnostic interface for send/receive | Must |
| MSG-002 | Normalize incoming webhooks to common format | Must |
| MSG-003 | Dispatch outgoing messages to correct provider | Must |
| MSG-004 | Support image and document media | Must |
| MSG-005 | Verify webhook signatures (Twilio, 360dialog) | Must |
| MSG-006 | Report delivery status if provider supports it | Should |
| MSG-007 | Fallback provider on primary failure | May |

### Dashboard

| ID | Requirement | Priority |
|----|-------------|----------|
| DASH-001 | Real-time order push via SSE (< 2s) | Must |
| DASH-002 | List orders with status filter | Must |
| DASH-003 | View order details (customer, items, total) | Must |
| DASH-004 | Update order status (confirmed → preparing → completed) | Must |
| DASH-005 | Confirm order and notify customer | Must |
| DASH-006 | CRUD catalog products | Must |
| DASH-007 | Authenticate and restrict to seller's own data | Must |
| DASH-008 | Configure bot language, greeting, business hours | Must |
| DASH-009 | Require cancellation reason | Must |
| DASH-010 | Auto-reconnect SSE with polling fallback | Must |

### Multi-tenancy

| ID | Requirement | Priority |
|----|-------------|----------|
| MT-001 | Register new sellers with unique identifiers | Must |
| MT-002 | Strict data isolation: seller can only access own data | Must |
| MT-003 | Independent seller config (provider, language, catalog) | Must |
| MT-004 | Route webhooks to correct seller by phone/API key | Must |
| MT-005 | Shared infrastructure without data leakage | Must |
| MT-006 | JWT tokens contain `seller_id` claim | Must |
| MT-007 | Support seller deactivation without affecting others | Must |

### Catalog

| ID | Requirement | Priority |
|----|-------------|----------|
| CAT-001 | Product CRUD (name, description, price, category, image) | Must |
| CAT-002 | Validate name non-empty and price > 0 | Must |
| CAT-003 | Image upload and storage | Must |
| CAT-004 | Serve images via URLs for WhatsApp | Must |
| CAT-005 | Category management | Should |
| CAT-006 | Toggle availability without deleting | Must |
| CAT-007 | Cache catalog in Redis | Must |
| CAT-008 | Bulk import/export via CSV | May |

### Orders

| ID | Requirement | Priority |
|----|-------------|----------|
| ORD-001 | Create order on customer confirmation | Must |
| ORD-002 | Status lifecycle: created → confirmed → preparing → completed / cancelled | Must |
| ORD-003 | Enforce valid status transitions | Must |
| ORD-004 | Record created_at, updated_at, status_changed_at | Must |
| ORD-005 | Support cancellation by seller or auto-abandonment | Must |
| ORD-006 | Notify customer on status change | Must |
| ORD-007 | Immutable status history (actor, timestamp, reason) | Must |
| ORD-008 | Search orders by customer phone or order ID | Must |
| ORD-009 | Provide order summary for bot display | Must |

---

## Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | Bot response latency | ≤ 3s |
| Performance | Dashboard SSE latency | ≤ 2s |
| Performance | Catalog API response | ≤ 200 ms |
| Performance | Order query by ID | ≤ 100 ms |
| Availability | Bot uptime | ≥ 99.9% |
| Availability | Dashboard uptime | ≥ 99.5% |
| Scalability | Sellers per instance | 50 |
| Scalability | Concurrent conversations per seller | 50 |
| Scalability | Orders per day | 100 |
| Scalability | Products per seller | 500 |
| Security | HTTPS/TLS 1.3 on all traffic | Must |
| Security | JWT expiry after 30 min inactivity | Must |
| Security | Webhook signature verification | Must |
| Security | SQL injection prevention | Must |
| Security | Rate limiting (API 100/min, webhooks 10/sec) | Must |
| Security | Provider API keys encrypted at rest | Must |
| Durability | ACID order persistence | Must |
| Durability | Daily automated backups | Must |
| Maintainability | Backend test coverage | ≥ 80% |
| Maintainability | CI/CD on every PR | Must |
| Maintainability | Versioned, reversible migrations | Must |
| Maintainability | Auto-generated API docs | Must |

---

## Out of Scope

| Feature | Reason |
|---------|--------|
| Payment gateway integration | Regulatory complexity; MVP is order-only |
| Native mobile app | Web dashboard is sufficient |
| Other messaging platforms (Telegram, Messenger) | WhatsApp dominates target market; adapter ready for future |
| Advanced analytics / sales reports | Nice-to-have; beyond MVP scope |
| Push notifications (outside dashboard) | SSE covers real-time needs |
| Advanced user roles (admin, manager) | Single seller = single user in MVP |
| AI / NLP intent recognition | Keyword matching is sufficient for ~100 orders/day |
| Multi-location per seller | Single location in MVP |
| Delivery tracking | Pickup / seller-managed delivery in MVP |
| Customer ratings & reviews | Beyond MVP scope |
| Inventory management (stock decrements) | Availability toggle is sufficient |
| Multi-currency support | Single currency per seller in MVP |

---

## Glossary

| Term | Definition |
|------|------------|
| BotLR | Bot de Listado de pedidos por WhatsApp — WhatsApp order bot |
| SSE | Server-Sent Events; unidirectional server→client push over HTTP |
| FSM | Finite State Machine; bot conversation flow engine |
| Webhook | HTTP callback from WhatsApp provider on incoming message |
| Messaging Adapter | Abstraction layer normalizing provider-specific messages |
| Provider | WhatsApp Business API service (360dialog, Twilio) |
| Seller | Business owner using BotLR |
| Customer | End user ordering via WhatsApp |
| Dashboard | Next.js web app for sellers |
| Multi-tenancy | Multiple sellers sharing one instance with isolated data |
| Catalog | Seller's product collection |
| Order Lifecycle | created → confirmed → preparing → completed (or cancelled) |
| Status History | Immutable audit trail of order status changes |
| ADR | Architecture Decision Record |
| JWT | JSON Web Token |
| MVP | Minimum Viable Product |

---

*End of Requirements Document*
