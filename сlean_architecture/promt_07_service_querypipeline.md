# QueryPipeline ядро сервисного слоя (один раз на проект)

Эти классы создаются **один раз на проект** и используются всеми сущностями.

## DTO-модели QueryPipeline

**Папка**: `src/ProjectName/Services/Models/QueryPipeline/`

**Файл**: `src/ProjectName/Services/Models/QueryPipeline/SearchField.cs`

```csharp
namespace Services.Models.QueryPipeline;

public record SearchField(
    ICollection<string> Fields,
    string Key,
    string Operator);
```

**Файл**: `src/ProjectName/Services/Models/QueryPipeline/FilterField.cs`

```csharp
namespace Services.Models.QueryPipeline;

public record FilterField(
    string Field,
    string Operator,
    string? Condition,
    object? Value,
    bool? IgnoreCase);
```

**Файл**: `src/ProjectName/Services/Models/QueryPipeline/OrderField.cs`

```csharp
namespace Services.Models.QueryPipeline;

public record OrderField(
    string Field,
    string Direction,
    bool AfterTake);
```

**Файл**: `src/ProjectName/Services/Models/QueryPipeline/PaginationField.cs`

```csharp
namespace Services.Models.QueryPipeline;

public record PaginationField(
    int? Skip,
    int? Take);
```

## Сервисный слой QueryPipeline (валидация и билдеры)

**Папка**: `src/ProjectName/Services/QueryPipeline/`

**Файл**: `src/ProjectName/Services/QueryPipeline/QueryPipelineOperators.cs`

```csharp
namespace Services.QueryPipeline;

public static class QueryPipelineOperators
{
    public static readonly IReadOnlySet<string> SearchOperators = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "contains",
        "startswith",
        "endswith"
    };

    public static readonly IReadOnlySet<string> FilterOperators = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "eq",
        "equal",
        "notequal",
        "greaterthan",
        "greaterthanorequal",
        "lessthan",
        "lessthanorequal",
        "contains",
        "doesnotcontain",
        "startswith",
        "endswith",
        "isnull",
        "isnotnull",
        "isempty",
        "isnotempty",
        "in",
        "notin"
    };

    public static readonly IReadOnlySet<string> OrderDirections = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "asc",
        "ascending",
        "desc",
        "descending"
    };
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/IEntityFieldSelectors.cs`

```csharp
using System.Linq.Expressions;

namespace Services.QueryPipeline;

public enum FieldType
{
    String,
    Number,
    DateTime,
    Boolean,
    Guid
}

public interface IEntityFieldSelectors<TEntity>
{
    Expression<Func<TEntity, string?>>? GetSearchSelector(string fieldName);
    (Expression<Func<TEntity, object>>? Selector, FieldType Type) GetFilterSelector(string fieldName);
    Expression<Func<TEntity, object>>? GetOrderSelector(string fieldName);
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/IQueryPipelineFieldMap.cs`

```csharp
namespace Services.QueryPipeline;

public interface IQueryPipelineFieldMap
{
    IReadOnlySet<string> SearchFields { get; }
    IReadOnlySet<string> FilterFields { get; }
    IReadOnlySet<string> OrderFields { get; }
    IReadOnlySet<string> SearchOperators { get; }
    IReadOnlySet<string> FilterOperators { get; }
    IReadOnlySet<string> OrderDirections { get; }
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/IQueryPipelineValidator.cs`

```csharp
using Services.Models.QueryPipeline;

namespace Services.QueryPipeline;

public interface IQueryPipelineValidator
{
    void Validate(
        ICollection<SearchField>? searchFields,
        ICollection<FilterField>? filterFields,
        ICollection<OrderField>? orderFields);

    void ValidateSearch(ICollection<SearchField>? searchFields);
    void ValidateFilter(ICollection<FilterField>? filterFields);
    void ValidateOrder(ICollection<OrderField>? orderFields);
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/QueryPipelineValidator.cs`

```csharp
using Services.Exceptions;
using Services.Models.QueryPipeline;

namespace Services.QueryPipeline;

public class QueryPipelineValidator : IQueryPipelineValidator
{
    private readonly IQueryPipelineFieldMap fieldMap;

    public QueryPipelineValidator(IQueryPipelineFieldMap fieldMap)
    {
        this.fieldMap = fieldMap ?? throw new ArgumentNullException(nameof(fieldMap));
    }

    public void Validate(
        ICollection<SearchField>? searchFields,
        ICollection<FilterField>? filterFields,
        ICollection<OrderField>? orderFields)
    {
        ValidateSearch(searchFields);
        ValidateFilter(filterFields);
        ValidateOrder(orderFields);
    }

    public void ValidateSearch(ICollection<SearchField>? searchFields)
    {
        if (searchFields == null || searchFields.Count == 0)
        {
            return;
        }

        foreach (var searchField in searchFields)
        {
            if (searchField.Fields == null || searchField.Fields.Count == 0)
            {
                throw new BadRequestException("Поле поиска должно включать хотя бы одно поле.");
            }

            foreach (var field in searchField.Fields)
            {
                if (!fieldMap.SearchFields.Contains(field))
                {
                    throw new BadRequestException($"Неизвестное поле поиска: {field}.");
                }
            }

            if (!string.IsNullOrWhiteSpace(searchField.Operator)
                && !fieldMap.SearchOperators.Contains(searchField.Operator))
            {
                throw new BadRequestException($"Неизвестный оператор поиска: {searchField.Operator}.");
            }
        }
    }

    public void ValidateFilter(ICollection<FilterField>? filterFields)
    {
        if (filterFields == null || filterFields.Count == 0)
        {
            return;
        }

        foreach (var filterField in filterFields)
        {
            if (string.IsNullOrWhiteSpace(filterField.Field))
            {
                throw new BadRequestException("Поле фильтра обязательно.");
            }

            if (!fieldMap.FilterFields.Contains(filterField.Field))
            {
                throw new BadRequestException($"Неизвестное поле фильтра: {filterField.Field}.");
            }

            if (!string.IsNullOrWhiteSpace(filterField.Operator)
                && !fieldMap.FilterOperators.Contains(filterField.Operator))
            {
                throw new BadRequestException($"Неизвестный оператор фильтра: {filterField.Operator}.");
            }
        }
    }

    public void ValidateOrder(ICollection<OrderField>? orderFields)
    {
        if (orderFields == null || orderFields.Count == 0)
        {
            return;
        }

        foreach (var orderField in orderFields)
        {
            if (string.IsNullOrWhiteSpace(orderField.Field))
            {
                throw new BadRequestException("Поле сортировки обязательно.");
            }

            if (!fieldMap.OrderFields.Contains(orderField.Field))
            {
                throw new BadRequestException($"Неизвестное поле сортировки: {orderField.Field}.");
            }

            if (!string.IsNullOrWhiteSpace(orderField.Direction)
                && !fieldMap.OrderDirections.Contains(orderField.Direction))
            {
                throw new BadRequestException($"Неизвестное направление сортировки: {orderField.Direction}.");
            }
        }
    }
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/PaginationParametersBuilder.cs`

```csharp
using Repositories.Ef.QueryPipeline;
using Services.Models.QueryPipeline;

namespace Services.QueryPipeline;

public static class PaginationParametersBuilder
{
    public static PaginationParameters? Build(PaginationField? paginationField)
    {
        if (paginationField == null || (paginationField.Skip == null && paginationField.Take == null))
        {
            return null;
        }

        return new PaginationParameters(paginationField.Skip, paginationField.Take);
    }
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/SearchParametersBuilder.cs`

Перенести “как есть” из FlightChat:  
`FlightChatApp/Services/QueryPipeline/SearchParametersBuilder.cs`

**Файл**: `src/ProjectName/Services/QueryPipeline/OrderParametersBuilder.cs`

Перенести “как есть” из FlightChat:  
`FlightChatApp/Services/QueryPipeline/OrderParametersBuilder.cs`

**Файл**: `src/ProjectName/Services/QueryPipeline/FilterParametersBuilder.cs`

Перенести “как есть” из FlightChat:  
`FlightChatApp/Services/QueryPipeline/FilterParametersBuilder.cs`  
Дополнительно учесть блоки для `FieldType.Boolean` и `TryGetBool/AddBoolFilter` (см. ниже в promt_07_service.md).
