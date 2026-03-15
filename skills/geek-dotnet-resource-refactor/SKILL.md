---
name: geek-dotnet-resource-refactor
description: review and refactor .net resource classes and multilingual message keys into clearer, behavior-preserving names and error messages. use when chatgpt is asked to inspect a resource class, improve english/spanish localization text, replace opaque keys such as msj0001 with descriptive names, decide whether migration or inheritance changes are actually necessary, ask which geek inheritance scope applies (base, corporation, corporation and company, or corporation + company + code) when that choice is unclear, avoid forcing structural changes when the current model is already appropriate, add only the missing resource texts when inheritance already provides message members, or choose and use the correct geek resource template command.
---

# Geek Dotnet Resource Refactor

Review and refactor resource models, localization keys, bilingual error messages, and Geek resource inheritance choices in .NET codebases while minimizing unnecessary structural change.

## Core workflow

1. Inspect the current resource model and identify:
   - existing property names
   - localization keys
   - inheritance model (`ResourceBase`, corporation/company variants, or standalone)
   - route or scope requirements such as corporation, company, or code
   - current English and Spanish texts when available
2. Decide whether the request needs:
   - text cleanup only
   - naming cleanup only
   - missing inheritance or template alignment
   - small structural adjustments
   - full migration to a base or scoped resource class
3. Prefer the smallest change that solves the real problem.
4. Rename properties and keys to descriptive, domain-oriented names only when the current names are materially confusing.
5. Improve default English and Spanish messages so the error is explicit and actionable.
6. If inheritance already defines the message members or message codes, do not recreate them in the child class. Add or adjust only the missing `.resx` texts.
7. If the inheritance choice is unclear, ask the user which scope applies: `base`, `corporation`, `corporation + company`, or `corporation + company + code`.
8. If the user needs a new resource model, choose and use the correct `dotnet new geek-*` template command.

## Default decision policy

Do **not** force migration to `ResourceBase` or another inherited resource type just because it is available.

Before proposing a migration, verify whether it is actually justified. Treat migration as necessary only when it provides a concrete benefit such as:

- removing duplicated culture-handling logic already covered by the inherited base class
- aligning with an existing project standard the user is already following
- enabling a required shared contract already used by the project
- replacing a wrong or incomplete inheritance choice
- eliminating inconsistent patterns that are causing maintenance problems

If the current standalone or inherited resource class is simple, clear, and not causing duplication or inconsistency, keep it as-is and limit the recommendation to text, key, or naming improvements.

When the input resource does **not** need migration, still review:

- clarity of property names
- clarity of localization keys
- quality of English and Spanish messages
- accidental ambiguity between generic and specific error messages
- opportunities for safe, low-risk cleanup
- whether inheritance is already supplying members that only need localized text entries

## Inheritance decision rule

When the class is missing a needed inheritance or the correct scope is ambiguous, ask a short targeted question before finalizing the refactor:

- `base`
- `corporation`
- `corporation + company`
- `corporation + company + code`

Map those answers to the Geek resource families and template choices:

- `base` -> `Resource` / `geek-resource`
- `corporation` -> `ResourceWithCorporation` / `geek-resource-corp`
- `corporation + company` -> `ResourceWithCorporationAndCompany` / `geek-resource-corp-co`
- `corporation + company + code` -> `ResourceWithCorporationCompanyAndCode` / `geek-resource-corp-co-code`

Ask this question when either of these is true:

- the resource shape cannot be inferred safely from the supplied code or route
- a migration is needed, but the correct inherited resource family is unclear

Do **not** ask this question when the current inheritance is already clear and correct.

## Refactor rules

- Prefer names that describe the business meaning of the error, not the storage sequence.
- Replace opaque names like `clientMsj0001` with names such as `notFoundByCodeMsg` or `notFoundByDocumentMsg` when the current names are genuinely unclear.
- Keep the `Msg` suffix when the existing codebase already uses that convention.
- Keep names in English for keys and properties unless the project clearly uses another convention.
- Use PascalCase for localization keys and camelCase for exposed C# properties when following the existing pattern.
- Do not silently change behavior, lookup mechanics, inheritance, or base-class structure.
- Preserve `_culture` handling. If the refactor moves to inherited resource types, use the base constructor and keep lookups compatible with the inherited `_culture` field.
- Treat message text as user-facing. Rewrite it for clarity, specificity, and recoverability.
- Avoid generic messages like `operation failed` or `record not found` when a more precise variant is possible.
- If the existing message text is already clear and specific, do not rewrite it just to rephrase it.
- If inheritance already provides the property or code contract, avoid duplicating members in the class. Prefer updating only the resource text entries.

## Necessity assessment

When evaluating an existing resource, explicitly classify each proposed change as one of these:

- **necessary**: fixes real ambiguity, duplication, inconsistency, missing inheritance, or misleading behavior
- **recommended**: improves clarity or maintainability, but can be deferred
- **optional**: stylistic cleanup only
- **not recommended**: change adds churn without enough value

Use this assessment especially for:

- migration to a base or scoped inherited resource
- renaming public properties
- renaming localization keys
- introducing or removing overrides
- switching to a generated Geek resource family

If migration is unnecessary, say so clearly and continue with a lighter review.

## Message-writing guidelines

When rewriting the text in localization files:

- State what was not found or what failed.
- Mention the lookup criteria when known, for example code or document.
- Keep English as the default source language.
- Provide a natural Spanish equivalent, not a word-for-word translation.
- Keep the tone concise and professional.
- Prefer direct user-facing clarity over internal terminology.

Good patterns:

- English: `Client not found for the provided code.`
- Spanish: `No se encontró el cliente para el código proporcionado.`

- English: `Client not found for the provided document.`
- Spanish: `No se encontró el cliente para el documento proporcionado.`

Avoid patterns like:

- `Client message 0001`
- `Error in client query`
- `Data not found`

## Compatibility strategy

When the user is refactoring an existing resource model, explicitly separate:

- safe renames inside the resource class
- required updates in localization files
- required updates in calling code
- optional compatibility aliases
- inheritance changes that are actually required
- changes that are not worth doing right now

If the user seems to be migrating incrementally, prefer suggesting a transition like:

- keep the new descriptive property
- optionally keep the old property temporarily as a forwarding alias
- mark the old name for later cleanup if the team supports deprecation

Example alias pattern:

```csharp
public string clientMsj0001 => notFoundByCodeMsg;
```

Use aliases only when the user values backward compatibility more than immediate cleanup.

## Inherited-resource migration pattern

When converting a standalone resource class to an inherited Geek resource type:

1. Confirm that migration is warranted under the necessity assessment.
2. Determine whether the correct target is `base`, `corporation`, `corporation + company`, or `corporation + company + code`.
3. If the target cannot be inferred safely, ask the user which scope applies.
4. Remove duplicated `_culture` or duplicated inherited members from the child class when the base already owns them.
5. Move constructor initialization to `: base(culture)` or the matching required signature.
6. Override only the members that are semantically required.
7. If inheritance already defines message members or message keys, do not recreate them in the child class; add or adjust only the missing texts in the `.resx` files.

When reviewing a migration, verify whether a generic override such as `notFoundMsg` should point to a genuine generic key or whether the project only needs more specific keys. If no base override is semantically needed, prefer not adding one.

## Naming heuristics

Use these naming patterns when they match the real intent:

- `notFoundMsg`
- `notFoundByCodeMsg`
- `notFoundByDocumentMsg`
- `alreadyExistsMsg`
- `invalidDocumentMsg`
- `invalidCodeMsg`
- `creationFailedMsg`
- `updateFailedMsg`
- `deleteFailedMsg`
- `requiredFieldMsg`

Prefer the most specific valid name. For example, use `notFoundByDocumentMsg` instead of `notFoundMsg` when the message is only used for a document lookup.

## Choosing and using the correct template command

Use these commands for new resource models or when the user asks to generate the resource family directly.

### Basic resource

Use when the route or key does not include corporation, company, or code.

```bash
dotnet new geek-resource -n <Name> --projectName <Project>
```

### Corporation-scoped resource

Use when the resource is scoped by corporation.

```bash
dotnet new geek-resource-corp -n <Name> --projectName <Project>
```

### Corporation and company scoped resource

Use when the resource is scoped by corporation and company.

```bash
dotnet new geek-resource-corp-co -n <Name> --projectName <Project>
```

### Corporation, company, and code scoped resource

Use when the resource is scoped by corporation, company, and code.

```bash
dotnet new geek-resource-corp-co-code -n <Name> --projectName <Project>
```

Before using a generation command, check whether the current task is better solved by editing the existing class instead of regenerating it from a template.

If generation is the best path, recommend the exact command and explain why that scope was selected.
