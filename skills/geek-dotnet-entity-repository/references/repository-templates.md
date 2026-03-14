# Repository template selection

## Read repositories

### No corporation or company columns
```bash
dotnet new geek-read-paginationful-filterless-entity --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

### Corporation only
```bash
dotnet new geek-read-paginationful-filterless-entity-corp --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

### Corporation and company
```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

### Corporation, company, and code field
```bash
dotnet new geek-read-paginationful-filterless-entity-corp-co-code --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

The code-field variant is usually appropriate when the primary key contains `CORPORATION`, `COMPANY`, and one additional key column.

## Write repositories

```bash
dotnet new geek-write-storeprocedureful-transactionless --dbSchema <schema> --contextName <ContextName> -n <BaseName>
```

## Target folders

```text
connection/<Schema>/Repositories/<BaseName>Entity/Read
connection/<Schema>/Repositories/<BaseName>Entity/Write
```
