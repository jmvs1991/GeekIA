# Entity Rules

## Goal
Refactor an existing entity so its inheritance and interface match the expected connection-layer tier.

## Tier rules

### Tier 1: company tier
Use when the entity contains `Company`.

Required declaration:
```csharp
public partial class <Name>Entity : EntityWithCorporationAndCompany, IEntityWithCorporationAndCompany
```

### Tier 2: corporation tier
Use when the entity contains `Corporation` but not `Company`.

Required declaration:
```csharp
public partial class <Name>Entity : EntityWithCorporation, IEntityWithCorporation
```

### Tier 3: base tier
Use when the entity contains neither `Corporation` nor `Company`.

Required declaration:
```csharp
public partial class <Name>Entity : Entity, IEntity
```

## Naming
- Remove `TR_` and `MR_` prefixes when deriving the clean logical name from a table-derived name.
- Convert to PascalCase.
- Use the clean name for repository generator `-n`.
- Keep the existing entity class filename unless the user explicitly asks for renaming.
