# Этап 5: Создание репозитория (Repositories.Ef/Api/)


### Требования

#### Общие требования для всех типов связей:
- Название классов опций: `EntityNameQueryOptions` и `EntityNameCommandOptions` (если нужны)
- Интерфейс и реализация репозитория должны находиться в одном файле
- Репозиторий получает `DbContextBlog` через DI (Scoped)
- Репозитории НЕ вызывают `SaveChanges/SaveChangesAsync` и НЕ управляют транзакциями
- Коммит выполняется через `IUnitOfWork.SaveChanges/SaveChangesAsync` на уровне сервисов
- Методы чтения — асинхронные и принимают `CancellationToken cancellationToken = default`
- Методы команд (Create/Update/Delete) могут быть синхронными, если не содержат `await`
- Использовать `AsNoTracking()` для запросов на чтение
- Использовать `OrderParameters<TModel>` для гибкой сортировки в QueryOptions (если этот механизм есть в проекте)
- Регистрировать репозиторий как `AddScoped` в контейнере зависимостей (DI)
- Зарегистрировать `IUnitOfWork/UnitOfWork` как `AddScoped` в контейнере зависимостей (DI)
- Регистрацию выполнять в composition root проекта (например, `Program.cs` или `ServiceRegistration.cs`)

#### Нюансы по типам связей:

**1. Без связей (например, Quote):**
- НЕ создавать `EntityNameCommandOptions` (не нужны)
- `Update` без параметра options
- QueryOptions только с фильтрацией и сортировкой
- НЕ использовать Include в методах чтения

**2. Один-ко-одному (One-to-One):**
- Создавать `EntityNameCommandOptions` для обновления связанной сущности
- В `Update` обновлять связь через навигационное свойство
- В QueryOptions добавить `IncludeRelatedEntity` для загрузки связи
- Использовать Include только при необходимости

**3. Один-ко-многим (например, UsefulLinkType → UsefulLink):**
- Создавать `EntityNameCommandOptions` с коллекцией связанных ID
- В `Update` загружать коллекцию и обновлять связи
- В QueryOptions добавить `IncludeRelatedEntities` для загрузки коллекции
- Использовать `Collection().Load()` / `LoadAsync()` для обновления связей

**4. Много-ко-многим (например, Books ↔ Tags):**
- Создавать `EntityNameCommandOptions` с коллекциями связанных ID
- В `Create` и `Update` устанавливать связи через коллекции
- В QueryOptions добавить опции включения связанных коллекций
- Обновлять связи через присвоение коллекций

**5. Самореферентные связи (например, Topic → Topic):**
- Добавлять опции для фильтрации по родительским/дочерним элементам
- Использовать рекурсивные методы для загрузки иерархии
- В QueryOptions добавить опции для работы с иерархией
- Учитывать производительность при загрузке глубоких иерархий

### Внимание

- база данных может состоять из таблиц которые имеют следующие связи:
  - без связей (например, Quote ; RepositoryQuote)
  - один-ко-одному (One-to-One, нет примера в проекте)
  - один-ко-многим (One-to-Many, например, UsefulLinkType → UsefulLink ; RepositoryUsefulLinkType ↔ RepositoryUsefulLink
  - много-ко-многим (Many-to-Many, например, Books ↔ Tags ; RepositoryBook ↔ RepositoryTag)
  - самореферентные связи (Self-Referencing, например, Topic → Topic ; RepositoryTopic)

### Создание класса опций

**Файл**: `src/Blog/Repositories.Ef/Options/EntityNameOptions.cs`

```csharp
using Repositories.Ef.Ordering;

namespace Repositories.Ef.Options;

public class EntityNameQueryOptions
{
    public bool IncludeRelatedEntities { get; set; }

    public OrderParameters<EntityName>? OrderParameters { get; set; }

    public string? Query { get; set; }
    public int Skip { get; set; }
    public int Take { get; set; }
}

public class EntityNameCommandOptions
{
    public IEnumerable<int>? RelatedEntityIds { get; set; }
}
```

### Создание интерфейса и реализации репозитория (в одном файле)

**Файл**: `src/Blog/Repositories.Ef/Api/RepositoryEntityName.cs`

```csharp
using Datasource.Ef.Contexts;
using Datasource.Ef.Models;
using Microsoft.EntityFrameworkCore;
using Repositories.Ef.Options;

namespace Repositories.Ef.Api;

public interface IRepositoryEntityName
{
    EntityName Create(EntityName model);
    EntityName Update(EntityName model, EntityNameCommandOptions? options = null);
    void Delete(EntityName model);

    EntityName GetNew();

    Task<EntityName?> GetSingleOrDefaultAsync(int id, EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default);
    Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default);
}

public class RepositoryEntityName : IRepositoryEntityName
{
    private readonly DbContextBlog context;

    public RepositoryEntityName(DbContextBlog context)
    {
        this.context = context;
    }

    public EntityName Create(EntityName model)
    {
        context.Add(model);
        return model;
    }

    public EntityName Update(EntityName model, EntityNameCommandOptions? options = null)
    {
        context.EntityNames.Update(model);

        if (options?.RelatedEntityIds != null)
        {
            context.Entry(model).Collection(p => p.RelatedEntities).Load();

            model.RelatedEntities = context.RelatedEntities
                .Where(related => options.RelatedEntityIds.Contains(related.Id))
                .ToList();
        }

        return model;
    }

    public void Delete(EntityName model)
    {
        context.Remove(model);
    }

    public EntityName GetNew()
    {
        return new EntityName() { Name = string.Empty };
    }

    public async Task<EntityName?> GetSingleOrDefaultAsync(int id, EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default)
    {
        var query = context.EntityNames.AsNoTracking();

        if (options != null && options.IncludeRelatedEntities)
        {
            query = query.Include(p => p.RelatedEntities);
        }

        return await query.FirstOrDefaultAsync(p => p.Id == id, cancellationToken);
    }

    public async Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default)
    {
        var query = context.EntityNames.AsNoTracking();

        if (options != null && options.IncludeRelatedEntities)
        {
            query = query.Include(p => p.RelatedEntities);
        }

        if (!string.IsNullOrWhiteSpace(options?.Query))
        {
            var lowerQuery = options.Query.ToLowerInvariant();
            query = query.Where(p => p.Name.ToLower().Contains(lowerQuery));
        }

        if (options?.OrderParameters?.OrderColumns != null && options.OrderParameters.OrderColumns.Count > 0)
        {
            query = options.OrderParameters.ApplySorting(query, false);
        }

        if (options != null)
        {
            var safeSkip = Math.Max(0, options.Skip);
            var safeTake = Math.Clamp(options.Take <= 0 ? 50 : options.Take, 1, 200);

            query = query.Skip(safeSkip).Take(safeTake);
        }

        return await query.ToListAsync(cancellationToken);
    }
}
```

### Unit Of Work

**Файл**: `src/Blog/Repositories.Ef/Api/UnitOfWork.cs`

```csharp
using Microsoft.EntityFrameworkCore.Storage;
using System.ComponentModel;

namespace Repositories.Ef.Api;

public interface IUnitOfWork : IDisposable, IAsyncDisposable
{
    IRepositoryEntityName EntityNames { get; }

    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
    int SaveChanges();

    Task<IDbContextTransaction> BeginTransactionAsync(CancellationToken cancellationToken = default);
    Task CommitTransactionAsync(IDbContextTransaction transaction, CancellationToken cancellationToken = default);
    Task RollbackTransactionAsync(IDbContextTransaction transaction, CancellationToken cancellationToken = default);

    [Obsolete("Не вызывать вручную: жизненный цикл управляется DI (Scoped).", false)]
    [EditorBrowsable(EditorBrowsableState.Never)]
    new void Dispose();

    [Obsolete("Не вызывать вручную: жизненный цикл управляется DI (Scoped).", false)]
    [EditorBrowsable(EditorBrowsableState.Never)]
    new ValueTask DisposeAsync();
}
```

### Использование Unit Of Work в сервисе

```csharp
var transaction = await unitOfWork.BeginTransactionAsync(cancellationToken);
try
{
    repositoryEntityName.Create(modelA);
    repositoryOther.Update(modelB);

    await unitOfWork.SaveChangesAsync(cancellationToken);
    await unitOfWork.CommitTransactionAsync(transaction, cancellationToken);
}
catch
{
    await unitOfWork.RollbackTransactionAsync(transaction, cancellationToken);
    throw;
}
```

### Добавление регистрации сервисов

**Файл**: `src/Blog/Client/Middleware/ServiceRegistration.cs`

Добавить в метод `AddDependencyInjectionExt`:

```csharp
services.AddScoped<IRepositoryEntityName, RepositoryEntityName>();
services.AddScoped<IUnitOfWork, UnitOfWork>();
```