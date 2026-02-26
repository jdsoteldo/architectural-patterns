# Software Architecture Patterns: Theory & Practice in Ruby

## Monolithic Architecture
Patterns described in this doc:
- Layered Architecture (N-tier)
- Modular Monolith

## 1. Layered Architecture (N-tier)
#### Theory:
Layered architecture is the most common and traditional pattern. It organizes code into horizontal layers, each with a specific role. Requests flow top-down through the layers.
```md
┌─────────────────────────────┐
│      Presentation Layer     │  ← UI, API controllers, CLI
├─────────────────────────────┤
│       Business Layer        │  ← Domain logic, rules, validations
├─────────────────────────────┤
│      Persistence Layer      │  ← Data access, repositories, ORM
├─────────────────────────────┤
│       Database Layer        │  ← Actual storage (SQL, files, etc.)
└─────────────────────────────┘
```

### Key Concepts:
- **Separation of concerns** — each layer has one job
- **Closed layers** — a request must pass through each layer sequentially (no skipping). This is called layers of isolation. When a layer is "closed," changes in one layer don't ripple into others. For example, if you swap your database, only the persistence layer changes.
- **Open layers** — sometimes a layer can be bypassed (e.g., a "shared services" layer). This adds flexibility but reduces isolation.
- **Sinkhole anti-pattern** — when requests pass through layers doing nothing (just forwarding data). If 80%+ of your requests are sinkholes, the architecture is wrong for your use case.

### Strengths:
- Simple, well-understood, easy to onboard new developers
- Clear separation of concerns
- Each layer is independently testable
- Good starting point for most applications

### Weaknesses:
- Monolithic deployment — the whole thing deploys as one unit
- Tends toward low overall agility — changes often touch multiple layers
- Can become a "big ball of mud" if discipline isn't maintained
- Performance can suffer due to unnecessary layer traversal
- Doesn't scale well — you scale the entire app, not individual pieces

**Best for**: Small to medium apps, CRUD-heavy applications, teams new to architecture patterns, internal tools.

## 2. Modular Monolith
#### Theory:
A modular monolith keeps the single deployment unit of a monolith but organizes code into vertical modules (by domain/feature) instead of horizontal layers. Each module is self-contained with its own layers internally.
```md
┌─────────────────────────────────────────────────┐
│                  Single Process                 │
│                                                 │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐   │
│  │  Orders    │ │  Users     │ │  Inventory │   │
│  │            │ │            │ │            │   │
│  │ ┌────────┐ │ │ ┌────────┐ │ │ ┌────────┐ │   │
│  │ │  API   │ │ │ │  API   │ │ │ │  API   │ │   │
│  │ ├────────┤ │ │ ├────────┤ │ │ ├────────┤ │   │
│  │ │ Logic  │ │ │ │ Logic  │ │ │ │ Logic  │ │   │
│  │ ├────────┤ │ │ ├────────┤ │ │ ├────────┤ │   │
│  │ │  Data  │ │ │ │  Data  │ │ │ │  Data  │ │   │
│  │ └────────┘ │ │ └────────┘ │ │ └────────┘ │   │
│  └────────────┘ └────────────┘ └────────────┘   │
│        │               │              │         │
│        └───── Well-defined APIs ──────┘         │
└─────────────────────────────────────────────────┘
```

### Key concepts:
- **Module boundaries** — each module owns its domain (Users, Orders, Payments). Modules communicate through well-defined public interfaces, never by reaching into each other's internals.
- **Data ownership** — each module ideally owns its own data/tables. No shared database tables between modules. This is the critical discipline.
- **Inter-module communication** — modules talk through public APIs (method calls, events, or a shared bus). Never through direct database access.
- **Stepping stone to microservices** — if a module needs to scale independently, it can be extracted into a service. The boundaries are already drawn.

### Strengths:
 - Single deployment simplicity (no network calls, no distributed transactions)
 - Strong domain boundaries prevent "big ball of mud"
 - Easier to refactor than layered; module boundaries are explicit
 - Path to microservices without the upfront complexity
 - Each module can be developed/tested independently
 - In-process communication = no network latency

### Weaknesses:
- Still a monolith — scales as one unit
- Requires discipline to maintain module boundaries (temptation to "just import" across modules)
- Shared database can lead to coupling if not carefully managed
- Single point of failure in deployment

**Best for**: Medium to large applications, teams that want domain separation without distributed system complexity, as a precursor to microservices.

