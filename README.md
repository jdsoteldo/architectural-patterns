*This page was written by AI
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
# architectural-patterns
