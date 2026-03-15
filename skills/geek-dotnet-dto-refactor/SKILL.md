---
name: geek-dotnet-dto-refactor
description: refactor existing c# dto files to match the team's dto pattern for entity-backed models. use when the user provides a dto file and its related entity and wants to correct dto inheritance, rename the dto to match the entity name without the entity suffix, make properties use private setters, consolidate old static mapping methods like singleselect or complexselect into a single transform method, and preserve existing mapping logic while standardizing mapdtotoentity and mapentitytodto overrides.
---

# DTO refactor

Refactor the provided DTO and related Entity to the team's DTO pattern.

Follow this workflow:

1. Inspect the provided DTO and related Entity.
2. Determine the correct DTO base type from the Entity:
   - If the Entity has `Company`, use `DTOWithCorporationAndCompany`.
   - Else if the Entity has `Corporation`, use `DTOWithCorporation`.
   - Else use `DTO`.
3. Rename the DTO class to match the Entity name without the `Entity` suffix and with a `DTO` suffix.
   - Example: `ClientEntity` -> `ClientDTO`
4. Rewrite the class signature completely when needed so inheritance matches the correct base DTO type.
5. Refactor properties so they keep public getters and use `private set`.
   - Example: `public string Name { get; private set; }`
6. Preserve the DTO's existing business mapping logic while standardizing method names and structure.

## Constructors

The DTO should have:

- A parameterless constructor.
- A constructor that receives the related Entity and maps from it.

Default pattern:

```csharp
public BuildingDTO()
{
}

public BuildingDTO(BuildingEntity entity)
{
    MapEntityToDto(entity);
}
```

## Static transform method

Remove legacy static mapping methods such as `SingleSelect`, `singleSelect`, `ComplexSelect`, `ComplexSelec`, and similar variations.

Leave exactly one static method:

```csharp
public static BuildingDTO Transform(BuildingEntity entity)
{
    return new BuildingDTO(entity);
}
```

Rules:

- `Transform` should not accept null.
- Do not keep compatibility wrappers for old static method names.
- If the DTO maps nested DTO collections, update calls like `UnitDTO.singleSelect(...)` to `UnitDTO.Transform(...)`.

## MapDtoToEntity override

Preserve existing child-field mapping logic, but standardize the override so it calls `base.MapDtoToEntity()` first and then fills child-specific fields.

Pattern:

```csharp
public override BuildingEntity MapDtoToEntity()
{
    BuildingEntity entity = base.MapDtoToEntity();
    entity.Codbuilding = Codbuilding;
    entity.Description = Description;
    entity.Type = Type;
    entity.Units = new List<UnitEntity>();

    return entity;
}
```

Rules:

- Keep existing field assignments unless they clearly violate the DTO pattern.
- Preserve collection and navigation mapping intent.
- For properties that normally involve encryption or decryption, assign the raw value only. Do not add encryption or decryption logic in the DTO.

## MapEntityToDto override

Preserve existing child-field mapping logic, but standardize the override so it calls `base.MapEntityToDto(entity)` first and then fills child-specific fields.

Pattern:

```csharp
public override void MapEntityToDto(BuildingEntity entity)
{
    base.MapEntityToDto(entity);
    Codbuilding = entity.Codbuilding;
    Description = entity.Description;
    Type = entity.Type;
    Units = entity.Units != null && entity.Units.Any()
        ? entity.Units.Select(UnitDTO.Transform).ToList()
        : new List<UnitDTO>();
}
```

Rules:

- Preserve the DTO's existing mapping logic when possible.
- Refactor method names and setter access, but do not remove useful domain mapping.
- If existing mapping code is already correct in behavior, prefer minimal edits.

## Output expectations

When editing the DTO:

- Show the refactored class or a focused diff, depending on what the user asked for.
- Explain the important changes briefly.
- Do not modify files other than the DTO unless the user explicitly asks.

## Related reference

For the full checklist and examples, consult `references/dto-rules.md`.
