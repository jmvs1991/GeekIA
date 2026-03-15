---
name: geek-dotnet-dto-refactor
description: refactor an existing dotnet dto using its related entity so it matches the geek dto pattern. use when a c# dto already exists but does not follow the expected inheritance, mutability, transform function, or mapping override conventions. use for entity-backed dto and view dto refactors where chatgpt should regenerate a clean dto with dotnet new, migrate the existing dto logic, preserve current business mapping behavior, and replace the old file only after migration is complete.
---

# Geek Dotnet Dto Refactor

Refactor an existing DTO by using the related Entity as the source of truth for base inheritance and structure, while preserving the DTO's current mapping logic.

## Inputs

Expect these inputs before editing:

- The existing DTO file
- The related Entity file
- Whether the DTO is an `entity dto` or a `view dto`
- `projectName` for the `dotnet new` command
- Optionally, confirmation of the target path if it is ambiguous

Infer as much as possible from the files before asking follow-up questions.

## Workflow overview

1. Read the DTO and related Entity.
2. Infer the DTO base pattern from the Entity.
3. Choose the correct `dotnet new` template.
4. Generate a clean DTO when command execution is available.
5. Migrate the useful logic from the old DTO into the generated DTO.
6. Preserve behavior in `MapDtoToEntity` and `MapEntityToDto`.
7. Replace the old DTO only after the migrated DTO is complete.
8. Report the applied changes and any uncertain parts.

## Determine the DTO base

Use the Entity relationship to select the DTO parent type.

- If the related Entity has `Company`, the DTO must inherit from `DTOWithCorporationAndCompany<TEntity>` and implement `IDTOWithCorporationAndCompany<TEntity>`.
- Else if the related Entity has `Corporation`, the DTO must inherit from `DTOWithCorporation<TEntity>` and implement `IDTOWithCorporation<TEntity>`.
- Else the DTO must inherit from `DTO<TEntity>` and implement `IDTO<TEntity>`.

Do not infer the DTO base from the current DTO alone. The Entity is the source of truth.

## DTO naming

If the Entity is `ClientEntity`, the DTO must be `ClientDTO`.

General rule:

- Strip the `Entity` suffix from the Entity class name.
- Append `DTO`.

## Template selection

The user indicates whether the target is a regular dto or a view dto.

### Entity-backed DTO templates

- No `Corporation` and no `Company`:
  `dotnet new geek-dto -n <DtoName> --projectName <ProjectName>`
- Has `Corporation` but not `Company`:
  `dotnet new geek-dto-corp -n <DtoName> --projectName <ProjectName>`
- Has both `Corporation` and `Company`:
  `dotnet new geek-dto-corp-co -n <DtoName> --projectName <ProjectName>`

### View DTO templates

- No `Corporation` and no `Company`:
  `dotnet new geek-view-dto -n <DtoName> --projectName <ProjectName>`
- Has `Corporation` but not `Company`:
  `dotnet new geek-view-dto-corp -n <DtoName> --projectName <ProjectName>`
- Has both `Corporation` and `Company`:
  `dotnet new geek-view-dto-corp-co -n <DtoName> --projectName <ProjectName>`

## Execution rules

When the environment allows command execution:

1. Generate the new DTO using the correct `dotnet new` template.
2. Use the generated DTO as the structural base.
3. Migrate the old DTO logic into the generated DTO.
4. Delete or replace the original DTO only after the migrated DTO is complete.

When the environment does not allow command execution:

- Produce the exact command to run.
- Produce a file-level diff or the final DTO content.
- Clearly mark what should replace the old file.

## Required DTO shape after refactor

### Usings

Always correct DTO imports when needed.

For base DTO inheritance, use:

- `ServiceDomain.Abstraction.DTOs`
- `SolutionDomain.Domains.Services.Interfaces.DTOs`

Also correct the Entity namespace if needed. For example, fix outdated imports like:

- `Connection.ClientEntities`

into the correct pattern such as:

- `Connection.Client.Entities`

Preserve any additional DTO-specific usings that are still needed.

### Class signature

Rewrite the full class signature when the current DTO does not follow the expected pattern.

Examples:

```csharp
public class ClientDTO : DTO<ClientEntity>, IDTO<ClientEntity>
```

```csharp
public class BuildingDTO : DTOWithCorporationAndCompany<BuildingEntity>, IDTOWithCorporationAndCompany<BuildingEntity>
```

### Properties

All DTO properties must use `private set`.

Example:

```csharp
public string Name { get; private set; } = "";
```

Do not leave mutable public setters.

### Constructors

Keep:

1. An empty constructor that calls `base()`.
2. A constructor receiving the related Entity that calls `base(entity)`.

Example:

```csharp
public ClientDTO() : base()
{
}

public ClientDTO(ClientEntity clientEntity) : base(clientEntity)
{
}
```

### Transform function

Keep exactly one static transform selector as a `Func<TEntity, TDto>`.

Use the lowercase name `transform`.

Example:

```csharp
public static readonly Func<ClientEntity, ClientDTO> transform = clientEntity => new ClientDTO(clientEntity);
```

Remove old selectors such as:

- `singleSelect`
- `SingleSelect`
- `complexSelect`
- `ComplexSelect`
- spelling variants of those names

Do not keep compatibility wrappers.

### Mapping overrides

The DTO must override the inherited mapping methods.

#### MapDtoToEntity

Rules:

- Must be `override`
- Must call `base.MapDtoToEntity()` first
- Must preserve current DTO-specific mapping logic
- Must assign raw values only
- Must not add encryption or decryption work here

Pattern:

```csharp
public override ClientEntity MapDtoToEntity()
{
    ClientEntity entity = base.MapDtoToEntity();
    entity.Codclient = Codclient;
    entity.Name = Name;
    return entity;
}
```

#### MapEntityToDto

Rules:

- Must be `override`
- Must call `base.MapEntityToDto(entity)` first
- Must preserve current DTO-specific mapping logic
- Must remove encryption and decryption responsibilities from the DTO
- Must assign raw values from the Entity

Pattern:

```csharp
public override void MapEntityToDto(ClientEntity entity)
{
    base.MapEntityToDto(entity);
    Name = entity.Name;
}
```

### Collections and related DTO mapping

Preserve the existing logic structure where possible.

When refactoring collection mappings inside the current DTO, replace references such as:

```csharp
ClientContactDTO.singleSelect
```

with:

```csharp
ClientContactDTO.transform
```

Only adjust references inside the current DTO being refactored. Do not edit the related DTO files as part of this skill.

## Preserve existing behavior

This skill is a refactor, not a rewrite of business behavior.

Preserve the current logic wherever possible, especially in:

- property-to-property mappings
- enum conversions
- collection population
- object initialization that belongs to the DTO behavior

Refactor structure and responsibilities, but do not silently remove useful mapping logic.

## Encryption and decryption rule

If the current DTO encrypts or decrypts values, remove that responsibility from the DTO.

For example, convert this:

```csharp
Document = Encryption.Decrypt(Configuration.EncryptionKey, clientEntity.Document);
```

into raw assignment:

```csharp
Document = clientEntity.Document;
```

The encryption or decryption process belongs to another stage.

## Output requirements

When showing the result, provide:

1. The exact `dotnet new` command used or to be used.
2. A concise summary of the selected DTO base.
3. The key structural changes applied.
4. The final DTO content or a precise diff.
5. Any ambiguity that still needs a human decision.

## Representative example

### Before

```csharp
public class ClientDTO : IMap<ClientEntity>
{
    public string Name { get; set; } = "";
    public static readonly Func<ClientEntity, ClientDTO> singleSelect = clientEntity => new ClientDTO(clientEntity);

    public ClientEntity MapDtoToEntity()
    {
        ClientEntity clientEntity = new ClientEntity()
        {
            Name = Name
        };

        return clientEntity;
    }

    public void MapEntityToDto(ClientEntity clientEntity)
    {
        Name = clientEntity.Name;
    }
}
```

### After

```csharp
public class ClientDTO : DTO<ClientEntity>, IDTO<ClientEntity>
{
    public string Name { get; private set; } = "";

    public static readonly Func<ClientEntity, ClientDTO> transform = clientEntity => new ClientDTO(clientEntity);

    public ClientDTO() : base()
    {
    }

    public ClientDTO(ClientEntity clientEntity) : base(clientEntity)
    {
    }

    public override ClientEntity MapDtoToEntity()
    {
        ClientEntity entity = base.MapDtoToEntity();
        entity.Name = Name;
        return entity;
    }

    public override void MapEntityToDto(ClientEntity clientEntity)
    {
        base.MapEntityToDto(clientEntity);
        Name = clientEntity.Name;
    }
}
```
