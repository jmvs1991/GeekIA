---
name: geek-dotnet-entity-repository
description: generate ef core scaffold commands, transform scaffolded sql server tables into connection schema entities, and create matching read and write repository commands and file structures. use when chatgpt needs to help with sql server table-to-entity scaffolding, schema-specific connection folders, entity inheritance conventions, context cleanup, or repository generation with geek dotnet templates.
---

# Connection entity and repository generation

Use this skill to turn a SQL Server table into:
1. a scaffold command for a schema-specific EF Core entity
2. a transformed `*Entity.cs` file that follows the Connection conventions
3. the expected schema context updates
4. read and write repository commands and folder structure

## Expected input

Collect or infer these inputs:
- connection string
- schema name, for example `Parking` or `Condominium`
- table name, for example `TR_TAG_INVOICE` or `MR_BUILDING`
- context name; default to `<Schema>Context` when not provided explicitly
- whether commands may be executed in the current environment
- table column list and key information if available

If the user does not provide the column list, still generate the scaffold and repository commands. Make clear which inheritance or read-template choice depends on columns that still need verification.

## Output expectations

When execution is possible, execute the commands. When execution is not possible, return the exact commands to run.

Always provide:
- the normalized entity name
- the scaffold command
- the target file and folder paths
- the chosen entity inheritance and interface
- the chosen read repository template
- the write repository command
- any follow-up manual checks still required

## Naming rules

### Table to entity name
- Remove leading `TR_` and `MR_` prefixes.
- Convert the remaining name to PascalCase.
- Append `Entity` to the class name.
- Use the base PascalCase name without `Entity` for repository template `-n` values.

Examples:
- `TR_TAG_INVOICE` -> `TagInvoiceEntity` and repository name `TagInvoice`
- `MR_BUILDING` -> `BuildingEntity` and repository name `Building`

### File names
- Entity file: `<EntityName>.cs`
- Read interface file: `I<BaseName>.read.repository.cs`
- Read implementation file: `<BaseName>.read.repository.cs`
- Write interface file: `I<BaseName>.write.repository.cs`
- Write implementation file: `<BaseName>.write.repository.cs`

Class names inside those files:
- `I<BaseName>ReadRepository`
- `<BaseName>ReadRepository`
- `I<BaseName>WriteRepository`
- `<BaseName>WriteRepository`

## Folder layout

Work under the `connection` folder.

For schema `<Schema>` and base name `<BaseName>`:

```text
connection/
└── <Schema>/
    ├── <Schema>Context.cs
    ├── Entities/
    │   └── <BaseName>Entity.cs
    └── Repositories/
        └── <BaseName>Entity/
            ├── Read/
            │   ├── I<BaseName>.read.repository.cs
            │   └── <BaseName>.read.repository.cs
            └── Write/
                ├── I<BaseName>.write.repository.cs
                └── <BaseName>.write.repository.cs
```

`Repositories` is always at the same level as `Entities` inside the schema folder.

## Workflow

### 1. Build the scaffold command
Always use `--data-annotations`.

Run or propose:

```bash
dotnet ef dbcontext scaffold "<connection-string>" Microsoft.EntityFrameworkCore.SqlServer --table <TABLE_NAME> --data-annotations --output-dir connection/<Schema>/Entities
```

After scaffolding:
- delete the generated context file from the scaffold output
- keep or create the schema context file named `<Schema>Context.cs`
- keep the context clean and schema-specific

Default context naming:
- `Parking` -> `ParkingContext`
- `Condominium` -> `CondominiumContext`

### 2. Choose entity inheritance
Apply this decision order:

1. If the table has `COMPANY`, it also has `CORPORATION`.
   - Base class: `EntityWithCorporationAndCompany`
   - Interface: `IEntityWithCorporationAndCompany`
2. Else if the table has `CORPORATION`.
   - Base class: `EntityWithCorporation`
   - Interface: `IEntityWithCorporation`
3. Else:
   - Base class: `Entity`
   - Interface: `IEntity`

### 3. Preserve base audit columns
Tables commonly include these base columns:
- `ID`
- `CORPORATION`
- `COMPANY`
- `CREATION_DATE`
- `CREATION_DATEL`
- `UPDATE_DATE`
- `UPDATE_DATEL`
- `CREATE_USER`
- `UPDATE_USER`

Keep them aligned with the selected base entity type.

### 4. Transform the scaffolded entity
The final entity should:
- use `[Table("<TABLE>", Schema = "<Schema>")]`
- use the normalized `*Entity` class name
- inherit and implement the correct base type pair
- keep the scaffolded data annotations
- expose a static builder action named `<BaseName>Builder`
- define key and property configuration in the static builder
- keep navigation properties as appropriate

Use the example style from `references/entity-pattern.md`.

### 5. Update or create the schema context
The schema context should:
- live at `connection/<Schema>/<Schema>Context.cs`
- use `namespace Connection.<Schema>` unless the user specifies a different root namespace
- import `Connection.<Schema>.Entities`
- hold one private readonly builder delegate per entity, named `_<BaseName>Builder`
- call `modelBuilder.HasDefaultSchema("<Schema>")`
- register entities through chained `.Entity(_<BaseName>Builder)` calls in `OnModelCreating`

When editing an existing context, preserve existing delegates and entity registrations and append the new entity cleanly.

### 6. Choose the read repository template
Select from these templates based on the table shape:

- No `CORPORATION` and no `COMPANY`:
  `geek-read-paginationful-filterless-entity`
- `CORPORATION` only:
  `geek-read-paginationful-filterless-entity-corp`
- `CORPORATION` and `COMPANY`:
  `geek-read-paginationful-filterless-entity-corp-co`
- `CORPORATION` and `COMPANY`, plus a third primary-key column that acts as the code field:
  `geek-read-paginationful-filterless-entity-corp-co-code`

The `code` case usually means the primary key is a ternary key composed of:
- `CORPORATION`
- `COMPANY`
- one additional key column

If the additional code field cannot be determined confidently, say so and ask the user to confirm the code field while still showing the most likely command.

Generate or execute the command from the entity-specific read folder:

```bash
dotnet new <read-template> --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

### 7. Create the write repository
Generate or execute this command from the entity-specific write folder:

```bash
dotnet new geek-write-storeprocedureful-transactionless --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

## Response format

Use this structure unless the user asks for a different format:

### Summary
- entity name
- context name
- selected entity base type
- selected read template

### Commands
Show the scaffold command, read command, and write command.

### Files
List the expected entity, context, and repository file paths.

### Notes
Mention any assumptions, especially:
- missing column metadata
- uncertain code-field detection
- whether commands were executed or only generated

## Execution mode

When the environment allows execution:
- run the scaffold command
- remove the scaffold-generated context file
- run the read and write template commands in their target folders
- report what was created or modified

When the environment does not allow execution:
- do not pretend to have run anything
- provide the exact commands and paths
- explain what the user should verify next
