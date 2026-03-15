# Template Commands

## Decision table

Choose the template based on the scope present in the route, request, or key.

- No corporation, no company, no code -> `geek-resource`
- Corporation only -> `geek-resource-corp`
- Corporation and company -> `geek-resource-corp-co`
- Corporation, company, and code -> `geek-resource-corp-co-code`

## Commands

```bash
dotnet new geek-resource -n <Name> --projectName <Project>
```

```bash
dotnet new geek-resource-corp -n <Name> --projectName <Project>
```

```bash
dotnet new geek-resource-corp-co -n <Name> --projectName <Project>
```

```bash
dotnet new geek-resource-corp-co-code -n <Name> --projectName <Project>
```

## Verification

List installed Geek templates:

```bash
dotnet new list geek
```

Install templates from a local package when missing:

```bash
dotnet new install .\nupkgs\<package>.nupkg
```

## Notes

The related template families are:

- Resource
- ResourceWithCorporation
- ResourceWithCorporationAndCompany
- ResourceWithCorporationCompanyAndCode
