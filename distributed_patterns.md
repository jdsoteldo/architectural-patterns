# Software Architecture Patterns: Theory & Practice in Ruby

## Distributed Patterns
Patterns described in this doc:
- Event-Driven Architecture (Broker & Mediator)
- Space-Based Architecture
- Service-Based Architecture

## 1. Event-Driven Architecture (EDA)
#### Theory:
Event-driven architecture is built around the **production, detection, and reaction to events**. Components communicate by emitting and consuming events rather than making direct calls.
There are two main topologies:

### Broker Topology
No central orchestrator. Events flow through a lightweight message broker. Each processor listens, reacts, and optionally emits new events — which feed back into the broker, creating a chain reaction.
```md
                        ┌─────────────────┐
  "OrderPlaced" ──────► │     Broker      │ ◄──────────────────┐
                        │   (Channels)    │                    │
                        └────────┬────────┘                    │
                                 │                             │
              ┌──────────────────┼──────────────────┐          │
              ▼                  ▼                  ▼          │
        ┌───────────┐    ┌────────────┐    ┌────────────┐      │
        │  Payment  │    │ Inventory  │    │  Notif.    │      │
        │ Processor │    │ Processor  │    │ Processor  │      │
        └─────┬─────┘    └─────┬──────┘    └────────────┘      │
              │                │                               │
              ▼                ▼                               │
       "PaymentCharged"  "StockUpdated" ───────────────────────┘
        (new event)       (new event)
         loops back        loops back
```

- Good for simple flows, high throughput
- Hard to track end-to-end flow, no central error handling
- No single component "owns" the workflow — the chain emerges from processors reacting independently
- Any processor can trigger further reactions by emitting new events

### Mediator Topology
A central **mediator** (orchestrator) receives the initial event and coordinates the workflow by generating **commands** and sending them to specific processors. Unlike the broker, the mediator knows and controls the full sequence.
```md
                        ┌──────────────────┐
  "OrderPlaced" ──────► │     Mediator     │
                        │  (Orchestrator)  │
                        └────────┬─────────┘
                                 │
                    Sends commands in sequence:
                                 │
          ┌──────────────────────┼──────────────────────┐
          │ Step 1               │ Step 2               │ Step 3
          ▼                      ▼                      ▼
  "ValidateOrder"        "ChargePayment"        "SendConfirmation"
          │                      │                      │
          ▼                      ▼                      ▼
   ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
   │ Validation  │       │   Payment   │       │   Notif.    │
   │  Processor  │       │  Processor  │       │  Processor  │
   └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                        Reports back to
                          Mediator
                    (success, failure, or
                     partial completion)
```
- Good for complex flows that require coordination
- The mediator knows the full workflow and controls the order of steps
- Easier error handling and recovery — if Step 2 fails, the mediator can retry or compensate
- But the mediator can become a bottleneck/single point of failure

#### Key concepts:
- **Events vs. Commands** — an event says "this happened" (past tense: OrderPlaced). A command says "do this" (ProcessPayment). Broker topology uses events; mediator uses both.
- **Eventual consistency** — since processing is asynchronous, the system is eventually consistent, not immediately consistent.
- **Fire and forget** — producers don't wait for consumers. This decouples them but makes error handling harder.

### Strengths:
- Highly decoupled components
- Excellent scalability (add more consumers)
- Great responsiveness (async processing)
- Easy to add new functionality (just add a new subscriber)

### Weaknesses:
- Hard to test end-to-end
- Error handling is complex (what if step 3 of 5 fails?)
- Eventual consistency can confuse users
- Debugging distributed event flows is difficult
- No straightforward request-reply

**Best for**: Systems requiring high scalability and responsiveness, complex event processing, systems where components change frequently.

## 2. Space-Based Architecture
#### Theory:
Space-based architecture is designed to solve **extreme scalability and concurrency** problems by removing the central database as a bottleneck. Instead of reading/writing from a single database, each processing unit has its own **in-memory data grid**.
The name comes from "tuple space" — the concept of shared distributed memory.
```md
┌──────────────────────────────────────────────────────────┐
│                   Virtualized Middleware                 │
│                                                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
│  │ Messaging   │   │  Data Grid  │   │ Processing  │     │
│  │   Grid      │   │   (Sync)    │   │    Grid     │     │
│  └─────────────┘   └─────────────┘   └─────────────┘     │
│         │                 │                  │           │
│         ▼                 ▼                  ▼           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │            Data Replication Engine                  │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
         │                   │                    │
         ▼                   ▼                    ▼
┌──────────────┐   ┌──────────────┐    ┌──────────────┐
│ Processing   │   │ Processing   │    │ Processing   │
│   Unit 1     │   │   Unit 2     │    │   Unit 3     │
│              │   │              │    │              │
│ ┌──────────┐ │   │ ┌──────────┐ │    │ ┌──────────┐ │
│ │ In-Memory│ │   │ │ In-Memory│ │    │ │ In-Memory│ │
│ │   Cache  │◄├───┤►│   Cache  │◄├────┤►│   Cache  │ │
│ └──────────┘ │   │ └──────────┘ │    │ └──────────┘ │
│ ┌──────────┐ │   │ ┌──────────┐ │    │ ┌──────────┐ │
│ │  Logic   │ │   │ │  Logic   │ │    │ │  Logic   │ │
│ └──────────┘ │   │ └──────────┘ │    │ └──────────┘ │
└──────────────┘   └──────────────┘    └──────────────┘
                          │
                          ▼
                   ┌──────────────┐
                   │   Database   │  ← Async write-behind
                   │  (Optional)  │
                   └──────────────┘
```

### Key concepts:
- **Processing Unit** — a self-contained unit with application logic AND an in-memory data grid. It can handle a request entirely on its own without hitting a central database.
- **Virtualized middleware** — manages the infrastructure:
  - **Messaging grid** — routes requests to available processing units
  - **Data grid** — replicates data between processing units so they all have the same state
  - **Processing grid** — orchestrates multi-step requests across units
- **Data replication** — when one unit updates data, the change is replicated to all other units' in-memory grids (usually async)
- **Database write-behind** — the database is written to asynchronously. It's a backup/recovery store, NOT the primary data source during operation.
- **Data collisions** — since all units process concurrently with local data, collisions can happen. Rate depends on: update frequency, cache size, replication latency, number of units.

### Strengths:
- Near-infinite scalability (just add processing units)
- No central database bottleneck
- Very high performance (in-memory)
- Handles variable load by spinning units up/down
- High fault tolerance

### Weaknesses:
- Very complex to implement and maintain
- Data consistency challenges (eventual consistency between units)
- Not suited for traditional large-scale relational data
- Testing the replication and grid layer is difficult
- High infrastructure cost
- Data collisions require conflict resolution strategies

**Best for**: High-volume, high-concurrency systems (concert ticket sales, online auctions, stock trading), applications with highly variable and unpredictable load spikes.

## 3. Service-based Architecture
#### Theory:
Service-based architecture is a **pragmatic middle ground** between a monolith and microservices. It splits the application into a small number of **coarse-grained domain services** (typically 4-12), each deployed independently but sharing a database.
```md
┌─────────────┐    ┌─────────────┐
│   Web UI    │    │  Mobile UI  │
└──────┬──────┘    └──────┬──────┘
       │                  │
       ▼                  ▼
┌────────────────────────────────┐
│          API Gateway           │
└──────┬──────────┬───────┬──────┘
       │          │       │
       ▼          ▼       ▼
┌──────────┐ ┌────────┐ ┌──────────┐
│  Order   │ │  User  │ │Inventory │
│ Service  │ │Service │ │ Service  │
│          │ │        │ │          │
│(multiple │ │(own    │ │(own      │
│ layers   │ │ layers)│ │ layers)  │
│ inside)  │ │        │ │          │
└────┬─────┘ └───┬────┘ └────┬─────┘
     │           │           │
     ▼           ▼           ▼
┌─────────────────────────────────┐
│     Shared Database (or         │
│     logically partitioned)      │
└─────────────────────────────────┘
```

### Key concepts:
- **Coarse-grained services** — unlike microservices (fine-grained, one purpose), service-based services are larger and encompass an entire domain. An "Order Service" handles order creation, updates, history, etc.
- **Shared database** — services typically share a database, which enables ACID transactions across domains. This is a deliberate tradeoff — you get data consistency but introduce coupling. Some teams partition the DB logically (separate schemas per service).
- **Independent deployment** — each service is deployed separately, so you can update one without redeploying all.
- **API layer** — services are accessed through a gateway or directly. The UI doesn't know which service to call — the gateway routes.
- **Inter-service communication** — services can call each other via REST, gRPC, or messaging. Keep this to a minimum; chatty inter-service calls create coupling.

### Strengths:
- Simpler than microservices (fewer services, shared DB)
- Independent deployability without full microservice complexity
- ACID transactions possible (shared DB)
- Easier to maintain domain boundaries than a monolith
- Good fault tolerance (one service can fail without taking down others)
- Pragmatic — gets 80% of microservice benefits with 20% of the effort

### Weaknesses:
- Shared database can become a coupling point and bottleneck
- Services can grow too large if boundaries aren't maintained
- Not as scalable as microservices (coarser units to scale)
- Database changes can affect multiple services
**Best for**: Most business applications, teams transitioning from monolith, medium to large applications that need independent deployment without microservice overhead.

