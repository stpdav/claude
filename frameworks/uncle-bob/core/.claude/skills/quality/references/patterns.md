# Design Patterns Reference

## When to Use Each Pattern

### Creational
- **Factory**: when object creation logic is complex or varies by type
- **Builder**: when constructing objects with many optional parameters
- **Singleton**: sparingly - prefer dependency injection instead

### Structural
- **Adapter**: wrapping a third-party interface you don't control
- **Decorator**: adding behavior without subclassing (e.g. logging, caching)
- **Repository**: abstracting data access from business logic

### Behavioral
- **Strategy**: swappable algorithms (e.g. different sorting, payment methods)
- **Observer**: event-driven updates between decoupled components
- **Command**: encapsulate actions as objects (undo, queues, audit trails)

## Red Flags (Avoid These)

- **God class**: one class knowing/doing too much
- **Feature envy**: a function that mostly uses another class's data
- **Shotgun surgery**: one change requiring edits in many unrelated places
- **Primitive obsession**: using strings/ints where a typed object belongs
- **Long parameter list**: >3 params → use a config object or builder
