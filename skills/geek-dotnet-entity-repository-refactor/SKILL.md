---
name: geek-dotnet-entity-repository-refactor
description: refactor an existing c# entity and its schema dbcontext to match the connection-layer conventions, then create matching read and write repositories. use when chatgpt is given an existing entity file that already declares its schema but does not follow the expected base class and interface pattern, when the schema dbcontext needs cleanup, or when read and write repositories must be generated for that entity. especially useful for connection schema entities paths files, schema contexts like parkingcontext or condominiumcontext, and repository generation based on existing entities in connection folders.
---

# Entity Repository Refactor

Refactor an existing entity to the connection-layer pattern, clean the schema context, and create read and write repositories. Work from the entity file itself. The entity already indicates the schema through its attributes, namespace, or surrounding folder structure.

## Workflow overview

1. Inspect the existing entity and determine its schema, base type tier, and clean entity name.
2. Refactor the entity inheritance and implemented interface only.
3. Clean the schema context by removing the entity `DbSet<>` and ensuring the builder pattern remains or is added consistently.
4. Create the repository folder structure for that entity.
5. Generate read and write repositories by running the appropriate `dotnet new` commands when the environment allows.
6. If commands cannot be executed, return the exact commands and a concrete patch/diff for the entity and context.

## Inputs to expect

The minimum useful input is the existing entity file.

Prefer to infer these values from the file and path instead of asking:
- schema name from `[Table(..., Schema = "...")]`, namespace, or folder path
- entity base name from the class name after removing the `Entity` suffix if present
- connection root from the current project structure
- context name as `<Schema>Context`

Only ask for clarification when a required value cannot be inferred reliably.

## Step 1: inspect and classify the entity

Use the entity file as the source of truth.

### Determine the schema
Use the first reliable signal available:
1. `[Table(..., Schema = "SchemaName")]`
2. namespace such as `Connection.Condominium.Entities`
3. folder path such as `connection/Condominium/Entities`

### Determine the clean entity name
- Remove a trailing `Entity` suffix from the class name if present.
- If the original table-derived name contains prefixes like `TR_` or `MR_`, remove those prefixes when deriving or confirming the clean name.
- Convert the remaining name to PascalCase.

Examples:
- `TR_TAG_INVOICE` -> `TagInvoice`
- `MR_BUILDING` -> `Building`
- `BuildingEntity` -> clean name `Building`

### Determine the inheritance tier
Use these rules in order:
1. If the entity has a `Company` property/column, it belongs to the corporation-and-company tier.
2. Otherwise, if it has a `Corporation` property/column, it belongs to the corporation tier.
3. Otherwise, it belongs to the base entity tier.

Apply these mappings:
- company present -> inherit `EntityWithCorporationAndCompany` and implement `IEntityWithCorporationAndCompany`
- corporation present without company -> inherit `EntityWithCorporation` and implement `IEntityWithCorporation`
- neither present -> inherit `Entity` and implement `IEntity`

## Step 2: refactor the entity

Only correct the inheritance and implemented interface for this skill.

Do not use this skill to rewrite unrelated parts of the entity unless needed to keep the inheritance line syntactically valid.

### Entity refactor rules
- Keep the existing class name unless the user explicitly asks for renaming.
- Preserve properties, attributes, navigation properties, and builders already in the file.
- Replace or add the correct base class and interface according to the tier rules above.
- Prefer proposing an explicit diff when editing is possible.

Example target shape:
```csharp
public partial class BuildingEntity : EntityWithCorporationAndCompany, IEntityWithCorporationAndCompany
```

## Step 3: clean the schema context

The schema context should be `<Schema>Context`, for example `ParkingContext` or `CondominiumContext`.

Work on the existing schema context if it already exists. Create it only if it does not exist.

### Context cleanup rules
- Remove any `DbSet<ThisEntity>` property from the context.
- Keep the context aligned with the builder-based pattern.
- Ensure there is a private readonly builder delegate field for the entity when the entity exposes a static builder.
- Ensure `OnModelCreating` includes `.Entity(_ThisEntityBuilder)` in the fluent chain.
- Preserve existing builders for other entities.
- Keep `modelBuilder.HasDefaultSchema("<Schema>")` intact or add it if the context is being created.
- Keep the context clean and consistent; do not leave both a `DbSet<>` entry and the builder-chain registration for the same entity.

Use this pattern as the target style:
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

## Step 4: create repository structure

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

If the entity class is `BuildingEntity`, create:
```text
connection/<Schema>/Repositories/BuildingEntity/Read
connection/<Schema>/Repositories/BuildingEntity/Write
```

## Step 5: generate repositories

Use the clean entity name for `-n`, not the full class name with `Entity`.

Example:
- class `BuildingEntity`
- `-n Building`

### Read repository command selection
Choose exactly one template.

For entity repositories:
- no corporation and no company:
  ```bash
  dotnet new geek-read-paginationful-filterless-entity --dbSchema <schema> --contextName <Schema>Context -n <Name>
  ```
- corporation only:
  ```bash
  dotnet new geek-read-paginationful-filterless-entity-corp --dbSchema <schema> --contextName <Schema>Context -n <Name>
  ```
- corporation and company:
  ```bash
  dotnet new geek-read-paginationful-filterless-entity-corp-co --dbSchema <schema> --contextName <Schema>Context -n <Name>
  ```
- corporation, company, and code field:
  ```bash
  dotnet new geek-read-paginationful-filterless-entity-corp-co-code --dbSchema <schema> --contextName <Schema>Context -n <Name>
  ```

### How to detect the code-field case
Prefer this heuristic:
- the entity is in the company tier
- the primary key is effectively ternary: `Corporation`, `Company`, and one additional key property
- that additional key property is the code field

If this cannot be inferred confidently, say so and either:
- ask the user to confirm the code field, or
- generate the non-code corporation/company template and note the ambiguity

### Write repository command
Always use:
```bash
dotnet new geek-write-storeprocedureful-transactionless --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

## Step 6: execution vs fallback

When the environment allows command execution:
- execute the `dotnet new` commands from the correct `Read` and `Write` directories
- then verify that the expected files were created

Expected naming pattern:
- `IBuilding.read.repository.cs` -> interface `IBuildingReadRepository`
- `Building.read.repository.cs` -> class `BuildingReadRepository`
- `IBuilding.write.repository.cs` -> interface `IBuildingWriteRepository`
- `Building.write.repository.cs` -> class `BuildingWriteRepository`

When the environment does not allow execution:
- provide the exact commands to run
- provide a focused diff for the entity inheritance/interface change
- provide a focused diff for the context cleanup
- show the exact folder structure that should exist

## Output expectations

Default to this order in the response:
1. Short summary of what tier the entity belongs to
2. Entity diff
3. Context diff
4. Repository commands or execution results
5. Resulting file structure
6. Any ambiguity that still needs confirmation

## References

Use these bundled references when needed:
- `references/entity-rules.md` for inheritance and naming rules
- `references/context-rules.md` for schema context cleanup
- `references/repository-rules.md` for repository paths and command selection
