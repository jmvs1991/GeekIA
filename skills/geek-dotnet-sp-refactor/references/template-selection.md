# Template selection

## Available generators

### Base entity only
```bash
dotnet new geek-sp -n <Name> --projectName <Project>
```
Use when the stored procedure model only needs the base fields for the entity.

### Entity plus corporation
```bash
dotnet new geek-sp-corp -n <Name> --projectName <Project>
```
Use when the stored procedure model also needs a Corporation field or key.

### Entity plus corporation and company
```bash
dotnet new geek-sp-corp-co -n <Name> --projectName <Project>
```
Use when the stored procedure model needs both Corporation and Company.

## What these generators produce

Each command generates:
- `SpInsert.cs`
- `SpUpdate.cs`
- `SpDelete.cs`

## Missing encrypted template

If there is no generator specifically for encrypted stored procedure classes:

1. Pick the nearest generator by shape alone.
2. Generate the files.
3. Upgrade only the classes that encrypt values:
   - change inheritance from `SpBase` to `SpBaseEnc` when needed
   - change interface from `ISp` to `ISpEnc` when needed
   - add `using SolutionDomain.Domains.Security.Interfaces;`
   - change `MapToSqlParameters()` signature to accept `IEncryptorService encryptor`
   - replace static or configuration-based encryption with `encryptor.Encrypt(...)`
4. Leave non-encrypted classes such as most deletes on the non-encrypted path.

## Practical decision matrix

- No corporation, no company, no encrypted fields:
  - start from `geek-sp`
- Corporation only, no encrypted fields:
  - start from `geek-sp-corp`
- Corporation and company, no encrypted fields:
  - start from `geek-sp-corp-co`
- Any of the above plus encrypted insert or update fields:
  - start from the same nearest template
  - manually upgrade the encrypted classes to `SpBaseEnc` and `ISpEnc`

## Notes to include in final answers

When the refactor required the encryptor path, explicitly state:
- which template was the closest starting point
- which generated classes stayed unchanged structurally
- which classes were manually upgraded because the repo lacks an encrypted generator
