# Этап 2–4: Модель, конфигурация, контекст

### Требования

- Примеры ниже являются демонстрационными шаблонами «на все случаи жизни» и могут содержать взаимоисключающие варианты конфигурации; при реализации выбирать и применять только подходящие фрагменты, а не копировать пример целиком
- Перед реализацией этапов 2–4 выполнить опрос из `promt_03_requirements.md`; в этом файле вопросы не дублировать

#### Модель (EntityName.cs)
Класс `EntityName` должен содержать:
- Первичный ключ `Id` типа `int`
- Обязательное поле `Name` типа `string`
- Навигационные свойства коллекций должны быть `virtual` с обязательной инициализацией пустым списком
- Foreign Key поля должны иметь соответствующие навигационные свойства
- Обязательные связи: non-nullable Foreign Key + non-nullable навигационное свойство с инициализацией `default!`
- Необязательные связи: nullable Foreign Key + nullable навигационное свойство

#### Конфигурация (ConfigurationEntityName.cs)
- Название таблицы во множественном числе
- Первичный ключ с автоинкрементом
- Уникальный индекс для Name с префиксом IX_
- Строковые поля с ограничением длины и пометкой Required/NotRequired
- Текстовые поля с типом "text" для длинного контента
- Числовые поля с указанием точности (например, numeric(7,2) для денег)
- Булевы поля без дополнительной конфигурации
- Foreign Key поля с пометкой Required/NotRequired
- Связи один-ко-многим настраивать только в сущности «многие»
- Связи многие-ко-многим с промежуточной таблицей, только в конфигурации левой таблицы.
- Настройка поведения при удалении (Restrict/SetNull/Cascade)

#### Контекст (DbContextProjectName.cs)
- DbSet свойства во множественном числе
- Добавление в алфавитном порядке среди других DbSet
- Конфигурации подключаются автоматически через ApplyConfigurationsFromAssembly()

### Этап 2: Создание модели

#### Файл

`src/ProjectName/Datasource.Ef/Models/EntityName.cs`

```csharp
namespace Datasource.Ef.Models;

public class EntityName
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    public virtual ICollection<RelatedEntity> RelatedEntities { get; set; } = [];
}
```

Пример многие-к-одному (обязательная связь):

```csharp
namespace Datasource.Ef.Models;

public class EntityName
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    public int EntityId { get; set; }
    public virtual Entity Entity { get; set; } = default!;
}
```

Пример многие-к-одному (необязательная связь):

```csharp
namespace Datasource.Ef.Models;

public class EntityName
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    public int? EntityId { get; set; }
    public virtual Entity? Entity { get; set; }
}
```

---

### Этап 3: Создание конфигурации

#### Файл

`src/ProjectName/Datasource.Ef/Configurations/ConfigurationEntityName.cs`

```csharp
using Datasource.Ef.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace Datasource.Ef.Configurations;

public class ConfigurationEntityName : IEntityTypeConfiguration<EntityName>
{
    public void Configure(EntityTypeBuilder<EntityName> builder)
    {
        // Название таблицы во множественном числе
        builder.ToTable("EntityNames");

        // Первичный ключ с автоинкрементом
        builder.HasKey(p => p.Id);
        builder.Property(p => p.Id)
            .ValueGeneratedOnAdd();

        // Уникальный индекс для Name с префиксом IX_
        builder.HasIndex(p => p.Name, "IX_EntityName_Name")
            .IsUnique();

        // Строковые поля с ограничениями длины , обязательные
        builder.Property(p => p.StringProperty)
            .HasMaxLength(512)  // Длина будет указана в требованиях
            .IsRequired();      // Требуется или Опционально указывается в требованиях

        // Строковые поля с ограничениями длины , опциональные - nullable
        builder.Property(p => p.NullStringProperty)
            .HasMaxLength(7)
            .IsRequired(false);

        // Строковые поля без ограничений длины
        builder.Property(p => p.StringProperty)
            .HasColumnType("text");

        // Числовые поля с точностью
        builder.Property(p => p.DecimalProperty)
            .HasColumnType("numeric(7,2)")
            .IsRequired(false);

        // Булевы поля без дополнительной конфигурации
        builder.Property(p => p.BoolProperty);

        // Foreign Key с обязательным значением , 
        // добавляем false и значение не обязаельно.
        builder.Property(p => p.EntityId)
            .IsRequired();
        
        // Foreign Key с значением по умолчанию
        builder.Property(p => p.EntityId)
            .IsRequired()
            .HasDefaultValue(0);

        // Связи один-ко-многим (в сущности «многие»)
        
        // Cascade - зависимые сущности удаляются вместе с родительской
        builder.HasOne(p => p.Entity)
            .WithMany(e => e.RelatedEntities)
            .HasForeignKey(p => p.EntityId)
            .OnDelete(DeleteBehavior.Cascade);

        // Restrict - нельзя удалить родительскую сущность, если есть связанные
        builder.HasOne(p => p.Entity)
            .WithMany(p => p.RelatedEntities)
            .HasForeignKey(p => p.EntityId)
            .OnDelete(DeleteBehavior.Restrict);

        // SetNull - при удалении родителя устанавливается NULL (необязательная связь)
        builder.HasOne(p => p.Entity)
            .WithMany(p => p.RelatedEntities)
            .HasForeignKey(p => p.EntityId)
            .OnDelete(DeleteBehavior.SetNull);

        // Связи многие-ко-многим (только в одной конфигурации)
        builder.HasMany(p => p.LeftEntities)
            .WithMany(t => t.RightEntities)
            .UsingEntity<Dictionary<string, object>>(
                "LeftEntitiesRightEntities",
                right => right
                    .HasOne<RightEntity>()
                    .WithMany()
                    .HasForeignKey("RightEntityId")
                    .HasPrincipalKey(p => p.Id)
                    .HasConstraintName("FK_LeftEntitiesRightEntities_RightEntityId"),
                left => left
                    .HasOne<LeftEntity>()
                    .WithMany()
                    .HasForeignKey("LeftEntityId")
                    .HasPrincipalKey(p => p.Id)
                    .HasConstraintName("FK_LeftEntitiesRightEntities_LeftEntityId"),
                join =>
                {
                    join.ToTable("LeftEntitiesRightEntities");
                    join.Property<int>("LeftEntityId").HasColumnName("LeftEntityId");
                    join.Property<int>("RightEntityId").HasColumnName("RightEntityId");
                    // Составной первичный ключ (обязательно)
                    join.HasKey("LeftEntityId", "RightEntityId").HasName("IX_LeftEntitiesRightEntities_LeftEntityId_RightEntityId");
                    // Отдельные индексы для каждого FK (рекомендуется)
                    join.HasIndex("LeftEntityId").HasDatabaseName("IX_LeftEntitiesRightEntities_LeftEntityId");
                    join.HasIndex("RightEntityId").HasDatabaseName("IX_LeftEntitiesRightEntities_RightEntityId");
                });
    }
}
```

---

### Этап 4: Создание контекста

#### Файл

`src/ProjectName/Datasource.Ef/Contexts/DbContextProjectName.cs`

Добавить свойство DbSet в класс `DbContextProjectName`:

```csharp
public virtual DbSet<EntityName> EntityNames { get; set; }
```
