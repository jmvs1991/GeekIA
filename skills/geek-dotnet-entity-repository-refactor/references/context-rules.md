# Context Rules

## Context name
The schema context should be named `<Schema>Context`, for example:
- `ParkingContext`
- `CondominiumContext`

## Cleanup rules
- Remove `DbSet<ThisEntity>` from the schema context.
- Keep or add `using Connection.<Schema>.Entities;`.
- Keep the `DbContextOptions<<Schema>Context>` constructor.
- Register the entity using the builder delegate pattern rather than a `DbSet<>`.
- Ensure `OnModelCreating` calls `modelBuilder.HasDefaultSchema("<Schema>")`.
- Ensure the entity builder is part of the fluent `.Entity(...)` chain.

## Example builder field
```csharp
private readonly Action<EntityTypeBuilder<BuildingEntity>> _BuildingBuilder = BuildingEntity.BuildingBuilder;
```

## Example registration
```csharp
modelBuilder.Entity(_BuildingBuilder);
```
