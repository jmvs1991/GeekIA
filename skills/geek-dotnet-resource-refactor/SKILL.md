---
name: geek-dotnet-resource-refactor
description: review and refactor .net resource classes and multilingual message keys into clearer, behavior-preserving names and error messages. use when chatgpt is asked to inspect a resource class, improve english/spanish localization text, replace opaque keys such as msj0001 with descriptive names, assess whether migration to resourcebase is actually necessary, avoid forcing structural changes when the current model is already appropriate, keep compatibility during refactors, or choose the correct geek resource template command for corporation/company/code scoped resources.
---

# Geek Dotnet Resource Refactor

Review and refactor resource models, localization keys, and bilingual error messages in .NET codebases while minimizing unnecessary structural change.

## Core workflow

1. Inspect the current resource model and identify:
   - existing property names
   - localization keys
   - inheritance model (`ResourceBase` or standalone)
   - route or scope requirements such as corporation, company, or code
   - current English and Spanish texts when available
2. Decide whether the request needs:
   - text cleanup only
   - naming cleanup only
   - small structural adjustments
   - full migration to `ResourceBase`
3. Prefer the smallest change that solves the real problem.
4. Rename properties and keys to descriptive, domain-oriented names only when the current names are materially confusing.
5. Improve default English and Spanish messages so the error is explicit and actionable.
6. Preserve compatibility by calling out any breaking changes and, when appropriate, suggesting a temporary alias strategy.
7. If the user needs a new resource model, choose the correct `dotnet new geek-*` template command.

## Default decision policy

Do **not** force migration to `ResourceBase` just because it is available.

Before proposing a migration, verify whether it is actually justified. Treat migration as necessary only when it provides a concrete benefit such as:

- removing duplicated culture-handling logic already covered by the base class
- aligning with an existing project standard the user is already following
- enabling a required shared contract such as `notFoundMsg`
- eliminating inconsistent patterns that are causing maintenance problems

If the current standalone resource class is simple, clear, and not causing duplication or inconsistency, keep it as-is and limit the recommendation to text, key, or naming improvements.

When the input resource does **not** need migration, still review:

- clarity of property names
- clarity of localization keys
- quality of English and Spanish messages
- accidental ambiguity between generic and specific error messages
- opportunities for safe, low-risk cleanup

## Refactor rules

- Prefer names that describe the business meaning of the error, not the storage sequence.
- Replace opaque names like `clientMsj0001` with names such as `notFoundByCodeMsg` or `notFoundByDocumentMsg` when the current names are genuinely unclear.
- Keep the `Msg` suffix when the existing codebase already uses that convention.
- Keep names in English for keys and properties unless the project clearly uses another convention.
- Use PascalCase for localization keys and camelCase for exposed C# properties when following the existing pattern.
- Do not silently change behavior, lookup mechanics, inheritance, or base-class structure.
- Preserve `_culture` handling. If the refactor moves to `ResourceBase`, use the base constructor and keep lookups compatible with the inherited `_culture` field.
- Treat message text as user-facing. Rewrite it for clarity, specificity, and recoverability.
- Avoid generic messages like `operation failed` or `record not found` when a more precise variant is possible.
- If the existing message text is already clear and specific, do not rewrite it just to rephrase it.

## Necessity assessment

When evaluating an existing resource, explicitly classify each proposed change as one of these:

- **necessary**: fixes real ambiguity, duplication, inconsistency, or misleading behavior
- **recommended**: improves clarity or maintainability, but can be deferred
- **optional**: stylistic cleanup only
- **not recommended**: change adds churn without enough value

Use this assessment especially for:

- migration to `ResourceBase`
- renaming public properties
- renaming localization keys
- introducing generic overrides such as `notFoundMsg`

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

## ResourceBase migration pattern

When converting a standalone resource class to `ResourceBase`:

1. Confirm that migration is warranted under the necessity assessment.
2. Remove the duplicated `_culture` field from the child class if the base already owns it.
3. Inherit from `ResourceBase`.
4. Move constructor initialization to `: base(culture)`.
5. Override only the base members that are semantically required.
6. Keep additional message properties focused and descriptive.

Example target pattern:

```csharp
public class ClientResource : ResourceBase
{
    public override string notFoundMsg => Localization<ClientResource>.GetValue("NotFoundByCodeMsg", _culture);

    public string notFoundByCodeMsg => Localization<ClientResource>.GetValue("NotFoundByCodeMsg", _culture);

    public string notFoundByDocumentMsg => Localization<ClientResource>.GetValue("NotFoundByDocumentMsg", _culture);

    public ClientResource(CultureInfo culture) : base(culture)
    {
    }
}
```

When reviewing a migration like this, verify whether `notFoundMsg` should point to a generic key or to the code-specific key. If the override is intended to represent the generic not-found case, prefer a distinct generic key instead of reusing the code-specific one.

If no base override is semantically needed, prefer not adding one.

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

## Choosing the correct template command

Use these commands for new resource models:

### Basic resource

Use when the route or key does not include corporation, company, or code.

```bash
dotnet new geek-resource -n <Name> --projectName <Project>
```

Example:

```bash
dotnet new geek-resource -n Customer --projectName MyApp.Api
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

## Support commands

```bash
dotnet new list geek
```

Use this to confirm the installed Geek template names.

```bash
dotnet new install .\nupkgs\<package>.nupkg
```

Use this when the Geek templates are not installed yet.

## Bundled references

- See `references/refactor-guidelines.md` for naming, review, and migration guidance.
- See `references/template-commands.md` for template selection rules and command examples.

## Output expectations

When answering, prefer this structure:

1. brief diagnosis of the current problem
2. necessity assessment for migration and renames
3. proposed new names and why
4. updated C# example only for the changes that are justified
5. suggested English and Spanish resource texts
6. migration notes, compatibility risks, or explanation for leaving the structure unchanged
