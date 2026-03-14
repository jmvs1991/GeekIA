# Entity pattern

Use this shape for transformed entities.

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace Connection.<Schema>.Entities;

[Table("<TABLE_NAME>", Schema = "<Schema>")]
public partial class <BaseName>Entity : <BaseEntityClass>, <BaseEntityInterface>
{
    // scaffolded properties and navigation properties

    public static Action<EntityTypeBuilder<<BaseName>Entity>> <BaseName>Builder = entity =>
    {
        // key configuration
        // default values
        // property configuration
    };
}
```

## Notes
- Preserve scaffolded data annotations from EF Core output.
- Keep the class partial if the scaffold output is partial.
- Use `<BaseName>Builder` as the static builder property name.
- Use the base entity type decision from SKILL.md.
- For corporation/company entities, configure the composite key in the builder.
- Keep `Id` value generation and SQL defaults when they are present in the source table.
