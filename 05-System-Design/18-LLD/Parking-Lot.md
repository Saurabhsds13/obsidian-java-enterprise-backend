---
type: system-design
level: LLD
domain: system-design
difficulty: medium
status: inbox
tags: [system-design]
---

# Parking Lot (LLD)

## Requirements
- Multiple levels, each with spots of types (motorcycle, compact, large).
- Assign a suitable spot on entry, free it on exit, charge by duration.
- Extensible pricing and spot types.

## Entities
- `Vehicle` (abstract) → `Motorcycle`, `Car`, `Bus`.
- `ParkingSpot` (with `SpotType`).
- `Level` (holds spots).
- `ParkingLot` (holds levels).
- `Ticket` (entry time, spot).
- `PricingStrategy` (interface).

## Interfaces
```java
interface PricingStrategy {
    BigDecimal price(Duration parked, SpotType type);
}

interface SpotAssignmentStrategy {
    Optional<ParkingSpot> assign(Vehicle vehicle, List<Level> levels);
}
```

## Classes & relationships
```java
class ParkingLot {
    private final List<Level> levels;
    private final SpotAssignmentStrategy assignment;
    private final PricingStrategy pricing;

    public Ticket park(Vehicle vehicle) {
        ParkingSpot spot = assignment.assign(vehicle, levels)
            .orElseThrow(() -> new NoSpotAvailableException());
        spot.occupy(vehicle);
        return new Ticket(spot, Instant.now());
    }

    public BigDecimal unpark(Ticket ticket) {
        Duration parked = Duration.between(ticket.entryTime(), Instant.now());
        ticket.spot().free();
        return pricing.price(parked, ticket.spot().type());
    }
}
```

## Design patterns
- **Strategy** for pricing and spot assignment (open/closed).
- **Factory** for creating vehicles/spots.
- **State** could model ticket lifecycle.

## SOLID principles
- SRP: pricing, assignment, and lot management are separate.
- OCP: add new pricing/assignment without changing `ParkingLot`.
- DIP: `ParkingLot` depends on strategy interfaces, not concretions.

## Extensibility
New spot types, pricing (surge, subscriptions), and multi-lot support slot in via strategies.

## Java implementation approach
Immutable value objects where possible; enums for `SpotType`; thread-safe spot occupation under concurrency ([[ConcurrentHashMap]] / locks).

## Common interview follow-ups
- Concurrency: two cars racing for the last spot? (atomic occupy / lock per spot)
- How to find the nearest available spot efficiently?
- Handling lost tickets and payments.

## Related concepts
- [[SOLID]]
- [[Design-Patterns]]
