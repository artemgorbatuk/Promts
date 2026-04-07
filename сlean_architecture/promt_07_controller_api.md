# Этап 7: Контроллер Web API (ASP.NET Core, latest .NET)

### Требования

- Цель: реализовать HTTP API для сущности `EntityName` поверх сервисного слоя `IServiceEntityName`
- Использовать ASP.NET Core Web API (Controllers) на актуальной версии .NET и C# (net8.0+), `nullable` включен
- Контроллер размещать в слое представления (например, `src/Blog/Client/Controllers/`)
- Контроллер должен быть помечен `[ApiController]`
- Роуты должны быть стабильными и человекочитаемыми, в kebab-case: базовый префикс `api/entity-names`
- Все операции должны принимать `CancellationToken cancellationToken`
- Все операции должны вызывать только сервисы и не обращаться к `DbContext`/репозиториям напрямую
- Порядок и группировка action-методов в контроллере должны соответствовать интерфейсу сервиса: DisplayCreate/Create, DisplayUpdate/Update, DisplayDelete/Delete, DisplayInfo, DisplayList, DisplayRelatedEntityDropdownItems
- Не логировать входные модели целиком и не писать в логи чувствительные данные
- Для стандартных CRUD-операций использовать HTTP-методы и коды ответов:
  - `GET` возвращает `200 OK`
  - `POST` возвращает `200 OK`; `201 Created` использовать только если контракт создания возвращает идентификатор созданной сущности и контроллер может вернуть корректный Location
  - `PUT` возвращает `200 OK`
  - `DELETE` возвращает `200 OK`
- Для ошибок домена использовать единый формат ошибок (ProblemDetails) через глобальный обработчик исключений либо через middleware проекта
- Если в проекте используется авторизация, контроллер должен быть защищен `[Authorize]`; публичные методы допускаются только при явном требовании
- Для OpenAPI/Swagger описывать ответы прагматично: на action указывать успешные коды (200/201), а типовые ошибки (401/403/500 + ProblemDetails) задавать на уровне контроллера или через Swagger-конвенции/фильтры

### Практика OpenAPI/Swagger

- Если в проекте используется `[Authorize]`, коды `401/403` можно указать один раз на контроллере, а не на каждом action
- Для единого формата ошибок использовать `ProblemDetails` и указать это через `ProducesErrorResponseType(typeof(ProblemDetails))`
- `404 NotFound` указывать на action только если:
  - action явно возвращает `NotFound()`, или
  - проект маппит `NotFoundException` в `404` глобально, и это важно отразить в контракте

### Создание контроллера

**Файл**: `src/Blog/Client/Controllers/EntityNamesController.cs`

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Services.Api;
using Services.Models;

namespace Client.Controllers;

[ApiController]
[Route("api/entity-names")]
[Authorize]
[Produces("application/json")]
[ProducesResponseType(StatusCodes.Status401Unauthorized)]
[ProducesResponseType(StatusCodes.Status403Forbidden)]
[ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
[ProducesErrorResponseType(typeof(ProblemDetails))]
public class EntityNamesController : ControllerBase
{
    private readonly IServiceEntityName service;

    public EntityNamesController(IServiceEntityName service)
    {
        this.service = service;
    }

    [HttpGet("create")]
    [ProducesResponseType(typeof(EntityNameDisplayCreateResponse), StatusCodes.Status200OK)]
    public async Task<ActionResult<EntityNameDisplayCreateResponse>> DisplayCreateAsync(CancellationToken cancellationToken)
    {
        var response = await service.DisplayCreateAsync(cancellationToken);
        return Ok(response);
    }

    [HttpPost]
    [ProducesResponseType(typeof(EntityNameCreateResponse), StatusCodes.Status200OK)]
    public async Task<ActionResult<EntityNameCreateResponse>> CreateAsync([FromBody] EntityNameCreateRequest request, CancellationToken cancellationToken)
    {
        var response = await service.CreateAsync(request, cancellationToken);
        return Ok(response);
    }

    [HttpGet("{id:int}/edit")]
    [ProducesResponseType(typeof(EntityNameDisplayUpdateResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<EntityNameDisplayUpdateResponse>> DisplayUpdateAsync([FromRoute] int id, CancellationToken cancellationToken)
    {
        var response = await service.DisplayUpdateAsync(new EntityNameDisplayUpdateRequest { Id = id }, cancellationToken);
        return Ok(response);
    }

    [HttpPut]
    [ProducesResponseType(typeof(EntityNameUpdateResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<EntityNameUpdateResponse>> UpdateAsync([FromBody] EntityNameUpdateRequest request, CancellationToken cancellationToken)
    {
        var response = await service.UpdateAsync(request, cancellationToken);
        return Ok(response);
    }

    [HttpGet("{id:int}/delete")]
    [ProducesResponseType(typeof(EntityNameDisplayDeleteResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<EntityNameDisplayDeleteResponse>> DisplayDeleteAsync([FromRoute] int id, CancellationToken cancellationToken)
    {
        var response = await service.DisplayDeleteAsync(new EntityNameDisplayDeleteRequest { Id = id }, cancellationToken);
        return Ok(response);
    }

    [HttpDelete("{id:int}")]
    [ProducesResponseType(typeof(EntityNameDeleteResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<EntityNameDeleteResponse>> DeleteAsync([FromRoute] int id, CancellationToken cancellationToken)
    {
        var response = await service.DeleteAsync(new EntityNameDeleteRequest { Id = id }, cancellationToken);
        return Ok(response);
    }

    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(EntityNameDisplayInfoResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<EntityNameDisplayInfoResponse>> DisplayInfoAsync([FromRoute] int id, CancellationToken cancellationToken)
    {
        var response = await service.DisplayInfoAsync(new EntityNameInfoRequest { Id = id }, cancellationToken);
        return Ok(response);
    }

    [HttpGet]
    [ProducesResponseType(typeof(EntityNameDisplayListResponse), StatusCodes.Status200OK)]
    public async Task<ActionResult<EntityNameDisplayListResponse>> DisplayListAsync([FromQuery] EntityNameListRequest request, CancellationToken cancellationToken)
    {
        var response = await service.DisplayListAsync(request, cancellationToken);
        return Ok(response);
    }

    [HttpGet("related-entities/dropdown")]
    [ProducesResponseType(typeof(EntityNameDisplayRelatedEntityDropdownResponse), StatusCodes.Status200OK)]
    public async Task<ActionResult<EntityNameDisplayRelatedEntityDropdownResponse>> DisplayRelatedEntityDropdownItemsAsync(CancellationToken cancellationToken)
    {
        var response = await service.DisplayRelatedEntityDropdownItemsAsync(cancellationToken);
        return Ok(response);
    }
}
```

### Подключение контроллеров в Program.cs

Убедиться, что в `Program.cs` зарегистрированы контроллеры и маршрутизация:

```csharp
builder.Services.AddControllers();
app.MapControllers();
```
