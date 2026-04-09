# Этап 6: Создание сервиса (Services/Api/)

### 6.1 Требования

- Сервисный слой реализует бизнес-операции для сущности `EntityName` поверх репозиториев и `IUnitOfWork`, без прямого доступа к `DbContext`
- Для сервиса должны быть созданы:
  - DTO-модели запросов/ответов в `src/ProjectName/Services/Models/EntityName.cs`
  - класс текстовых констант в `src/ProjectName/Services/Texts/EntityNameTexts.cs`
  - интерфейс `IServiceEntityName` и реализация `ServiceEntityName` в `src/ProjectName/Services/Api/ServiceEntityName.cs`
- Именование:
  - интерфейс: `IServiceEntityName`, реализация: `ServiceEntityName`
  - модели: `EntityName{Operation}Request/Response` и `EntityNameDisplay{Operation}Request/Response` (как в примере ниже)
- DI:
  - сервис регистрировать как `AddScoped<IServiceEntityName, ServiceEntityName>()`
  - сервис получает зависимости через конструктор: `IUnitOfWork`, `ILogger<ServiceEntityName>`, `IEntityNameQueryPipelineValidator`, `IEntityFieldSelectors<EntityName>`
- Асинхронность и отмена:
  - все публичные методы сервиса асинхронные и принимают `CancellationToken cancellationToken` (передавать дальше во все `...Async`)
- Транзакции и коммит:
  - `Create/Update/Delete` выполняются в транзакции `unitOfWork.BeginTransactionAsync(...)`
  - коммит изменений выполнять через `unitOfWork.SaveChangesAsync(...)` и `unitOfWork.CommitTransactionAsync(...)`
  - при ошибке обязательно вызывать `unitOfWork.RollbackTransactionAsync(...)` и пробрасывать исключение дальше
  - `Display*`, `DisplayInfo`, `DisplayList`, `DisplayRelatedEntityDropdownItems` не должны вызывать `SaveChanges` и не должны открывать транзакции
- Доступ к данным:
  - для чтения использовать репозитории из `unitOfWork` и при необходимости передавать `QueryOptions` (например, `IncludeRelatedEntities = true`)
  - для команд использовать `CommandOptions` для управления связями (например, `RelatedEntityIds`)
  - детали контрактов репозиториев/`IUnitOfWork`/`QueryOptions`/`CommandOptions` описаны в `promt_06_repository.md`; сервис не должен придумывать новые опции или свойства (например, `OrderByNameAsc`), которых нет в репозиторном промте
  - `Include*` опции добавлять только если у сущности есть соответствующие связи и они реально нужны в текущей операции; если связей несколько — в `QueryOptions` должно быть несколько `Include*` флагов (по одному на каждую связь), как описано в репозиторном промте
- Валидация входных данных:
  - если `request == null`, выбрасывать `BadRequestException` с сообщением `EntityNameTexts.Messages.Validation.RequestNull`
  - если метод принимает `Id` (DisplayUpdate/Update/DisplayDelete/Delete/DisplayInfo), то `Id` должен быть `> 0`, иначе `BadRequestException` с сообщением `EntityNameTexts.Messages.Validation.IdNotValid`
  - для `Create/Update` поле `Name` не должно быть пустым, иначе `BadRequestException` с сообщением `EntityNameTexts.Messages.Validation.NameEmpty`
  - для списков id (`...SelectedIds`):
    - `null` трактовать как пустую коллекцию
    - если есть элементы `<= 0`, выбрасывать `BadRequestException` с сообщением `EntityNameTexts.Messages.Validation.RelatedEntityIdsNotValid`
  - для `DisplayList`:
    - валидировать QueryPipeline через `queryPipelineValidator.Validate(...)` (неизвестные поля/операторы → `BadRequestException`)
    - валидировать пагинацию (`Skip >= 0`, `Take` в допустимом диапазоне), иначе `BadRequestException` с сообщением `EntityNameTexts.Messages.Validation.PaginationNotValid`
- Обработка отсутствующих данных:
  - если сущность не найдена, выбрасывать `NotFoundException` с сообщением из `EntityNameTexts.Messages.Error.NotFoundById`
- Кастомные исключения и глобальный маппинг в Web API:
  - вынесено в `promt_07_service_exceptions.md`
- Логирование и тексты:
  - в начале операций логировать `EntityNameTexts.Messages.Start.*`
  - при успешном завершении логировать `EntityNameTexts.Messages.Success.*`
  - при ошибке логировать `LogError(exception, "...")`, используя сообщения из `EntityNameTexts.Messages.Error.*`, и пробрасывать исключение
  - не логировать входные модели целиком; не логировать потенциально чувствительные данные
- Маппинг DTO:
  - DTO должны содержать все поля, необходимые UI/API; коллекции инициализировать пустыми (`= []`)
  - `Display*` методы возвращают данные для экранов/форм (включая dropdown-списки), а `Create/Update/Delete` — результат выполнения команды



**Файл**: `src/ProjectName/Services/Models/EntityName.cs`

```csharp
using Services.Models.QueryPipeline;

namespace Services.Models;

// === CREATE ===
public class EntityNameCreateRequest
{
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<int> RelatedEntityDropdownSelectedIds { get; set; } = [];
}

public class EntityNameDisplayCreateResponse
{
	public required string? Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<EntityNameRelatedEntityDropdownItem> RelatedEntityDropdownItems { get; set; } = [];
}

public class EntityNameCreateResponse
{
	public required int Id { get; set; }
}

// === UPDATE ===
public class EntityNameDisplayUpdateRequest
{
	public required int Id { get; set; }
}

public class EntityNameDisplayUpdateResponse
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<int> RelatedEntityDropdownSelectedIds { get; set; } = [];
	public required ICollection<EntityNameRelatedEntityDropdownItem> RelatedEntityDropdownItems { get; set; } = [];
}

public class EntityNameUpdateRequest
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<int> RelatedEntityDropdownSelectedIds { get; set; } = [];
}

public class EntityNameUpdateResponse
{
	public required int Id { get; set; }
}

// === DELETE ===
public class EntityNameDisplayDeleteRequest
{
	public required int Id { get; set; }
}

public class EntityNameDisplayDeleteResponse
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
}

public class EntityNameDeleteRequest
{
	public required int Id { get; set; }
}

public class EntityNameDeleteResponse
{
	public required bool IsDeleted { get; set; }
}

// === INFO ===
public class EntityNameInfoRequest
{
	public required int Id { get; set; }
}

public class EntityNameDisplayInfoResponse
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required int RelatedEntitiesCount { get; set; }
	public required ICollection<EntityNameRelatedEntityListModel> RelatedEntities { get; set; } = [];
}

// === LIST ===
public class EntityNameListRequest
{
	public ICollection<SearchField>? SearchFields { get; set; }
	public ICollection<FilterField>? FilterFields { get; set; }
	public ICollection<OrderField>? OrderFields { get; set; }
	public PaginationField? PaginationField { get; set; }
}

public class EntityNameDisplayListResponse
{
	public required ICollection<EntityNameListModel> EntityNames { get; set; } = [];
	public required int EntityNameCount { get; set; }
}

public class EntityNameListModel
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<EntityNameRelatedEntityListModel> RelatedEntities { get; set; } = [];
}

public class EntityNameRelatedEntityListModel
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
}

// === DROPDOWN ===

public class EntityNameDisplayRelatedEntityDropdownResponse
{
	public required ICollection<EntityNameRelatedEntityDropdownItem> RelatedEntityDropdownItems { get; set; } = [];
}

public class EntityNameRelatedEntityDropdownItem
{
	public required int Id { get; set; }
	public required string Name { get; set; }
}
```

### 6.2.0 QueryPipeline ядро (общие файлы)

Вынесено в `promt_07_service_querypipeline.md`.

### 6.2.1 QueryPipeline для списка (поиск/фильтр/сортировка/пагинация)

Нужно добавить в `Services/QueryPipeline` сущностно-специфичные классы:

**Файл**: `src/ProjectName/Services/QueryPipeline/EntityNameFieldSelectors.cs`

```csharp
using Datasource.Ef.Models;
using System.Linq.Expressions;

namespace Services.QueryPipeline;

public class EntityNameFieldSelectors : IEntityFieldSelectors<EntityName>
{
    public Expression<Func<EntityName, string?>>? GetSearchSelector(string fieldName)
    {
        return fieldName?.ToLowerInvariant() switch
        {
            "name" => x => x.Name,
            _ => null
        };
    }

    public (Expression<Func<EntityName, object>>? Selector, FieldType Type) GetFilterSelector(string fieldName)
    {
        return fieldName?.ToLowerInvariant() switch
        {
            "id" => (x => x.Id, FieldType.Number),
            "name" => (x => x.Name, FieldType.String),
            _ => (null, FieldType.String)
        };
    }

    public Expression<Func<EntityName, object>>? GetOrderSelector(string fieldName)
    {
        return fieldName?.ToLowerInvariant() switch
        {
            "id" => x => x.Id,
            "name" => x => x.Name,
            _ => null
        };
    }
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/EntityNameFieldMap.cs`

```csharp
namespace Services.QueryPipeline;

public static class EntityNameFieldMap
{
    public static readonly IReadOnlySet<string> SearchFields = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "name"
    };

    public static readonly IReadOnlySet<string> FilterFields = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "id",
        "name"
    };

    public static readonly IReadOnlySet<string> OrderFields = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
    {
        "id",
        "name"
    };

    public static IReadOnlySet<string> SearchOperators => QueryPipelineOperators.SearchOperators;
    public static IReadOnlySet<string> FilterOperators => QueryPipelineOperators.FilterOperators;
    public static IReadOnlySet<string> OrderDirections => QueryPipelineOperators.OrderDirections;
}

public class EntityNameFieldMapAdapter : IQueryPipelineFieldMap
{
    public IReadOnlySet<string> SearchFields => EntityNameFieldMap.SearchFields;
    public IReadOnlySet<string> FilterFields => EntityNameFieldMap.FilterFields;
    public IReadOnlySet<string> OrderFields => EntityNameFieldMap.OrderFields;
    public IReadOnlySet<string> SearchOperators => EntityNameFieldMap.SearchOperators;
    public IReadOnlySet<string> FilterOperators => EntityNameFieldMap.FilterOperators;
    public IReadOnlySet<string> OrderDirections => EntityNameFieldMap.OrderDirections;
}
```

**Файл**: `src/ProjectName/Services/QueryPipeline/EntityNameQueryPipelineValidator.cs`

```csharp
namespace Services.QueryPipeline;

public interface IEntityNameQueryPipelineValidator : IQueryPipelineValidator
{
}

public class EntityNameQueryPipelineValidator : QueryPipelineValidator, IEntityNameQueryPipelineValidator
{
    public EntityNameQueryPipelineValidator() : base(new EntityNameFieldMapAdapter())
    {
    }
}
```

Нюансы при переносе QueryPipeline ядра:
- `FieldType.Boolean` должен быть реализован в `FilterParametersBuilder`
- если используется фильтрация по списку дат (`in/notin`), эти операторы должны быть разрешены в `QueryPipelineOperators.FilterOperators`

Минимальные правки:

**Файл**: `src/ProjectName/Services/QueryPipeline/QueryPipelineOperators.cs`

```csharp
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
```

**Файл**: `src/ProjectName/Services/QueryPipeline/FilterParametersBuilder.cs`

```csharp
case FieldType.Boolean:
    if (TryGetBool(value, out var boolValue))
    {
        AddBoolFilter(filterParameters, selector, op, boolValue, condition);
    }
    break;

private static void AddBoolFilter(
    FilterParameters<TEntity> filterParameters,
    Expression<Func<TEntity, object>> selector,
    string op,
    bool value,
    string? condition)
{
    var parameter = selector.Parameters[0];
    var body = selector.Body;

    if (body is UnaryExpression { NodeType: ExpressionType.Convert } unary)
    {
        body = unary.Operand;
    }

    var valueConstant = Expression.Constant(value, body.Type);

    Expression? bodyExpression = op switch
    {
        "eq" or "equal" => Expression.Equal(body, valueConstant),
        "notequal" => Expression.NotEqual(body, valueConstant),
        _ => null
    };

    if (bodyExpression == null)
    {
        return;
    }

    filterParameters.AddFilterColumn(Expression.Lambda<Func<TEntity, bool>>(bodyExpression, parameter), condition);
}

private static bool TryGetBool(object? value, out bool result)
{
    if (value is bool boolValue)
    {
        result = boolValue;
        return true;
    }

    if (value is JsonElement jsonElement)
    {
        if (jsonElement.ValueKind == JsonValueKind.True)
        {
            result = true;
            return true;
        }

        if (jsonElement.ValueKind == JsonValueKind.False)
        {
            result = false;
            return true;
        }

        if (jsonElement.ValueKind == JsonValueKind.String)
        {
            var text = jsonElement.GetString();
            if (bool.TryParse(text, out var parsed))
            {
                result = parsed;
                return true;
            }
        }
    }

    if (value is string stringValue && bool.TryParse(stringValue, out var parsedBool))
    {
        result = parsedBool;
        return true;
    }

    result = default;
    return false;
}
```

### 6.3 Создание класса с текстами

**Файл**: `src/ProjectName/Services/Texts/EntityNameTexts.cs`

```csharp
namespace Services.Texts;

public static class EntityNameTexts
{
    public static class Messages
    {
        public static class Validation
        {
            public const string RequestNull = "Request не должен быть null";
            public const string IdNotValid = "Id должен быть больше 0";
            public const string NameEmpty = "Name не должен быть пустым";
            public const string RelatedEntityIdsNotValid = "Список идентификаторов связанных сущностей содержит недопустимые значения";
            public const string PaginationNotValid = "Параметры пагинации заданы некорректно";
        }

        public static class Start
        {
            public const string DisplayCreating = "Начало загрузки данных для создания EntityName";
            public const string DisplayUpdating = "Начало загрузки данных для обновления EntityName";
            public const string DisplayDeleting = "Начало загрузки данных для удаления EntityName";

            public const string Creating = "Начало выполнения создания EntityName";
            public const string Updating = "Начало выполнения обновления EntityName";
            public const string Deleting = "Начало выполнения удаления EntityName";

            public const string DisplayInfoLoading = "Начало загрузки данных для просмотра EntityName";
            public const string DisplayListLoading = "Начало загрузки данных списка EntityNames";
            public const string DisplayRelatedEntityDropdownLoading = "Начало загрузки данных выпадающего списка связанных сущностей";

        }

        public static class Success
        {
            public const string DisplayCreateCompleted = "Данные для создания EntityName успешно загружены";
            public const string DisplayUpdateCompleted = "Данные для обновления EntityName успешно загружены";
            public const string DisplayDeleteCompleted = "Данные для удаления EntityName успешно загружены";

            public const string CreateCompleted = "EntityName успешно создан";
            public const string UpdateCompleted = "EntityName успешно обновлен";
            public const string DeleteCompleted = "EntityName успешно удален";

            public const string DisplayInfoCompleted = "Данные для просмотра EntityName успешно загружены";
            public const string DisplayListCompleted = "Данные списка EntityNames успешно загружены";
            public const string DisplayRelatedEntityDropdownCompleted = "Данные выпадающего списка связанных сущностей успешно загружены";
        }

        public static class Error
        {
            public const string DisplayCreateError = "Не удалось загрузить данные для создания EntityName";
            public const string DisplayUpdateError = "Не удалось загрузить данные для обновления EntityName";
            public const string DisplayDeleteError = "Не удалось загрузить данные для удаления EntityName";

            public const string CreateError = "Не удалось создать EntityName";
            public const string UpdateError = "Не удалось обновить EntityName";
            public const string DeleteError = "Не удалось удалить EntityName";

            public const string DisplayInfoError = "Не удалось загрузить данные для просмотра EntityName";
            public const string DisplayListError = "Не удалось загрузить данные списка EntityNames";
            public const string DisplayRelatedEntityDropdownError = "Не удалось загрузить данные выпадающего списка связанных сущностей";

            public const string NotFoundById = "EntityName не был найден по id";
        }
    }
}
```

### 6.4 Создание интерфейса сервиса

**Файл**: `src/ProjectName/Services/Api/ServiceEntityName.cs`

```csharp
using System.Threading;
using Datasource.Ef.Models;
using Microsoft.Extensions.Logging;
using Repositories.Ef.Api;
using Repositories.Ef.Options;
using Repositories.Ef.QueryPipeline;
using Services.QueryPipeline;
using Services.Texts;
using Services.Exceptions;
using Services.Models;

namespace Services.Api;

public interface IServiceEntityName
{
    Task<EntityNameDisplayCreateResponse> DisplayCreateAsync(CancellationToken cancellationToken);
    Task<EntityNameCreateResponse> CreateAsync(EntityNameCreateRequest request, CancellationToken cancellationToken);

    Task<EntityNameDisplayUpdateResponse> DisplayUpdateAsync(EntityNameDisplayUpdateRequest request, CancellationToken cancellationToken);
    Task<EntityNameUpdateResponse> UpdateAsync(EntityNameUpdateRequest request, CancellationToken cancellationToken);

    Task<EntityNameDisplayDeleteResponse> DisplayDeleteAsync(EntityNameDisplayDeleteRequest request, CancellationToken cancellationToken);
    Task<EntityNameDeleteResponse> DeleteAsync(EntityNameDeleteRequest request, CancellationToken cancellationToken);
    
    Task<EntityNameDisplayInfoResponse> DisplayInfoAsync(EntityNameInfoRequest request, CancellationToken cancellationToken);
    Task<EntityNameDisplayListResponse> DisplayListAsync(EntityNameListRequest request, CancellationToken cancellationToken);
    Task<EntityNameDisplayRelatedEntityDropdownResponse> DisplayRelatedEntityDropdownItemsAsync(CancellationToken cancellationToken);
}
```

### 6.5 Создание реализации сервиса

```csharp
public class ServiceEntityName : IServiceEntityName
{
    private readonly IUnitOfWork unitOfWork;
    private readonly ILogger<ServiceEntityName> logger;
    private readonly IEntityNameQueryPipelineValidator queryPipelineValidator;
    private readonly IEntityFieldSelectors<EntityName> fieldSelectors;

    public ServiceEntityName(IUnitOfWork unitOfWork, ILogger<ServiceEntityName> logger, IEntityNameQueryPipelineValidator queryPipelineValidator, IEntityFieldSelectors<EntityName> fieldSelectors)
    {
        this.unitOfWork = unitOfWork;
        this.logger = logger;
        this.queryPipelineValidator = queryPipelineValidator;
        this.fieldSelectors = fieldSelectors;
    }

    public async Task<EntityNameDisplayCreateResponse> DisplayCreateAsync(CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayCreating);

        try
        {
            var relatedEntities = await unitOfWork.RelatedEntities.GetListAsync(options: null, cancellationToken: cancellationToken);
            var relatedEntityDropdownItems = relatedEntities.Select(relatedEntity => new EntityNameRelatedEntityDropdownItem
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name,
            });

            var response = new EntityNameDisplayCreateResponse
            {
                Name = string.Empty,
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems]
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayCreateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayCreateError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameCreateResponse> CreateAsync(EntityNameCreateRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Creating);

        if (request == null)
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
        }

        if (string.IsNullOrWhiteSpace(request.Name))
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.NameEmpty);
        }

        var relatedEntityIds = request.RelatedEntityDropdownSelectedIds ?? [];
        if (relatedEntityIds.Any(p => p <= 0))
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.RelatedEntityIdsNotValid);
        }

        var transaction = await unitOfWork.BeginTransactionAsync(cancellationToken);
        try
        {
            var model = unitOfWork.EntityNames.GetNew();
            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = relatedEntityIds };

            unitOfWork.EntityNames.Create(model, entityNameCommandOptions);

             await unitOfWork.SaveChangesAsync(cancellationToken);
             await unitOfWork.CommitTransactionAsync(transaction, cancellationToken);

            var response = new EntityNameCreateResponse { Id = model.Id };

            logger.LogInformation(EntityNameTexts.Messages.Success.CreateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.CreateError}. Ошибка: {exception.Message}");
            await unitOfWork.RollbackTransactionAsync(transaction, cancellationToken);
            throw;
        }
    }

    public async Task<EntityNameDisplayUpdateResponse> DisplayUpdateAsync(EntityNameDisplayUpdateRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayUpdating);

        try
        {
            if (request == null)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
            }

            if (request.Id <= 0)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.IdNotValid);
            }

            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await unitOfWork.EntityNames.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions, cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var relatedEntities = await unitOfWork.RelatedEntities.GetListAsync(options: null, cancellationToken: cancellationToken);
            var relatedEntityDropdownItems = relatedEntities.Select(relatedEntity => new EntityNameRelatedEntityDropdownItem
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name
            });
            var relatedEntityDropdownSelectedIds = model.RelatedEntities.Select(relatedEntity => relatedEntity.Id);

            var response = new EntityNameDisplayUpdateResponse
            {
                Id = model.Id,
                Name = model.Name,
                RelatedEntityDropdownSelectedIds = [.. relatedEntityDropdownSelectedIds],
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems] 
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayUpdateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayUpdateError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameUpdateResponse> UpdateAsync(EntityNameUpdateRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Updating);

        if (request == null)
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
        }

        if (request.Id <= 0)
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.IdNotValid);
        }

        if (string.IsNullOrWhiteSpace(request.Name))
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.NameEmpty);
        }

        var relatedEntityIds = request.RelatedEntityDropdownSelectedIds ?? [];
        if (relatedEntityIds.Any(p => p <= 0))
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.RelatedEntityIdsNotValid);
        }

        var transaction = await unitOfWork.BeginTransactionAsync(cancellationToken);
        try
        {
            var model = await unitOfWork.EntityNames.GetSingleOrDefaultAsync(request.Id, cancellationToken: cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = relatedEntityIds };
            
            var response = unitOfWork.EntityNames.Update(model, entityNameCommandOptions);

            await unitOfWork.SaveChangesAsync(cancellationToken);
            await unitOfWork.CommitTransactionAsync(transaction, cancellationToken);

            logger.LogInformation(EntityNameTexts.Messages.Success.UpdateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.UpdateError}. Ошибка: {exception.Message}");
            await unitOfWork.RollbackTransactionAsync(transaction, cancellationToken);
            throw;
        }
    }

    public async Task<EntityNameDisplayDeleteResponse> DisplayDeleteAsync(EntityNameDisplayDeleteRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayDeleting);

        try
        {
            if (request == null)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
            }

            if (request.Id <= 0)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.IdNotValid);
            }

            var model = await unitOfWork.EntityNames.GetSingleOrDefaultAsync(request.Id, cancellationToken: cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var response = new EntityNameDisplayDeleteResponse
            {
                Id = model.Id,
                Name = model.Name
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayDeleteCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayDeleteError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDeleteResponse> DeleteAsync(EntityNameDeleteRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Deleting);

        if (request == null)
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
        }

        if (request.Id <= 0)
        {
            throw new BadRequestException(EntityNameTexts.Messages.Validation.IdNotValid);
        }

        var transaction = await unitOfWork.BeginTransactionAsync(cancellationToken);
        try
        {
            var model = await unitOfWork.EntityNames.GetSingleOrDefaultAsync(request.Id, cancellationToken: cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            unitOfWork.EntityNames.Delete(model);
            
            await unitOfWork.SaveChangesAsync(cancellationToken);
            await unitOfWork.CommitTransactionAsync(transaction, cancellationToken);
            
            var response = new EntityNameDeleteResponse { IsDeleted = true };

            logger.LogInformation(EntityNameTexts.Messages.Success.DeleteCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DeleteError}. Ошибка: {exception.Message}");
            await unitOfWork.RollbackTransactionAsync(transaction, cancellationToken);
            throw;
        }
    }

    public async Task<EntityNameDisplayInfoResponse> DisplayInfoAsync(EntityNameInfoRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayInfoLoading);

        try
        {
            if (request == null)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
            }

            if (request.Id <= 0)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.IdNotValid);
            }

            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await unitOfWork.EntityNames.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions, cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var relatedEntities = model.RelatedEntities?.Select(relatedEntity => new EntityNameRelatedEntityListModel
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name
            }).ToList() ?? [];

            var response = new EntityNameDisplayInfoResponse
            {
                Id = model.Id,
                Name = model.Name,
                RelatedEntitiesCount = relatedEntities.Count,
                RelatedEntities = relatedEntities,
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayInfoCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayInfoError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDisplayListResponse> DisplayListAsync(EntityNameListRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayListLoading);

        try
        {
            if (request == null)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.RequestNull);
            }

            if (request.PaginationField?.Skip != null && request.PaginationField.Skip < 0)
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.PaginationNotValid);
            }

            if (request.PaginationField?.Take != null && (request.PaginationField.Take < 0 || request.PaginationField.Take > 200))
            {
                throw new BadRequestException(EntityNameTexts.Messages.Validation.PaginationNotValid);
            }

            queryPipelineValidator.Validate(request.SearchFields, request.FilterFields, request.OrderFields);

            var searchParameters = SearchParametersBuilder<EntityName>.Build(request.SearchFields, fieldSelectors);
            var filterParameters = FilterParametersBuilder<EntityName>.Build(request.FilterFields, fieldSelectors);
            var orderParameters = OrderParametersBuilder<EntityName>.Build(request.OrderFields, fieldSelectors);
            var paginationParameters = PaginationParametersBuilder.Build(request.PaginationField);

            var countOptions = new EntityNameQueryOptions
            {
                SearchParameters = searchParameters,
                FilterParameters = filterParameters
            };

            var options = new EntityNameQueryOptions
            {
                IncludeRelatedEntities = true,
                SearchParameters = searchParameters,
                FilterParameters = filterParameters,
                OrderParameters = orderParameters,
                PaginationParameters = paginationParameters
            };

            var total = await unitOfWork.EntityNames.CountAsync(countOptions, cancellationToken);
            var models = await unitOfWork.EntityNames.GetListAsync(options, cancellationToken);

            var entityNames = models.Select(model => new EntityNameListModel
            {
                Id = model.Id,
                Name = model.Name,
                RelatedEntities = model.RelatedEntities?.Select(relatedEntity => new EntityNameRelatedEntityListModel
                {
                    Id = relatedEntity.Id,
                    Name = relatedEntity.Name
                     }).ToList() ?? []
            });

            var response = new EntityNameDisplayListResponse
            {
                EntityNames = [.. entityNames],
                EntityNameCount = total,
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayListCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayListError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDisplayRelatedEntityDropdownResponse> DisplayRelatedEntityDropdownItemsAsync(CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayRelatedEntityDropdownLoading);

        try
        {
            var models = await unitOfWork.RelatedEntities.GetListAsync(options: null, cancellationToken: cancellationToken);
            var relatedEntityDropdownItems = models.Select(model => new EntityNameRelatedEntityDropdownItem
            {
                Id = model.Id,
                Name = model.Name
            });

            var response = new EntityNameDisplayRelatedEntityDropdownResponse
            {
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems]
            };

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayRelatedEntityDropdownCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayRelatedEntityDropdownError}. Ошибка: {exception.Message}");
            throw;
        }
    }
}
```

### 6.6 Добавление регистрации сервисов

**Файл**: `src/ProjectName/Client/Middleware/ServiceRegistration.cs`

Добавить в метод `AddDependencyInjectionExt`:

```csharp
// Регистрация сервисов
services.AddScoped<IServiceEntityName, ServiceEntityName>();
```
