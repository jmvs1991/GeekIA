# Refactor patterns

## Legacy shape

Common legacy signals:
- Outer container class named `SomethingSP`
- Nested operation classes inheriting from an inner `Base`
- Repeated `GetXParameter()` helpers
- Static encryption calls
- `MapToSqlParameters()` methods returning `(String spX, SqlParameter[] parameters)`

## Target shape

Common target signals:
- Files move into a dedicated `Sp` folder
- Namespace ends in `.Models.Sp`
- One top-level class per operation
- `SpBase` or `SpBaseEnc` inheritance
- `ISp` or `ISpEnc` implementation
- Expression-bodied parameter helpers where possible
- Encryption delegated to `IEncryptorService`

## Mapping cheat sheet

### Delete operations
- Usually inherit from `SpBase`
- Usually implement `ISp`
- Usually keep only identifier fields and `GetUserParameter()` if actually used
- `MapToSqlParameters()` has no encryptor argument

### Insert and update operations with encrypted fields
- Usually inherit from `SpBaseEnc`
- Usually implement `ISpEnc`
- `MapToSqlParameters(IEncryptorService encryptor)`
- Parameter helper for encrypted values receives `encryptor`

## Migration checklist

1. Split nested operations into top-level classes.
2. Move the new files into the `Sp` folder and update namespace declarations to match that folder.
3. Move only operation-specific properties into each class.
4. Keep stored procedure text unchanged.
5. Keep parameter order unchanged.
6. Replace static encryption with `encryptor.Encrypt(...)`.
7. Remove obsolete outer wrapper and obsolete inner base when the new architecture covers them.
8. Preserve enum serialization behavior such as `TypeDocument.ToString()`.
9. Keep user parameter inherited when available from the new base class.

## Assumptions to call out

Explicitly mention assumptions when the source code does not show:
- implementation of `SpBase` or `SpBaseEnc`
- contract of `ISp` or `ISpEnc`
- whether `User` is inherited
- whether unused properties should remain
