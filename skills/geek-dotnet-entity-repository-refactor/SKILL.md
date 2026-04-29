---
name: geek-dotnet-entity-repository-refactor
description: refactor an existing c# entity and schema dbcontext to match connection-layer conventions, then create matching read and write repositories using the geek-cli mcp when available. use when given an existing entity file that declares its schema but needs corrected base class/interface, schema context cleanup, or repository generation for connection schema entities such as parkingcontext or condominiumcontext.
---

# Entity Repository Refactor

Refactor an existing entity to the connection-layer pattern, clean the schema context, and create read and write repositories. Work from the entity file itself. The entity already indicates the schema through its attributes, namespace, or surrounding folder structure.

## Workflow overview

1. Inspect the entity and determine schema, clean entity name, and inheritance tier.
2. Refactor only the entity inheritance and implemented interface.
3. Clean the schema context by removing the entity `DbSet<>` and keeping builder registration.
4. Create the repository folder structure.
5. Generate repositories using the `geek-cli` MCP tools `DotnetRead` and `DotnetWrite` when available.
6. Use `dotnet new ...` commands only as fallback when `geek-cli` is unavailable.

## Inputs to infer

The minimum useful input is the existing entity file.

Prefer to infer these values from the file and path instead of asking:
- schema from `[Table(..., Schema = "...")]`, namespace, or folder path
- clean entity name from the class name after removing the trailing `Entity` suffix if present
- connection root from the current project structure
- context name as `<Schema>Context`

Only ask for clarification when required values cannot be inferred reliably.

## Step 1: inspect and classify

Use the entity file as the source of truth.

### Schema

Use the first reliable signal:

1. `[Table(..., Schema = "SchemaName")]`
2. namespace like `Connection.Condominium.Entities`
3. folder path like `connection/Condominium/Entities`

### Clean entity name

- Remove a trailing `Entity` suffix from the class name if present.
- Remove table prefixes like `TR_` or `MR_` when deriving or confirming the clean name.
- Convert the remaining name to PascalCase.

Examples:

- `TR_TAG_INVOICE` -> `TagInvoice`
- `MR_BUILDING` -> `Building`
- `BuildingEntity` -> `Building`

### Inheritance tier

Use these rules in order:

1. If the entity has a `Company` property or column, use the corporation-and-company tier.
2. Otherwise, if the entity has a `Corporation` property or column, use the corporation tier.
3. Otherwise, use the base entity tier.

Mappings:

- company present -> inherit `EntityWithCorporationAndCompany` and implement `IEntityWithCorporationAndCompany`
- corporation present without company -> inherit `EntityWithCorporation` and implement `IEntityWithCorporation`
- neither present -> inherit `Entity` and implement `IEntity`

## Step 2: refactor entity

Only correct the inheritance and implemented interface for this skill.

Do not use this skill to rewrite unrelated parts of the entity unless needed to keep the inheritance line syntactically valid.

Preserve:

- class name unless the user explicitly asks for renaming
- properties
- attributes
- navigation properties
- builders already in the file

Replace or add the correct base class and interface according to the tier rules above.

Example target shape:

```csharp
public partial class BuildingEntity : EntityWithCorporationAndCompany, IEntityWithCorporationAndCompany
```

## Step 3: clean schema context

The schema context should be `<Schema>Context`, for example `ParkingContext` or `CondominiumContext`.

Work on the existing schema context if it already exists. Create it only if it does not exist.

Rules:

- Remove any `DbSet<ThisEntity>` property from the context.
- Keep the context aligned with the builder-based pattern.
- If the entity exposes a static builder, ensure there is a private readonly builder delegate field for the entity.
- Ensure `OnModelCreating` includes `.Entity(_ThisEntityBuilder)` in the fluent chain.
- Preserve existing builders for other entities.
- Keep or add `modelBuilder.HasDefaultSchema("<Schema>")`.
- Do not leave both a `DbSet<>` entry and builder-chain registration for the same entity.

Target style:

```csharp
using Connection.Condominium.Entities;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace Connection.Condominium
{
    public class CondominiumContext : DbContext
    {
        private readonly Action<EntityTypeBuilder<BuildingEntity>> _BuildingBuilder = BuildingEntity.BuildingBuilder;

        public CondominiumContext(DbContextOptions<CondominiumContext> options) : base(options) { }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.HasDefaultSchema("Condominium");
            modelBuilder.Entity(_BuildingBuilder);
        }
    }
}
```

## Step 4: repository structure

Repositories live beside `Entities` under the same schema folder.

Expected structure:

```text
connection/
└── <Schema>/
    ├── Entities/
    │   └── <EntityClass>.cs
    └── Repositories/
        └── <EntityClass>/
            ├── Read/
            └── Write/
```

Example for `BuildingEntity`:

```text
connection/<Schema>/Repositories/BuildingEntity/Read
connection/<Schema>/Repositories/BuildingEntity/Write
```

## Step 5: generate repositories with geek-cli MCP

When the `geek-cli` MCP is available, prefer it over shell commands.

Use the clean entity name for `name`, not the full class name with `Entity`.

Example:

- class `BuildingEntity`
- `name = "Building"`

### Read repository

Call:

```csharp
geek-cli.DotnetRead(name, dbSchema, contextName, scope, view)
```

Arguments:

```text
name = <CleanEntityName>
dbSchema = <Schema>
contextName = <Schema>Context
```

Map tier to `scope`:

- no corporation and no company -> `"basic"`
- corporation only -> `"corp"`
- corporation and company -> `"corp-co"`
- corporation, company, and code field -> `"corp-co-code"`

Use `view` to indicate whether the read repository is for a database view or for an entity/table:

- `view = false` for entity/table repositories
- `view = true` for database view repositories

For this skill, default to:

```text
view = false
```

because the workflow targets existing entities.

Set `view = true` only when the user explicitly says the repository is for a database view, or when the source file or convention clearly indicates a view-based model.

### Code-field case

Use `scope = "corp-co-code"` only when:

- the entity is in the corporation-and-company tier
- the effective key is `Corporation`, `Company`, and one additional key property
- that additional key property is the code field

If this cannot be inferred confidently, use `scope = "corp-co"` and mention the ambiguity.

### Write repository

Always call:

```csharp
geek-cli.DotnetWrite(name, dbSchema, contextName)
```

Arguments:

```text
name = <CleanEntityName>
dbSchema = <Schema>
contextName = <Schema>Context
```

## Step 6: fallback dotnet commands

Only use these commands when the `geek-cli` MCP is unavailable.

When fallback commands can be executed:
- execute the `dotnet new` commands from the correct `Read` and `Write` directories
- then verify that the expected files were created

When fallback commands cannot be executed:
- provide the exact commands to run
- provide a focused diff for the entity inheritance/interface change
- provide a focused diff for the context cleanup
- show the exact folder structure that should exist

### Read fallback

Base entity:

```bash
dotnet new geek-read-paginationful-filterless-entity --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

Corporation:

```bash
dotnet new geek-read-paginationful-filterless-entity-corp --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

Corporation and company:

```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

Corporation, company, and code:

```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co-code --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

### Write fallback

```bash
dotnet new geek-write-storeprocedureful-transactionless --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

Expected fallback naming pattern:

- `IBuilding.read.repository.cs` -> interface `IBuildingReadRepository`
- `Building.read.repository.cs` -> class `BuildingReadRepository`
- `IBuilding.write.repository.cs` -> interface `IBuildingWriteRepository`
- `Building.write.repository.cs` -> class `BuildingWriteRepository`

## Output expectations

Return results in this order:

1. Short summary of the entity tier
2. Entity diff
3. Context diff
4. `geek-cli` MCP calls/results, or fallback commands if MCP is unavailable
5. Resulting file structure
6. Any ambiguity that still needs confirmation
