# Этап 12: Миграция

### Требования

- Спрашивать разрешение на создание миграции.
- Миграция создается по шаблону **TaskXXX**, где XXX - это порядковый номер (например, Task001, Task002 и т.д.), который можно найти в папке `Datasource.Ef/Migrations/`
- Убедиться, что в папке `Datasource.Ef/Migrations/` нет файлов с тем же номером

### Подготовка

**Переход в рабочую директорию (выполнить один раз):**
```bash
cd src/ProjectName
```

### Проверка перед созданием миграции

**Команда для просмотра всех существующих миграций (выполнить ПЕРЕД созданием новой):**
```bash
dotnet ef migrations list --project Datasource.Ef --startup-project Client
```

### Создание и применение миграции

**Команды для выполнения:**
```bash
dotnet ef migrations add TaskXXX --project Datasource.Ef --startup-project Client
dotnet ef database update --project Datasource.Ef --startup-project Client
```

### Проверка после создания миграции

**Команда для проверки что будет применено (выполнить ПОСЛЕ создания, но ПЕРЕД применением):**
```bash
dotnet ef database update --project Datasource.Ef --startup-project Client --dry-run
```

**Команда для проверки статуса миграций после применения:**
```bash
dotnet ef migrations list --project Datasource.Ef --startup-project Client
```
