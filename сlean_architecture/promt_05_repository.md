# Этап 5: Создание репозитория (Repositories.Ef/Api/)


### Требования

#### Общие требования для всех типов связей:
- Название классов опций: `EntityNameQueryOptions` и `EntityNameCommandOptions` (если нужны)
- Интерфейс и реализация репозитория должны находиться в одном файле
- Репозиторий получает `DbContextBlog` через DI (Scoped)
- Для каждой Entity/модели, с которой работает сервисный слой, должен быть создан репозиторий (интерфейс + реализация), а также соответствующее свойство в `IUnitOfWork` (только если оно реально используется сервисами)
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

**1. База данных может состоять из таблиц, которые имеют следующие типы связей:**
  - без связей (например, Quote; RepositoryQuote)
  - один-ко-одному (One-to-One, нет примера в проекте)
  - один-ко-многим (One-to-Many, например, UsefulLinkType → UsefulLink; RepositoryUsefulLinkType ↔ RepositoryUsefulLink)
  - много-ко-многим (Many-to-Many, например, Books ↔ Tags; RepositoryBook ↔ RepositoryTag)
  - самореферентные связи (Self-Referencing, например, Topic → Topic; RepositoryTopic)

**2. Основная и связанная модели (общая логика репозиториев):**
- Если есть основная модель и связанные модели (например, `Order` и `OrderItem`), репозитории создаются для обеих моделей
- Репозиторий основной модели отвечает за CRUD самой сущности и управление связями через `CommandOptions` (например, список `OrderItemIds` для привязки товаров к заказу)
- Если у основной модели несколько разных связей (например, `Order` связан и с `OrderItem`, и с `Customer`, и с `DeliveryMethod`), то в репозитории основной модели должна быть отдельная логика управления каждой связью (через отдельные поля в `CommandOptions` и соответствующие блоки обновления связей)
- Репозиторий связанной модели отвечает за CRUD связанной сущности и чтение списков/одиночных записей, которые нужны сервисам (например, список товаров для dropdown, проверка существования, загрузка карточки товара)
- В `IUnitOfWork` перечисляются репозитории только тех моделей, которые реально используются сервисным слоем; `unitOfWork.RelatedEntities` — пример связанной модели и должен добавляться только если такая сущность/связь есть и она требуется сервисам (например, `unitOfWork.Orders` и `unitOfWork.OrderItems`)

**3. Без связей (например, Quote):**
- НЕ создавать `EntityNameCommandOptions` (не нужны)
- `Update` без параметра options
- QueryOptions только с фильтрацией и сортировкой
- НЕ использовать Include в методах чтения

**4. Один-ко-одному (One-to-One):**
- Создавать `EntityNameCommandOptions` для обновления связанной сущности
- В `Update` обновлять связь через навигационное свойство
- В QueryOptions добавить `IncludeRelatedEntity` для загрузки связи
- Использовать Include только при необходимости

**5. Один-ко-многим (например, UsefulLinkType → UsefulLink):**
- Создавать `EntityNameCommandOptions` с коллекцией связанных ID
- В `Update` загружать коллекцию и обновлять связи
- В QueryOptions добавить `IncludeRelatedEntities` для загрузки коллекции
- Использовать `Collection().Load()` / `LoadAsync()` для обновления связей

**6. Много-ко-многим (например, Books ↔ Tags):**
- Создавать `EntityNameCommandOptions` с коллекциями связанных ID
- В `Create` и `Update` устанавливать связи через коллекции
- В QueryOptions добавить опции включения связанных коллекций
- Обновлять связи через присвоение коллекций

**7. Самореферентные связи (например, Topic → Topic):**
- Добавлять опции для фильтрации по родительским/дочерним элементам
- Использовать рекурсивные методы для загрузки иерархии
- В QueryOptions добавить опции для работы с иерархией
- Учитывать производительность при загрузке глубоких иерархий

### Создание класса опций

**Файл**: `src/Blog/Repositories.Ef/Options/EntityNameOptions.cs`

```csharp
using Repositories.Ef.QueryPipeline;

namespace Repositories.Ef.Options;

public class EntityNameQueryOptions
{
    public bool IncludeRelatedEntities { get; set; }

    public OrderParameters<EntityName>? OrderParameters { get; set; }
    public SearchParameters<EntityName>? SearchParameters { get; set; }
    public FilterParameters<EntityName>? FilterParameters { get; set; }
    public PaginationParameters? PaginationParameters { get; set; }
}

public class EntityNameCommandOptions
{
    public IEnumerable<int>? RelatedEntityIds { get; set; }
}
```

### QueryPipeline ядро репозитория (поиск/фильтрация/сортировка/пагинация)

Эти классы реализуются **один раз на проект** в `Repositories.Ef/QueryPipeline` и используются всеми репозиториями через `*QueryOptions`.

**Файл**: `src/Blog/Repositories.Ef/QueryPipeline/PaginationParameters.cs`

```csharp
namespace Repositories.Ef.QueryPipeline;

public class PaginationParameters
{
    public int? Skip { get; set; }
    public int? Take { get; set; }

    public PaginationParameters(int? skip = null, int? take = null)
    {
        Skip = skip;
        Take = take;
    }

    public IQueryable<TModel> ApplyPagination<TModel>(IQueryable<TModel> query) where TModel : class
    {
        if (Skip.HasValue)
        {
            query = query.Skip(Skip.Value);
        }

        if (Take.HasValue)
        {
            query = query.Take(Take.Value);
        }

        return query;
    }
}
```

**Файл**: `src/Blog/Repositories.Ef/QueryPipeline/OrderParameters.cs`

```csharp
using System.Linq.Expressions;

namespace Repositories.Ef.QueryPipeline;

public class OrderParameters<TModel> where TModel : class
{
    public List<(Expression<Func<TModel, object>> KeySelector, bool IsDescending, bool afterTake)> OrderColumns { get; set; }

    public OrderParameters(params (Expression<Func<TModel, object>> KeySelector, bool IsDescending, bool afterTake)[] orderColumns)
    {
        OrderColumns = [.. orderColumns];
    }

    public OrderParameters(params (Expression<Func<TModel, object>> KeySelector, bool IsDescending)[] orderColumns)
    {
        OrderColumns = [.. orderColumns.Select(p => (p.KeySelector, p.IsDescending, false))];
    }

    public OrderParameters(params Expression<Func<TModel, object>>[] orderColumns)
    {
        OrderColumns = [.. orderColumns.Select(keySelector => (keySelector, false, false))];
    }

    public void AddSortColumn(Expression<Func<TModel, object>> keySelector, bool isDescending = false, bool afterTake = false)
    {
        OrderColumns.Add((keySelector, isDescending, afterTake));
    }

    public IOrderedQueryable<TModel> ApplySorting(IQueryable<TModel> query, bool afterTake = false)
    {
        var tempOrderColumns = OrderColumns.Where(p => p.afterTake == afterTake);

        var firstColumn = tempOrderColumns.FirstOrDefault();

        if (firstColumn == default)
        {
            return (IOrderedQueryable<TModel>)query;
        }

        var orderedQuery = firstColumn.IsDescending
            ? query.OrderByDescending(firstColumn.KeySelector)
            : query.OrderBy(firstColumn.KeySelector);

        foreach (var orderColumn in tempOrderColumns.Skip(1))
        {
            var keySelector = orderColumn.KeySelector;

            orderedQuery = orderColumn.IsDescending
                ? orderedQuery.ThenByDescending(keySelector)
                : orderedQuery.ThenBy(keySelector);
        }

        return orderedQuery;
    }
}
```

**Файл**: `src/Blog/Repositories.Ef/QueryPipeline/FilterParameters.cs`

```csharp
using System.Linq.Expressions;

namespace Repositories.Ef.QueryPipeline;

public class FilterParameters<TModel> where TModel : class
{
    public List<FilterColumn<TModel>> FilterColumns { get; set; }

    public string? Condition { get; set; }

    public FilterParameters(params Expression<Func<TModel, bool>>[] filterColumns)
    {
        FilterColumns = [.. filterColumns.Select(filter => new FilterColumn<TModel>(filter, null))];
    }

    public void AddFilterColumn(Expression<Func<TModel, bool>> predicate)
    {
        FilterColumns.Add(new FilterColumn<TModel>(predicate, null));
    }

    public void AddFilterColumn(Expression<Func<TModel, bool>> predicate, string? condition)
    {
        FilterColumns.Add(new FilterColumn<TModel>(predicate, NormalizeCondition(condition)));
    }

    public IQueryable<TModel> ApplyFiltering(IQueryable<TModel> query)
    {
        if (FilterColumns.Count == 0)
        {
            return query;
        }

        var defaultCondition = NormalizeCondition(Condition);

        var predicate = FilterColumns[0].Predicate;

        foreach (var filter in FilterColumns.Skip(1))
        {
            var isOr = string.Equals(filter.Condition ?? defaultCondition, "or", StringComparison.OrdinalIgnoreCase);
            predicate = Combine(predicate, filter.Predicate, isOr);
        }

        return query.Where(predicate);
    }

    private static Expression<Func<TModel, bool>> Combine(Expression<Func<TModel, bool>> left, Expression<Func<TModel, bool>> right, bool isOr)
    {
        var parameter = Expression.Parameter(typeof(TModel));
        var leftBody = new ReplaceParameterVisitor(left.Parameters[0], parameter).Visit(left.Body);
        var rightBody = new ReplaceParameterVisitor(right.Parameters[0], parameter).Visit(right.Body);

        var body = isOr
            ? Expression.OrElse(leftBody!, rightBody!)
            : Expression.AndAlso(leftBody!, rightBody!);

        return Expression.Lambda<Func<TModel, bool>>(body, parameter);
    }

    private static string NormalizeCondition(string? condition)
    {
        return string.Equals(condition, "or", StringComparison.OrdinalIgnoreCase) ? "or" : "and";
    }

    public record FilterColumn<TFilter>(
        Expression<Func<TFilter, bool>> Predicate,
        string? Condition) where TFilter : class;

    private sealed class ReplaceParameterVisitor : ExpressionVisitor
    {
        private readonly ParameterExpression source;
        private readonly ParameterExpression target;

        public ReplaceParameterVisitor(ParameterExpression source, ParameterExpression target)
        {
            this.source = source;
            this.target = target;
        }

        protected override Expression VisitParameter(ParameterExpression node)
        {
            return node == source ? target : base.VisitParameter(node);
        }
    }
}
```

**Файл**: `src/Blog/Repositories.Ef/QueryPipeline/SearchParameters.cs`

```csharp
using System.Linq.Expressions;

namespace Repositories.Ef.QueryPipeline;

public class SearchParameters<TModel> where TModel : class
{
    public List<Expression<Func<TModel, bool>>> SearchColumns { get; set; }

    public SearchParameters(params Expression<Func<TModel, bool>>[] searchColumns)
    {
        SearchColumns = [.. searchColumns];
    }

    public void AddSearchColumn(Expression<Func<TModel, bool>> predicate)
    {
        SearchColumns.Add(predicate);
    }

    public IQueryable<TModel> ApplySearching(IQueryable<TModel> query)
    {
        if (SearchColumns.Count == 0)
        {
            return query;
        }

        var predicate = SearchColumns[0];

        foreach (var searchColumn in SearchColumns.Skip(1))
        {
            predicate = Combine(predicate, searchColumn);
        }

        return query.Where(predicate);
    }

    private static Expression<Func<TModel, bool>> Combine(Expression<Func<TModel, bool>> left, Expression<Func<TModel, bool>> right)
    {
        var parameter = Expression.Parameter(typeof(TModel));
        var leftBody = new ReplaceParameterVisitor(left.Parameters[0], parameter).Visit(left.Body);
        var rightBody = new ReplaceParameterVisitor(right.Parameters[0], parameter).Visit(right.Body);

        var body = Expression.AndAlso(leftBody!, rightBody!);

        return Expression.Lambda<Func<TModel, bool>>(body, parameter);
    }

    private sealed class ReplaceParameterVisitor : ExpressionVisitor
    {
        private readonly ParameterExpression source;
        private readonly ParameterExpression target;

        public ReplaceParameterVisitor(ParameterExpression source, ParameterExpression target)
        {
            this.source = source;
            this.target = target;
        }

        protected override Expression VisitParameter(ParameterExpression node)
        {
            return node == source ? target : base.VisitParameter(node);
        }
    }
}
```

### Создание интерфейса и реализации репозитория (в одном файле)

**Файл**: `src/Blog/Repositories.Ef/Api/RepositoryEntityName.cs`

```csharp
using Datasource.Ef.Contexts;
using Datasource.Ef.Models;
using Microsoft.EntityFrameworkCore;
using Repositories.Ef.Options;
using Repositories.Ef.QueryPipeline;

namespace Repositories.Ef.Api;

public interface IRepositoryEntityName
{
    EntityName Create(EntityName model, EntityNameCommandOptions? options = null);
    EntityName Update(EntityName model, EntityNameCommandOptions? options = null);
    void Delete(EntityName model);

    EntityName GetNew();

    Task<EntityName?> GetSingleOrDefaultAsync(int id, EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default);
    Task<int> CountAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default);
    Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default);
}

public class RepositoryEntityName : IRepositoryEntityName
{
    private readonly DbContextBlog context;

    public RepositoryEntityName(DbContextBlog context)
    {
        this.context = context;
    }

    public EntityName Create(EntityName model, EntityNameCommandOptions? options = null)
    {
        context.Add(model);

        if (options?.RelatedEntityIds != null)
        {
            context.Entry(model).Collection(p => p.RelatedEntities).Load();

            model.RelatedEntities = context.RelatedEntities
                .Where(related => options.RelatedEntityIds.Contains(related.Id))
                .ToList();
        }

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

    public async Task<int> CountAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default)
    {
        var query = context.EntityNames.AsNoTracking();

        if (options?.SearchParameters?.SearchColumns != null && options.SearchParameters.SearchColumns.Count > 0)
        {
            query = options.SearchParameters.ApplySearching(query);
        }

        if (options?.FilterParameters?.FilterColumns != null && options.FilterParameters.FilterColumns.Count > 0)
        {
            query = options.FilterParameters.ApplyFiltering(query);
        }

        return await query.CountAsync(cancellationToken);
    }

    public async Task<IEnumerable<EntityName>> GetListAsync(EntityNameQueryOptions? options = null, CancellationToken cancellationToken = default)
    {
        var query = context.EntityNames.AsNoTracking();

        if (options != null && options.IncludeRelatedEntities)
        {
            query = query.Include(p => p.RelatedEntities);
        }

        if (options?.SearchParameters?.SearchColumns != null && options.SearchParameters.SearchColumns.Count > 0)
        {
            query = options.SearchParameters.ApplySearching(query);
        }

        if (options?.FilterParameters?.FilterColumns != null && options.FilterParameters.FilterColumns.Count > 0)
        {
            query = options.FilterParameters.ApplyFiltering(query);
        }

        if (options?.OrderParameters?.OrderColumns != null && options.OrderParameters.OrderColumns.Count > 0)
        {
            query = options.OrderParameters.ApplySorting(query, false);
        }

        if (options?.PaginationParameters != null)
        {
            query = options.PaginationParameters.ApplyPagination(query);
        }

        if (options?.OrderParameters?.OrderColumns != null && options.OrderParameters.OrderColumns.Count > 0)
        {
            query = options.OrderParameters.ApplySorting(query, true);
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
    IRepositoryRelatedEntity RelatedEntities { get; }

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
    unitOfWork.EntityNames.Create(modelA);
    unitOfWork.RelatedEntities.Update(modelB);

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

Добавить в метод `AddDependencyInjectionExt` (при необходимости добавить `using Datasource.Ef.Models;` и `using Services.QueryPipeline;`):

```csharp
services.AddScoped<IRepositoryEntityName, RepositoryEntityName>();
services.AddScoped<IUnitOfWork, UnitOfWork>();
services.AddScoped<IEntityFieldSelectors<EntityName>, EntityNameFieldSelectors>();
services.AddScoped<IEntityNameQueryPipelineValidator, EntityNameQueryPipelineValidator>();
```
