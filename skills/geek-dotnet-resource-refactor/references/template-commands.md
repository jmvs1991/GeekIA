# Template Commands

## Decision table

Choose the template based on the scope present in the route, request, inheritance, or key.

- No corporation, no company, no code -> `geek-resource`
- Corporation only -> `geek-resource-corp`
- Corporation and company -> `geek-resource-corp-co`
- Corporation, company, and code -> `geek-resource-corp-co-code`

If the correct scope is not inferable from the code, ask the user which one applies before recommending or using a command.

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

## How to use them in this skill

Use the generation commands when one of these is true:

- the user wants to create a new resource from scratch
- the existing class is missing the correct inherited resource family and regenerating is cleaner than hand-editing
- the user explicitly asks for the exact command to create the needed resource

Do not default to regeneration when a light edit to the current class is safer.

When inheritance already contributes the message members or message codes, prefer using the correct resource family and then adding only the missing localized texts in the `.resx` files.

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
