# DTO refactor rules

## Base DTO selection

Choose the DTO base by inspecting the related Entity.

1. If the Entity has `Company`, the DTO should use `DTOWithCorporationAndCompany`.
2. Else if the Entity has `Corporation`, the DTO should use `DTOWithCorporation`.
3. Else use `DTO`.

Because DTOs and Entities are tightly related, if the Entity has `Company`, the DTO should also include `Company`. If the Entity has `Corporation`, the DTO should also include `Corporation`.

## Naming

- Entity: `ClientEntity`
- DTO: `ClientDTO`

Rename the DTO if it does not follow this pattern.

## Property mutability

Convert mutable properties to public getter plus private setter.

Preferred pattern:

```csharp
public string Name { get; private set; } = null!;
```

## Required methods

The DTO should end with these core members:

- parameterless constructor
- constructor that receives the related Entity
- `public static <DtoName> Transform(<EntityName> entity)`
- `public override <EntityName> MapDtoToEntity()`
- `public override void MapEntityToDto(<EntityName> entity)`

## Legacy static methods

Delete legacy names such as:

- `SingleSelect`
- `singleSelect`
- `ComplexSelect`
- `ComplexSelec`
- other casing or spelling variations of the same idea

Replace them with exactly one method named `Transform`.

## Mapping preservation

Preserve existing mapping behavior whenever possible.

Good refactors:

- renaming helper methods to `Transform`
- changing setters to `private set`
- moving parent-field population to `base.MapDtoToEntity()` and `base.MapEntityToDto(entity)`
- keeping nested collection mapping behavior

Avoid these mistakes:

- removing meaningful child-field assignments
- adding encryption or decryption logic in the DTO
- changing domain behavior unrelated to the DTO pattern

## Example transformation targets

### Constructor + transform

```csharp
public BuildingDTO()
{
}

public BuildingDTO(BuildingEntity entity)
{
    MapEntityToDto(entity);
}

public static BuildingDTO Transform(BuildingEntity entity)
{
    return new BuildingDTO(entity);
}
```

### Child-to-entity mapping

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

### Entity-to-child mapping

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
