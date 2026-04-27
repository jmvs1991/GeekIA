---
name: geek-dotnet-db-migration
description: manage dotnet database migration tasks through the geek cli mcp tools. use this skill when the user asks to create, add, remove, rollback, or script database migrations, or makes any request related to database schema changes in a dotnet project that uses *.context, *.manager, *.schemainitialization, and *.schemaupdates project structure. always inspect the available mcp tool schemas, validate the project structure, ask for all missing required fields, and report executed actions, generated files, paths, and summaries.
---

# Geek Dotnet DB Migration

## Objective

Use the Geek CLI MCP tools to manage database migration work for .NET projects. This skill covers migration creation, removal, rollback, and migration script generation.

Supported MCP tools:

- `db_migration_add` — add a new migration.
- `db_migration_remove` — remove a migration.
- `db_migration_rollback` — rollback a migration.
- `db_script` — generate script files that will run in the migration.

Do not invent tool parameters, command names, project paths, migration names, environments, or targets. Inspect the available MCP tool schemas when possible and ask for every missing required field before executing.

## Required behavior

1. Treat any database schema-change request as a potential migration workflow.
2. Confirm the current working directory is a .NET project or solution before using migration tools.
3. Validate the expected project structure before running any MCP tool.
4. Inspect the MCP tool schema for the intended operation and identify required parameters.
5. Ask the user for any missing required parameters or ambiguous values.
6. Execute the relevant MCP tool only after required inputs are known.
7. Show the user:
   - the operation performed,
   - the tool called,
   - the parameters used, excluding secrets,
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

## Intent routing

Use this routing table to decide which tool to use:

| User intent | Tool |
|---|---|
| create a migration, add a migration, new schema change | `db_migration_add` |
| remove/delete the latest or named migration | `db_migration_remove` |
| rollback/revert a migration or database schema state | `db_migration_rollback` |
| create/add/generate a migration script file | `db_script` |

If the request combines operations, perform them in a safe order. For example, create a migration before generating a script for that migration. Ask before doing destructive or irreversible operations.

## Parameter handling

Always inspect the MCP tool definition or schema for the selected tool. Use that schema as the source of truth for parameters, required fields, defaults, and allowed values.

If a required field is missing, ask a focused question. Examples of fields that may be required depending on the tool schema:

- migration name,
- target project or startup project,
- context name,
- schema updates project path,
- environment,
- connection string or database target,
- rollback target migration,
- script name,
- script type,
- output path.

Do not ask for fields that the tool schema does not need unless they are necessary to disambiguate the project or operation.

Never expose secrets. If a parameter contains a connection string, token, password, or credential, mask it in the final response.

## Add migration workflow

Use this workflow for requests like “quiero crear una migración”, “add migration”, or “create a schema change”.

1. Validate the .NET workspace and required project patterns.
2. Inspect `db_migration_add` schema.
3. Ask for missing required fields, especially migration name and any required project/context values.
4. Call `db_migration_add` with the completed parameters.
5. Report created migration files and any warnings returned by the tool.

## Remove migration workflow

Use this workflow for requests like “remove migration” or “elimina la migración”.

1. Validate the .NET workspace and required project patterns.
2. Inspect `db_migration_remove` schema.
3. Ask which migration should be removed if the tool requires it or if the request is ambiguous.
4. Warn the user if the operation appears destructive or could remove code.
5. Call `db_migration_remove` with the completed parameters.
6. Report removed files, affected projects, and any warnings.

## Rollback workflow

Use this workflow for requests like “rollback migration”, “revert database”, or “hacer rollback”.

1. Validate the .NET workspace and required project patterns.
2. Inspect `db_migration_rollback` schema.
3. Ask for the rollback target, database target/environment, or connection information if required by the tool.
4. Make sure the user understands the rollback target before running if the operation changes an actual database.
5. Call `db_migration_rollback` with the completed parameters.
6. Report the target reached, affected migration, and any database/script output returned by the tool.

## Script generation workflow

Use this workflow for requests like “quiero agregar un script”, “generate db script”, or “script para la migración”.

1. Validate the .NET workspace and required project patterns.
2. Inspect `db_script` schema.
3. Ask for missing script details, such as script name, target migration, script type, content source, or output path if required.
4. Call `db_script` with the completed parameters.
5. Report generated script file paths and how they relate to the migration.

## Clarifying-question style

Ask only for information needed to safely execute the selected MCP tool. Keep questions specific and grouped by operation.

Good examples:

- “¿Cuál es el nombre de la migración que quieres crear?”
- “¿Qué `DbContext` debo usar? Encontré `BillingContext` y `IdentityContext`.”
- “¿A qué migración quieres hacer rollback?”
- “¿En qué ambiente o connection string debo generar el script? Puedes pegar el valor; lo voy a ocultar en el resumen.”
- “Encontré varios proyectos `*.SchemaUpdates`. ¿Cuál debo usar?”

Avoid vague questions like “¿qué quieres hacer?” when the intent is already clear.

## Output format

After each completed operation, respond in this structure:

```markdown
## Resultado
[success/failure summary]

## Validación
- Proyecto .NET: [ok/fail]
- Estructura requerida: [ok/fail]
- Proyectos detectados: [list]

## Tool ejecutada
- Tool: `[tool name]`
- Parámetros: `[safe parameter summary with secrets masked]`

## Archivos / cambios
- [paths or generated/removed files]

## Notas
- [warnings, next steps, or tool messages]
```

If the operation cannot proceed because information is missing, do not use the final result template. Ask the missing questions directly.
