# Context pattern

Use this shape for schema contexts.

```csharp
using Connection.<Schema>.Entities;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace Connection.<Schema>
{
    public class <Schema>Context : DbContext
    {
        private readonly Action<EntityTypeBuilder<<BaseName>Entity>> _<BaseName>Builder = <BaseName>Entity.<BaseName>Builder;

        public <Schema>Context(DbContextOptions<<Schema>Context> options) : base(options)
        {
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.HasDefaultSchema("<Schema>");
            modelBuilder.Entity(_<BaseName>Builder);
        }
    }
}
```

## Notes
- Reuse the existing context file when it already exists.
- Append new builder delegates without removing existing ones.
- Chain `.Entity(...)` calls when multiple entities exist.
- Remove the scaffold-generated context file instead of keeping two contexts.
