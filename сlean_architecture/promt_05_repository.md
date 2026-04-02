# Этап 5: Создание репозитория (Repositories.Ef/Api/)

### Внимание

- база данных может состоять из таблиц которые имеют следующие связи:
  - без связей (например, Quote ; RepositoryQuote)
  - один-ко-одному (One-to-One, нет примера в проекте)
  - один-ко-многим (One-to-Many, например, UsefulLinkType → UsefulLink ; RepositoryUsefulLinkType ↔ RepositoryUsefulLink 
  - много-ко-многим (Many-to-Many, например, Books ↔ Tags ; RepositoryBook ↔ RepositoryTag)
  - самореферентные связи (Self-Referencing, например, Topic → Topic ; RepositoryTopic)

### Требования

#### Общие требования для всех типов связей:
- Название класса: `EntityNameQueryOptions` и `EntityNameCommandOptions` (если нужны)
- Интерфейс и реализация репозитория должны находиться в одном файле
- Использовать `IDbContextFactory<DbContextBlog>` для создания контекста
- Все методы асинхронные
- Использовать `AsNoTracking()` для запросов на чтение
- Использовать `OrderParameters<TModel>` для гибкой сортировки в QueryOptions
- Регистрировать репозиторий как `AddScoped` в контейнере зависимостей (DI)
- Зарегистрировать в секции репозиториев файла `ServiceRegistration.cs`

#### Нюансы по типам связей:

**1. Без связей (например, Quote):**
- НЕ создавать `EntityNameCommandOptions` (не нужны)
- Метод `UpdateAsync` без параметра options
- Простые QueryOptions только с фильтрацией и сортировкой
- НЕ использовать Include в методах чтения

**2. Один-ко-одному (One-to-One):**
- Создавать `EntityNameCommandOptions` для обновления связанной сущности
- В `UpdateAsync` обновлять связь через навигационное свойство
- В QueryOptions добавить `IncludeRelatedEntity` для загрузки связи
- Использовать Include только при необходимости

**3. Один-ко-многим (например, UsefulLinkType → UsefulLink):**
- Создавать `EntityNameCommandOptions` с коллекцией связанных ID
- В `UpdateAsync` загружать коллекцию и обновлять связи
- В QueryOptions добавить `IncludeRelatedEntities` для загрузки коллекции
- Использовать `Collection().LoadAsync()` для обновления связей

**4. Много-ко-многим (например, Books ↔ Tags):**
- Создавать `EntityNameCommandOptions` с коллекциями связанных ID
- В `CreateAsync` и `UpdateAsync` устанавливать связи через коллекции
- В QueryOptions добавить опции включения связанных коллекций
- Обновлять связи через присвоение коллекций

**5. Самореферентные связи (например, Topic → Topic):**
- Добавлять опции для фильтрации по родительским/дочерним элементам
- Использовать рекурсивные методы для загрузки иерархии
- В QueryOptions добавить опции для работы с иерархией
- Учитывать производительность при загрузке глубоких иерархий

### Создание класса опций

**Файл**: `src/Blog/Repositories.Ef/Options/EntityNameOptions.cs`

```csharp
using Repositories.Ef.Ordering;

namespace Repositories.Ef.Options;

public class EntityNameQueryOptions
{
    public bool IncludeRelatedEntities { get; set; }
    
    public OrderParameters<EntityName>? OrderParameters { get; set; }
    
    // Добавить другие опции фильтрации и сортировки
}

public class EntityNameCommandOptions
{
    public IEnumerable<int>? RelatedEntityIds { get; set; }
    // Добавить другие связанные сущности по необходимости
}
```

### Создание интерфейса репозитория

**Файл**: `src/Blog/Repositories.Ef/Api/RepositoryEntityName.cs`

```csharp
using Datasource.Ef.Models;
using Repositories.Ef.Options;

namespace Repositories.Ef.Api;

public interface IRepositoryEntityName
{
    Task<EntityName> CreateAsync(EntityName model);
    Task<EntityName> UpdateAsync(EntityName model, EntityNameCommandOptions? options = null);
    Task DeleteAsync(EntityName model);

    EntityName GetNew();
    Task<EntityName?> GetSingleOrDefaultAsync(int id, EntityNameQueryOptions? options = null);
    Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null);
}
```

### Создание реализации репозитория

```csharp
using Datasource.Ef.Contexts;
using Datasource.Ef.Models;
using Microsoft.EntityFrameworkCore;
using Repositories.Ef.Options;

namespace Repositories.Ef.Api;

public class RepositoryEntityName : IRepositoryEntityName
{
    private readonly IDbContextFactory<DbContextBlog> contextFactory;

    public RepositoryEntityName(IDbContextFactory<DbContextBlog> contextFactory)
    {
        this.contextFactory = contextFactory;
    }

    public async Task<EntityName> CreateAsync(EntityName model)
    {
        using var context = await contextFactory.CreateDbContextAsync();

        await context.AddAsync(model);
        await context.SaveChangesAsync();

        return model;
    }

    public async Task<EntityName> UpdateAsync(EntityName model, EntityNameCommandOptions? options = null)
    {
        using var context = await contextFactory.CreateDbContextAsync();

        context.EntityNames.Update(model);

        if (options?.RelatedEntityIds != null)
        {
            await context.Entry(model).Collection(p => p.RelatedEntities).LoadAsync();
            
            model.RelatedEntities = await context.RelatedEntities
                .Where(related => options.RelatedEntityIds.Contains(related.Id))
                .ToListAsync();
        }

        await context.SaveChangesAsync();

        return model;
    }

    public async Task DeleteAsync(EntityName model)
    {
        using var context = await contextFactory.CreateDbContextAsync();

        context.Remove(model);
        await context.SaveChangesAsync();
    }

    public EntityName GetNew()
    {
        return new EntityName() { Name = string.Empty };
    }

    public async Task<EntityName?> GetSingleOrDefaultAsync(int id, EntityNameQueryOptions? options = null)
    {
        using var context = await contextFactory.CreateDbContextAsync();

        var query = context.EntityNames.AsNoTracking();

        if (options != null && options.IncludeRelatedEntities)
        {
            query = query.Include(p => p.RelatedEntities);
        }

        return await query.FirstOrDefaultAsync(p => p.Id == id);
    }

    public async Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null)
    {
        using var context = await contextFactory.CreateDbContextAsync();

        var query = context.EntityNames.AsNoTracking();

        if (options != null && options.IncludeRelatedEntities)
        {
            query = query.Include(p => p.RelatedEntities);
        }

        if (options?.OrderParameters?.OrderColumns != null && options?.OrderParameters?.OrderColumns.Count > 0)
        {
            query = options.OrderParameters.ApplySorting(query, false);
        }

        return await query.ToListAsync();
    }
}
```

### Добавление регистрации сервисов

**Файл**: `src/Blog/Client/Middleware/ServiceRegistration.cs`

Добавить в метод `AddDependencyInjectionExt`:

```csharp
// Регистрация репозиториев
services.AddScoped<IRepositoryEntityName, RepositoryEntityName>();
```
