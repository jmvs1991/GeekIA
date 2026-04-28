---
name: geek-dotnet-db-migration
description: manage dotnet database migration and database script tasks through the geek cli mcp tools. use this skill when the user asks to create, add, remove, rollback, or script database migrations, or makes any request related to database schema changes in a dotnet project that uses *.context, *.manager, *.schemainitialization, and *.schemaupdates project structure. validate the dotnet workspace, derive projectname only from the required project structure when unambiguous, ask for every missing tool parameter, and report executed actions, parameters, generated files, paths, and summaries.
---

# Geek Dotnet DB Migration

## Objective

Use the Geek CLI MCP tools to manage database migration work for .NET projects. This skill is for any AI agent using the available MCP tools, not only ChatGPT.

Supported MCP tools and exact signatures:

```csharp
DbMigrationAdd(string projectName, string migrationName, string issue, bool init = false)
DbMigrationRemove(string projectName, bool init = false)
DbMigrationRollback(string projectName, string migrationName, bool init = false)
DbScript(string projectName, string schema, string type, string issue, bool init = false, string? objectName = null)
```

Tool names available through MCP:

- `db_migration_add` → `DbMigrationAdd`
- `db_migration_remove` → `DbMigrationRemove`
- `db_migration_rollback` → `DbMigrationRollback`
- `db_script` → `DbScript`

Do not invent parameters, command names, paths, schemas, script types, object names, or migration targets. Use only the parameters above unless the live MCP schema explicitly differs. Ask for every missing parameter required to execute the selected tool.

## Required behavior

1. Treat any database schema-change request as a potential migration workflow.
2. Confirm the current working directory is a .NET project or solution before using migration tools.
3. Validate the expected project structure before running any MCP tool.
4. Determine `projectName` from the workspace only when it is unambiguous.
5. Ask the user for any missing required parameters or ambiguous values before executing.
6. Use defaults only when documented in this skill.
7. Execute the relevant MCP tool only after all required inputs are known.
8. Show the user:
   - the operation performed,
   - the tool called,
   - the parameters used,
   - generated or modified file paths,
   - validation results,
   - warnings or follow-up actions,
   - a concise summary of the outcome.

## Project validation

Before running `db_migration_add`, `db_migration_remove`, `db_migration_rollback`, or `db_script`, validate that the current workspace is a .NET project or solution and that it contains projects or folders matching these patterns:

- `*.Context`
- `*.Manager`
- `*.SchemaInitialization`
- `*.SchemaUpdates`

A valid workspace normally has one or more of these .NET indicators:

- a `.sln` file,
- one or more `.csproj` files,
- a repository layout clearly containing .NET source projects.

If validation fails, stop and explain what is missing. Ask the user to provide or navigate to the correct project path. Do not run migration tools from an unvalidated directory.

## Project name derivation

Derive `projectName` from the required project/folder names when possible.

Example:

- `ParkingMigration.Context`
- `ParkingMigration.Manager`
- `ParkingMigration.SchemaInitialization`
- `ParkingMigration.SchemaUpdates`

In this case, `projectName` is `ParkingMigration`.

Rules:

1. Strip the suffix `.Context`, `.Manager`, `.SchemaInitialization`, or `.SchemaUpdates` from matching projects/folders.
2. Group matches by shared prefix.
3. If exactly one prefix has the complete required structure, use that prefix as `projectName`.
4. If multiple prefixes match, ask which `projectName` to use.
5. If no complete prefix matches, stop and ask the user to provide the correct project/workspace.
6. Do not infer `projectName` from unrelated folder names, repository names, branch names, or namespaces.

## Intent routing

Use this routing table to decide which tool to use:

| User intent | Tool |
|---|---|
| create a migration, add a migration, new schema change | `db_migration_add` |
| remove/delete the latest migration | `db_migration_remove` |
| rollback/revert a migration or database schema state | `db_migration_rollback` |
| create/add/generate a migration script file | `db_script` |

If the request combines operations, perform them in a safe order. For example, create a migration before generating a script for that migration. Ask before doing destructive or irreversible operations.

## Parameter rules

### `projectName`

Use the project name derivation rules above. Ask only if it cannot be determined unambiguously.

### `migrationName`

Required for:

- `db_migration_add`
- `db_migration_rollback`

The migration name must follow the pattern:

```text
MIGRATION_NAME
```

Ask for it if missing. If the user provides a name in another style, ask whether to convert it to the required pattern or request the exact value to use. Do not silently rename it.

### `issue`

Required for:

- `db_migration_add`
- `db_script`

If the user does not specify `issue`, generate it automatically from the current date using:

```text
TL_yyyyMMdd
```

Example for April 27, 2026:

```text
TL_20260427
```

Use the agent's current local date. Tell the user when this default was applied.

### `init`

Optional for all tools and defaults to `false`.

Normally use `false`. It is acceptable to ask whether `init` should be `true` when the request suggests initialization or when the agent is unsure. Do not block execution solely to ask about `init` when the normal default `false` is appropriate and no initialization intent is present.

### `schema`

Required for:

- `db_script`

Always ask for `schema` if missing. Do not infer it from project names, namespaces, database names, or object names.

### `type`

Required for:

- `db_script`

Always ask for `type` if missing. The expected values are:

```csharp
public enum DbScriptType
{
    Query,
    ModifyStoredProcedure,
    CreateStoredProcedure,
    ModifyTable,
    CreateTable,
    CreateView,
    ModifyView
}
```

Accept only one of:

- `Query`
- `ModifyStoredProcedure`
- `CreateStoredProcedure`
- `ModifyTable`
- `CreateTable`
- `CreateView`
- `ModifyView`

If the user's wording maps clearly to one value, confirm the mapping before executing unless the user already used the exact enum value.

### `objectName`

Optional for:

- `db_script`

Ask for the object name when creating or modifying a table, view, or stored procedure, or when the user asks to add a named database object. Do not infer object names from free-form descriptions unless the user explicitly gave the exact name.

For `Query`, `objectName` may remain `null` unless the user gives one.

## Add migration workflow

Use this workflow for requests like “quiero crear una migración”, “add migration”, or “create a schema change”.

Required parameters:

- `projectName`
- `migrationName`
- `issue`
- `init`

Steps:

1. Validate the .NET workspace and required project patterns.
2. Derive `projectName` if possible.
3. Ask for `migrationName` if missing and enforce the `MIGRATION_NAME` pattern.
4. Use provided `issue`; otherwise generate `TL_yyyyMMdd` from the current date.
5. Use `init = false` unless initialization was requested or should be clarified.
6. Call `db_migration_add` with the completed parameters.
7. Report created migration files and any warnings returned by the tool.

## Remove migration workflow

Use this workflow for requests like “remove migration” or “elimina la migración”.

Required parameters:

- `projectName`
- `init`

Steps:

1. Validate the .NET workspace and required project patterns.
2. Derive `projectName` if possible.
3. Use `init = false` unless initialization was requested or should be clarified.
4. Warn the user if the operation appears destructive or could remove code.
5. Call `db_migration_remove` with the completed parameters.
6. Report removed files, affected projects, and any warnings.

Do not ask for `migrationName` for removal unless the live MCP schema changes, because `DbMigrationRemove` only accepts `projectName` and `init`.

## Rollback workflow

Use this workflow for requests like “rollback migration”, “revert database”, or “hacer rollback”.

Required parameters:

- `projectName`
- `migrationName`
- `init`

Steps:

1. Validate the .NET workspace and required project patterns.
2. Derive `projectName` if possible.
3. Ask for `migrationName` if missing and enforce the `MIGRATION_NAME` pattern.
4. Use `init = false` unless initialization was requested or should be clarified.
5. Make sure the user understands the rollback target before running if the operation changes an actual database.
6. Call `db_migration_rollback` with the completed parameters.
7. Report the target reached, affected migration, and any database/script output returned by the tool.

## Script generation workflow

Use this workflow for requests like “quiero agregar un script”, “generate db script”, or “script para la migración”.

Required parameters:

- `projectName`
- `schema`
- `type`
- `issue`
- `init`
- `objectName` when relevant to the selected script type

Steps:

1. Validate the .NET workspace and required project patterns.
2. Derive `projectName` if possible.
3. Ask for `schema` if missing.
4. Ask for `type` if missing and restrict it to the expected enum values.
5. Ask for `objectName` when the script type is for creating or modifying a stored procedure, table, or view.
6. Use provided `issue`; otherwise generate `TL_yyyyMMdd` from the current date.
7. Use `init = false` unless initialization was requested or should be clarified.
8. Call `db_script` with the completed parameters.
9. Report generated script file paths and how they relate to the migration.

## Clarifying-question style

Ask only for information needed to safely execute the selected MCP tool. Keep questions specific and grouped by operation.

Good examples:

- “¿Cuál es el `migrationName`? Debe seguir el patrón `MIGRATION_NAME`.”
- “Encontré `ParkingMigration` y `BillingMigration`. ¿Cuál `projectName` debo usar?”
- “¿A qué `migrationName` quieres hacer rollback?”
- “¿Cuál es el `schema` para el script?”
- “¿Qué `type` debo usar? Valores válidos: `Query`, `ModifyStoredProcedure`, `CreateStoredProcedure`, `ModifyTable`, `CreateTable`, `CreateView`, `ModifyView`.”
- “¿Cuál es el `objectName` exacto?”

Avoid vague questions like “¿qué quieres hacer?” when the intent is already clear.

## Output format

After each completed operation, respond in this structure:

```markdown
## Resultado
[success/failure summary]

## Validación
- Proyecto .NET: [ok/fail]
- Estructura requerida: [ok/fail]
- ProjectName detectado: [value]
- Proyectos detectados: [list]

## Tool ejecutada
- Tool: `[tool name]`
- Parámetros: `[safe parameter summary]`

## Archivos / cambios
- [paths or generated/removed files]

## Notas
- [warnings, defaults applied, next steps, or tool messages]
```

If the operation cannot proceed because information is missing, do not use the final result template. Ask the missing questions directly.
