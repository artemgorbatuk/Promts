# Этап 6: Создание сервиса (Services/Api/)

### 6.1 Требования

Требования находятся в правилах Cursor: [.cursor/rules/promt_06_service.rules](../../.cursor/rules/promt_06_service.rules)

### 6.2 Создание моделей запросов и ответов

**Файл**: `src/Blog/Services/Models/EntityName.cs`

```csharp
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
	// Может содержать параметры для фильтрации и пагинации
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

### 6.3 Создание класса с текстами

**Файл**: `src/Blog/Services/Texts/EntityNameTexts.cs`

```csharp
namespace Services.Texts;

public static class EntityNameTexts
{
    public static class Messages
    {
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

**Файл**: `src/Blog/Services/Api/ServiceEntityName.cs`

```csharp
using System.Threading;
using Microsoft.Extensions.Logging;
using Repositories.Ef.Api;
using Repositories.Ef.Options;
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
    private readonly IRepositoryEntityName repositoryEntityName;
    private readonly IRepositoryRelatedEntity repositoryRelatedEntity;
    private readonly ILogger<ServiceEntityName> logger;

    public ServiceEntityName(IRepositoryEntityName repositoryEntityName, IRepositoryRelatedEntity repositoryRelatedEntity, ILogger<ServiceEntityName> logger)
    {
        this.repositoryEntityName = repositoryEntityName;
        this.repositoryRelatedEntity = repositoryRelatedEntity;
        this.logger = logger;
    }

    public async Task<EntityNameDisplayCreateResponse> DisplayCreateAsync(CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayCreating);

        try
        {
            var relatedEntities = await repositoryRelatedEntity.GetListAsync(cancellationToken);
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

        try
        {
            var model = repositoryEntityName.GetNew();
            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = request.RelatedEntityDropdownSelectedIds };

            var response = await repositoryEntityName.CreateAsync(model, entityNameCommandOptions, cancellationToken);

            logger.LogInformation(EntityNameTexts.Messages.Success.CreateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.CreateError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDisplayUpdateResponse> DisplayUpdateAsync(EntityNameDisplayUpdateRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayUpdating);

        try
        {
            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions, cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var relatedEntities = await repositoryRelatedEntity.GetListAsync(cancellationToken);
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

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = request.RelatedEntityDropdownSelectedIds };
            
            var response = await repositoryEntityName.UpdateAsync(model, entityNameCommandOptions, cancellationToken);

            logger.LogInformation(EntityNameTexts.Messages.Success.UpdateCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.UpdateError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDisplayDeleteResponse> DisplayDeleteAsync(EntityNameDisplayDeleteRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayDeleting);

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, cancellationToken);

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

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, cancellationToken);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            await repositoryEntityName.DeleteAsync(model, cancellationToken);
            
            var response = new EntityNameDeleteResponse { IsDeleted = true };

            logger.LogInformation(EntityNameTexts.Messages.Success.DeleteCompleted);

            return response;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DeleteError}. Ошибка: {exception.Message}");
            throw;
        }
    }

    public async Task<EntityNameDisplayInfoResponse> DisplayInfoAsync(EntityNameInfoRequest request, CancellationToken cancellationToken)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayInfoLoading);

        try
        {
            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions, cancellationToken);

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
            var entityNameQueryOptions = new EntityNameQueryOptions { OrderByNameAsc = true, IncludeRelatedEntities = true };
            var models = await repositoryEntityName.GetListAsync(entityNameQueryOptions, cancellationToken);

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
                EntityNameCount = entityNames.Count(),
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
            var relatedEntityQueryOptions = new RelatedEntityQueryOptions { OrderByNameAsc = true };
            var models = await repositoryRelatedEntity.GetListAsync(relatedEntityQueryOptions, cancellationToken);
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

**Файл**: `src/Blog/Client/Middleware/ServiceRegistration.cs`

Добавить в метод `AddDependencyInjectionExt`:

```csharp
// Регистрация сервисов
services.AddScoped<IServiceEntityName, ServiceEntityName>();
```
