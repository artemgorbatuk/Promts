# Этап 6: Создание сервиса (Services/Api/)

### 6.1 Требования

Требования находятся в правилах Cursor: [.cursor/rules/promt_06_service.rules](../../.cursor/rules/promt_06_service.rules)

### 6.2 Создание моделей запросов и ответов

**Файл**: `src/Blog/Services/Models/EntityName.cs`

```csharp
using Services.Models.Shared;

namespace Services.Models;

// === CREATE ===
public class EntityNameCreateRequest
{
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<int> RelatedEntityDropdownSelectedIds { get; set; } = [];
}

public class EntityNameCreatePageResponse
{
	public required string? Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required ICollection<EntityNameRelatedEntityDropdownItem> RelatedEntityDropdownItems { get; set; } = [];
}

// === UPDATE ===
public class EntityNameUpdatePageRequest
{
	public required int Id { get; set; }
}

public class EntityNameUpdatePageResponse
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

// === DELETE ===
public class EntityNameDeletePageRequest
{
	public required int Id { get; set; }
}

public class EntityNameDeletePageResponse
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
public class EntityNameInfoPageRequest
{
	public required int Id { get; set; }
}

public class EntityNameInfoPageResponse
{
	public required int Id { get; set; }
	public required string Name { get; set; }
	// Примечание: если модель из `Datasource.Ef/Models` содержит дополнительные поля, их необходимо добавить в эту модель.
	public required int RelatedEntitiesCount { get; set; }
	public required ICollection<EntityNameRelatedEntityListModel> RelatedEntities { get; set; } = [];
}

// === LIST ===
public class EntityNameListPageRequest
{
	// Может содержать параметры для фильтрации и пагинации
}

public class EntityNameListPageResponse
{
	public required ICollection<EntityNameListModel> EntityNames { get; set; } = [];
  public required bool IsVisible { get; set; }
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

public class EntityNameRelatedEntityDropdownResponse
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
            public const string DisplayCreating = "Начало отображения страницы создания EntityName";
            public const string DisplayUpdating = "Начало отображения страницы обновления EntityName";
            public const string DisplayDeleting = "Начало отображения страницы удаления EntityName";

            public const string Creating = "Начало выполнения создания EntityName";
            public const string Updating = "Начало выполнения обновления EntityName";
            public const string Deleting = "Начало выполнения удаления EntityName";

            public const string DisplayInfoLoading = "Начало отображения страницы просмотра EntityName";
            public const string DisplayListLoading = "Начало отображения страницы списка EntityNames";
            public const string DisplayRelatedEntityDropdownLoading = "Начало отображения выпадающего списка связанных сущностей";

        }

        public static class Success
        {
            public const string DisplayCreateCompleted = "Страница создания EntityName успешно отображена";
            public const string DisplayUpdateCompleted = "Страница обновления EntityName успешно отображена";
            public const string DisplayDeleteCompleted = "Страница удаления EntityName успешно отображена";

            public const string CreateCompleted = "EntityName успешно создан";
            public const string UpdateCompleted = "EntityName успешно обновлен";
            public const string DeleteCompleted = "EntityName успешно удален";

            public const string DisplayInfoCompleted = "Страница просмотра EntityName успешно отображена";
            public const string DisplayListCompleted = "Страница списка EntityNames успешно отображена";
            public const string DisplayRelatedEntityDropdownCompleted = "Выпадающий список связанных сущностей успешно отображён";
        }

        public static class Error
        {
            public const string DisplayCreateError = "Не удалось отобразить страницу создания EntityName";
            public const string DisplayUpdateError = "Не удалось отобразить страницу обновления EntityName";
            public const string DisplayDeleteError = "Не удалось отобразить страницу удаления EntityName";

            public const string CreateError = "Не удалось создать EntityName";
            public const string UpdateError = "Не удалось обновить EntityName";
            public const string DeleteError = "Не удалось удалить EntityName";

            public const string DisplayInfoError = "Не удалось отобразить страницу просмотра EntityName";
            public const string DisplayListError = "Не удалось отобразить страницу списка EntityNames";
            public const string DisplayRelatedEntityDropdownError = "Не удалось отобразить выпадающий список связанных сущностей";

            public const string NotFoundById = "EntityName не был найден по id";
        }
    }
}
```

### 6.4 Создание интерфейса сервиса

**Файл**: `src/Blog/Services/Api/ServiceEntityName.cs`

```csharp
using Microsoft.Extensions.Logging;
using Repositories.Ef.Api;
using Repositories.Ef.Options;
using Services.Texts;
using Services.Enums;
using Services.Exceptions;
using Services.Models;
using Services.Models.Shared;

namespace Services.Api;

public interface IServiceEntityName
{
    Task<ResponseInfo<EntityNameCreatePageResponse>> DisplayCreatePageAsync();
    Task<ResponseInfo<EntityNameCreatePageResponse>> CreateAsync(EntityNameCreateRequest request);

    Task<ResponseInfo<EntityNameUpdatePageResponse>> DisplayUpdatePageAsync(EntityNameUpdatePageRequest request);
    Task<ResponseInfo<EntityNameUpdatePageResponse>> UpdateAsync(EntityNameUpdateRequest request);

    Task<ResponseInfo<EntityNameDeletePageResponse>> DisplayDeletePageAsync(EntityNameDeletePageRequest request);
    Task<ResponseInfo<EntityNameDeleteResponse>> DeleteAsync(EntityNameDeleteRequest request);
    
    Task<ResponseInfo<EntityNameInfoPageResponse>> DisplayInfoPageAsync(EntityNameInfoPageRequest request);
    Task<ResponseInfo<EntityNameListPageResponse>> DisplayListPageAsync(EntityNameListPageRequest request);
    Task<ResponseInfo<EntityNameRelatedEntityDropdownResponse>> DisplayRelatedEntityDropdownItemsAsync();
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

    public async Task<ResponseInfo<EntityNameCreatePageResponse>> DisplayCreatePageAsync()
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayCreating);

        try
        {
            var relatedEntities = await repositoryRelatedEntity.GetListAsync();
            var relatedEntityDropdownItems = relatedEntities.Select(relatedEntity => new EntityNameRelatedEntityDropdownItem
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name,
            });

            var response = new EntityNameCreatePageResponse
            {
                Name = string.Empty,
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems]
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayCreateCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayCreateCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayCreateError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameCreatePageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayCreateError);
        }
    }

    public async Task<ResponseInfo<EntityNameCreatePageResponse>> CreateAsync(EntityNameCreateRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Creating);

        try
        {
            var model = repositoryEntityName.GetNew();
            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = request.RelatedEntityIds };
            await repositoryEntityName.CreateAsync(model, entityNameCommandOptions);

            var responseInfo = ResponseInfo.Success<EntityNameCreatePageResponse>(
                MessageType.SAVED,
                EntityNameTexts.Messages.Success.CreateCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.CreateCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.CreateError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameCreatePageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.CreateError);
        }
    }

    public async Task<ResponseInfo<EntityNameUpdatePageResponse>> DisplayUpdatePageAsync(EntityNameUpdatePageRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayUpdating);

        try
        {
            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var relatedEntities = await repositoryRelatedEntity.GetListAsync();
            var relatedEntityDropdownItems = relatedEntities.Select(relatedEntity => new EntityNameRelatedEntityDropdownItem
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name
            });
            var relatedEntityDropdownSelectedIds = model.RelatedEntities.Select(relatedEntity => relatedEntity.Id);

            var response = new EntityNameUpdatePageResponse
            {
                Id = model.Id,
                Name = model.Name,
                RelatedEntityIds = [.. relatedEntityDropdownSelectedIds],
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems] 
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayUpdateCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayUpdateCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayUpdateError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameUpdatePageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayUpdateError);
        }
    }

    public async Task<ResponseInfo<EntityNameUpdatePageResponse>> UpdateAsync(EntityNameUpdateRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Updating);

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            model.Name = request.Name;

            var entityNameCommandOptions = new EntityNameCommandOptions { RelatedEntityIds = request.RelatedEntityIds };
            await repositoryEntityName.UpdateAsync(model, entityNameCommandOptions);

            var responseInfo = ResponseInfo.Success<EntityNameUpdatePageResponse>(
                MessageType.SAVED,
                EntityNameTexts.Messages.Success.UpdateCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.UpdateCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.UpdateError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameUpdatePageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.UpdateError);
        }
    }

    public async Task<ResponseInfo<EntityNameDeletePageResponse>> DisplayDeletePageAsync(EntityNameDeletePageRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayDeleting);

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var response = new EntityNameDeletePageResponse
            {
                Id = model.Id,
                Name = model.Name
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayDeleteCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayDeleteCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayDeleteError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameDeletePageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayDeleteError);
        }
    }

    public async Task<ResponseInfo<EntityNameDeleteResponse>> DeleteAsync(EntityNameDeleteRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.Deleting);

        try
        {
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            await repositoryEntityName.DeleteAsync(model);

            var responseInfo = ResponseInfo.Success<EntityNameDeleteResponse>(
                MessageType.SAVED,
                EntityNameTexts.Messages.Success.DeleteCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DeleteCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DeleteError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameDeleteResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DeleteError);
        }
    }

    public async Task<ResponseInfo<EntityNameInfoPageResponse>> DisplayInfoPageAsync(EntityNameInfoPageRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayInfoLoading);

        try
        {
            var entityNameQueryOptions = new EntityNameQueryOptions { IncludeRelatedEntities = true };
            var model = await repositoryEntityName.GetSingleOrDefaultAsync(request.Id, entityNameQueryOptions);

            if (model == default)
                throw new NotFoundException($"{EntityNameTexts.Messages.Error.NotFoundById} : {request.Id}");

            var relatedEntities = model.RelatedEntities?.Select(relatedEntity => new EntityNameRelatedEntityListModel
            {
                Id = relatedEntity.Id,
                Name = relatedEntity.Name
            }).ToList() ?? [];

            var response = new EntityNameInfoPageResponse
            {
                Id = model.Id,
                Name = model.Name,
                RelatedEntitiesCount = relatedEntities.Count,
                RelatedEntities = relatedEntities,
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayInfoCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayInfoCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayInfoError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameInfoPageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayInfoError);
        }
    }

    public async Task<ResponseInfo<EntityNameListPageResponse>> DisplayListPageAsync(EntityNameListPageRequest request)
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayListLoading);

        try
        {
            var entityNameQueryOptions = new EntityNameQueryOptions { OrderByNameAsc = true, IncludeRelatedEntities = true };
            var models = await repositoryEntityName.GetListAsync(entityNameQueryOptions);

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

            var response = new EntityNameListPageResponse
            {
                EntityNames = [.. entityNames],
                EntityNameCount = entityNames.Count,
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayListCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayListCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayListError}. Ошибка: {exception.Message}");

            return ResponseInfo.Error<EntityNameListPageResponse>(
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayListError);
        }
    }

    public async Task<ResponseInfo<EntityNameRelatedEntityDropdownResponse>> DisplayRelatedEntityDropdownItemsAsync()
    {
        logger.LogInformation(EntityNameTexts.Messages.Start.DisplayRelatedEntityDropdownLoading);

        try
        {
            var relatedEntityQueryOptions = new RelatedEntityQueryOptions { OrderByNameAsc = true };
            var models = await repositoryRelatedEntity.GetListAsync(relatedEntityQueryOptions);
            var relatedEntityDropdownItems = models.Select(model => new EntityNameRelatedEntityDropdownItem
            {
                Id = model.Id,
                Name = model.Name
            });

            var response = new EntityNameRelatedEntityDropdownResponse
            {
                RelatedEntityDropdownItems = [.. relatedEntityDropdownItems]
            };

            var responseInfo = ResponseInfo.Success(
                response,
                MessageType.LOADED,
                EntityNameTexts.Messages.Success.DisplayRelatedEntityDropdownCompleted);

            logger.LogInformation(EntityNameTexts.Messages.Success.DisplayRelatedEntityDropdownCompleted);

            return responseInfo;
        }
        catch (Exception exception)
        {
            logger.LogError(exception, $"{EntityNameTexts.Messages.Error.DisplayRelatedEntityDropdownError}. Ошибка: {exception.Message}");

            var response = new EntityNameRelatedEntityDropdownResponse
            {
                RelatedEntityDropdownItems = []
            };

            return ResponseInfo.Error(
                response,
                MessageType.ERROR,
                EntityNameTexts.Messages.Error.DisplayRelatedEntityDropdownError);
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
