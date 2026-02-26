# Practical Exercises in Ruby
All exercises build the same domain — a **simple online bookstore** — so you can directly compare how each pattern handles the same requirements.
**Core features across all exercises:**
- Users can browse books
- Users can place orders
- Inventory is tracked
- Notifications are sent when orders are placed

## Exercise 1: Layered Architecture — Bookstore CLI
**Goal:** Build a CLI bookstore where every request flows through Presentation → Business → Persistence → Database layers.
```md
bookstore_layered/
├── app.rb                      # Entry point
├── presentation/
│   └── cli.rb                  # CLI interface (reads input, displays output)
├── business/
│   ├── book_service.rb         # Business rules for books
│   ├── order_service.rb        # Business rules for orders
│   └── validators/
│       └── order_validator.rb  # Validation logic
├── persistence/
│   ├── book_repository.rb      # Data access for books
│   ├── order_repository.rb     # Data access for orders
│   └── inventory_repository.rb
└── database/
    └── store.rb                # In-memory hash simulating a DB
```

### **Tasks:**
- Build the Database layer — create an in-memory store (hash) with seed data (books with titles, prices, stock counts).
- Build the Persistence layer — create repository classes with methods like find_all, find_by_id, save. They ONLY talk to the database layer.
- Build the Business layer — create services that enforce rules: "can't order more than stock allows," "order total = sum of book prices × quantities." Services ONLY call repositories, never the database directly.
- Build the Presentation layer — a CLI that shows menus, takes input, calls the business layer, formats output. Never accesses persistence or database directly.
- Observe the sinkhole anti-pattern — add a "list all books" feature. Notice how the request flows through Business → Persistence just to forward data unchanged. Reflect on when this pattern adds overhead without value.
- Test each layer in isolation — write unit tests for each layer, mocking the layer below.

### **What to observe:**
- How changes to one layer (e.g., swapping from hash to JSON file) don't affect others
- How the sinkhole pattern shows up in simple read operations
- How adding a new feature (e.g., "search books by author") requires touching multiple layers
