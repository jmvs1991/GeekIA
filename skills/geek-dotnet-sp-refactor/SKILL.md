---
name: geek-dotnet-sp-refactor
description: refactor .net stored procedure model classes from a legacy nested sp container pattern into separated classes such as spinsert, spupdate, and spdelete. use when the user shares c# classes that build sqlparameter arrays for stored procedures, especially when they want to migrate from inner classes and duplicated parameter helpers to a new architecture based on spbase, spbaseenc, isp, and ispenc. also use when the task involves choosing or adapting repo templates created with dotnet new geek-sp, geek-sp-corp, or geek-sp-corp-co, including cases where insert or update operations require encryption but no dedicated encrypted template exists.
---

# Geek Dotnet Sp Refactor

Refactor legacy C# stored-procedure models into the target SP architecture used by the project.
Preserve behavior, stored procedure names, parameter order, and property defaults unless the user explicitly asks for a semantic change.

## Default workflow

1. Read the input class and identify each operation represented inside it, usually insert, update, delete, get, list, or another stored procedure that impacts data in the database.
2. Detect shared concerns in the legacy base class:
   - common properties such as `User`
   - parameter helper methods
   - encryption logic
   - enum-to-string conversions
   - corporation or company keys
3. Choose the nearest scaffold strategy from the repo templates:
   - `dotnet new geek-sp -n <Name> --projectName <Project>` for base fields only
   - `dotnet new geek-sp-corp -n <Name> --projectName <Project>` when corporation is required
   - `dotnet new geek-sp-corp-co -n <Name> --projectName <Project>` when both corporation and company are required
4. If no template covers encryption directly, still use the nearest structural template and then adapt the generated classes manually:
   - keep `SpDelete` on `SpBase` + `ISp` unless delete itself needs encryption
   - move encrypted insert or update operations to `SpBaseEnc` + `ISpEnc`
   - replace static encryption with `IEncryptorService`
5. Split each operation into its own concrete class under a dedicated `Sp` folder in the target location.
6. Move the files physically into that folder and update the namespace to match the new folder path, usually `...Models.Sp`.
7. Move only the properties required by that operation into the generated class.
8. Keep each parameter helper local to the new class unless it already belongs in the inherited base class.
9. Rewrite `MapToSqlParameters` using the target signature:
   - `override (string sp, SqlParameter[] parameters) MapToSqlParameters()` for non-encrypted operations
   - `override (string sp, SqlParameter[] parameters) MapToSqlParameters(IEncryptorService encryptor)` for encrypted operations
10. Return the SQL text and parameter array with the same stored procedure name and parameter order as the original code.

## Transformation rules

### 1. Scope of operations

Treat insert, update, delete, and any other stored procedure that writes or impacts data in the database as valid targets for this refactor.
Do not limit the refactor to only the three canonical CRUD write operations when the source shows other write-oriented SPs.

### 2. Class splitting

Convert nested classes like `ClientSP.ClientInsert`, `ClientSP.ClientUpdate`, and `ClientSP.ClientDelete` into separate top-level classes such as:
- `SpInsert`
- `SpUpdate`
- `SpDelete`

For additional write operations, keep the `Sp` prefix and use a name that matches the operation intent, for example `SpActivate`, `SpBlock`, `SpAssign`, or `SpMerge`.

Use one file per generated class unless the user explicitly asks for a single-file result.

### 3. Namespace and folder migration

Move the resulting classes into a dedicated `Sp` folder when the target pattern shown by the user does that.
Update the namespace so it matches the new folder path, typically from something like `ClientServices.Client.Models` to `ClientServices.Client.Models.Sp`.
Do not keep the old namespace if the files now live inside the `Sp` folder.

### 4. Base inheritance

Assume `GetUserParameter()` already exists on `SpBase` or `SpBaseEnc` in the new architecture unless the user shows otherwise.
Do not re-implement inherited helpers that are already provided by the base class.

### 5. Encryption migration

When legacy code uses static encryption utilities such as:
- `Encryption.Encrypt(...)`
- `Configuration.EncryptionKey`

replace that logic with `IEncryptorService` and call `encryptor.Encrypt(...)` inside the specific parameter helper.

When only some operations need encryption:
- allow mixed output classes for the same entity
- keep encrypted operations on `SpBaseEnc` + `ISpEnc`
- keep non-encrypted operations on `SpBase` + `ISp`

### 6. Template gap handling

If the repository has only these generators:
- `geek-sp`
- `geek-sp-corp`
- `geek-sp-corp-co`

and none generates encrypted variants, do not block on the missing template.
Use the closest template for structure and then refactor the generated files to the encrypted architecture manually.

Selection order:
1. choose by required structural keys first: base, corporation, or corporation plus company
2. then upgrade only the classes that actually encrypt fields to `SpBaseEnc` + `ISpEnc`
3. keep delete or other non-encrypted operations unchanged unless the source requires encryption there too

### 7. Parameter helpers

Prefer expression-bodied helpers for simple parameter creation.
Example pattern:

```csharp
protected SqlParameter GetNameParameter() => new SqlParameter("@name", Name);
```

Keep helper names aligned with the original intent unless a naming cleanup is obvious and safe.

### 8. Parameter order

`SqlParameter[]` order is load-bearing. Preserve it exactly as the original `MapToSqlParameters` method emitted it.
Do not reorder for style.

### 9. Stored procedure string

Preserve the stored procedure name and placeholder order exactly.
Normalize obvious type casing only when it is purely stylistic, for example `String` to `string`.

### 10. Extra legacy members

If a legacy operation contains properties not used in `MapToSqlParameters`, keep them only when they are clearly part of the operation contract. Otherwise, omit them and mention the omission.

## Output contract

When generating a refactor, produce:

1. The refactored C# code.
2. The target file split or file paths when relevant, for example `Models/Sp/SpInsert.cs`, `Models/Sp/SpUpdate.cs`, and `Models/Sp/SpDelete.cs`.
3. A short note listing:
   - which classes were created
   - which base class and interface each one uses
   - which template command is the closest starting point
   - whether any class had to be manually upgraded because no encrypted template exists
   - where encryption handling changed
   - any assumptions made

If the user gives a single source file with multiple nested operations, default to returning all resulting classes and place them in the `Sp` folder unless the user shows a different folder convention.

## Quality bar

Before finalizing, verify all of the following:

- namespaces and `using` directives match the new architecture
- every property used by a helper exists on the class or inherited base
- encrypted operations accept `IEncryptorService encryptor`
- non-encrypted operations do not depend on encryptor services
- stored procedure names match the source
- parameter arrays preserve original order
- the selected template command matches the required structural keys
- if encryption is needed, the output explicitly notes that it was adapted beyond the available templates
- no legacy nested container class remains unless the user asked to keep it

## Example request patterns

- “Refactor this `*SP` class to the new `SpBase/SpBaseEnc` pattern.”
- “Split this nested SP model into `SpInsert`, `SpUpdate`, and `SpDelete`.”
- “Which `dotnet new geek-sp*` template should I use for this SP model?”
- “I only have `geek-sp`, `geek-sp-corp`, and `geek-sp-corp-co`; adapt the result for encryptor-based insert and update.”

## Reference

For a concise migration checklist and before/after conventions, read `references/refactor-patterns.md`.
For template-selection rules and the encryptor gap workflow, read `references/template-selection.md`.
