*This page was written _mostly_ by AI
# Software Architecture Patterns: Theory & Practice in Ruby*

A study guide covering core software architecture patterns — from monolithic to distributed — with theory, ASCII diagrams, and hands-on exercises in Ruby. All exercises build the same domain (an online bookstore) so patterns can be compared directly.

---

## Index

### Monolithic Patterns

| Topic | File | Description |
|-------|------|-------------|
| Theory | [mono_patterns_theory.md](mono_patterns_theory.md) | Layered Architecture (N-tier) and Modular Monolith — concepts, diagrams, strengths/weaknesses, and when to use each |
| Exercises | [mono_exercises.md](mono_exercises.md) | Bookstore CLI exercise for Layered Architecture with tasks and observations |

### Distributed Patterns

| Topic | File | Description |
|-------|------|-------------|
| Theory | [distributed_patterns.md](distributed_patterns.md) | Event-Driven Architecture (Broker & Mediator), Space-Based Architecture, and Service-Based Architecture — concepts, diagrams, strengths/weaknesses |
| Exercises | [distributed_patterns_exercises.md](distributed_patterns_exercises.md) | *(in progress)* |

---

## Patterns Covered

1. **Layered Architecture (N-tier)** — horizontal layers with top-down request flow
2. **Modular Monolith** — vertical domain modules inside a single deployment unit
3. **Event-Driven Architecture** — broker and mediator topologies for async event processing
4. **Space-Based Architecture** — in-memory data grids for extreme scalability
5. **Service-Based Architecture** — coarse-grained domain services as a middle ground between monolith and microservices

## Summary Comparison

| Aspect                | Layered                | Modular Monolith                | Event-Driven           | Space-Based                | Service-Based                           |
|-----------------------|------------------------|---------------------------------|------------------------|----------------------------|-----------------------------------------|
| **Deployment**        | Single unit            | Single unit                     | Per processor          | Per unit                   | Per service                             |
| **Communication**     | Method calls (down)    | Public APIs + events            | Async events           | In-memory replication      | HTTP/REST                               |
| **Data**              | Single DB              | Partioned per module            | Per processor          | In-memory per unit         | Shared DB                               |
| **Scalability**       | Low                    | Low-Medium                      | High                   | Very High                  | Medium                                  |
| **Complexity**        | Low                    | Medium                          | High                   | Very High                  | Medium                                  |
| **Fault Tolerance**   | Low                    | Low                             | High                   | High                       | Medium - High                           |
| **Best Use Case**     | Small simple CRUD apps | Growing apps, pre-microservices | High-thoroughput async | Extreme concurrency spiked | Business apps needing deploy assistance |


## Recommended Order of Implementation
1. **Layered** — start here to establish the baseline
2. **Modular Monolith** — refactor the layered version into modules to see the contrast
3. **Service-Based** — extract modules into services to experience the monolith-to-distributed transition
4. **Event-Driven** — rebuild with events to see the paradigm shift from request/response to event/reaction
5. **Space-Based** — build last, as it's the most complex and benefits from understanding the others first
