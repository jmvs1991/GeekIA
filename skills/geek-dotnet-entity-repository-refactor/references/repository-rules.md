# Repository Rules

## Folder structure
For entity class `<Name>Entity` under schema `<Schema>`, create:

```text
connection/<Schema>/Repositories/<Name>Entity/Read
connection/<Schema>/Repositories/<Name>Entity/Write
```

## Read command selection

### Base entity
```bash
dotnet new geek-read-paginationful-filterless-entity --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

### Corporation tier
```bash
dotnet new geek-read-paginationful-filterless-entity-corp --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

### Corporation and company tier
```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

### Corporation, company, and code tier
```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co-code --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

Use the `-code` template when the company-tier entity's primary key is `Corporation`, `Company`, and one additional key property that functions as the code field.

## Write command
```bash
dotnet new geek-write-storeprocedureful-transactionless --dbSchema <schema> --contextName <Schema>Context -n <Name>
```

## Expected generated files
Read:
- `I<Name>.read.repository.cs`
- `<Name>.read.repository.cs`

Write:
- `I<Name>.write.repository.cs`
- `<Name>.write.repository.cs`
