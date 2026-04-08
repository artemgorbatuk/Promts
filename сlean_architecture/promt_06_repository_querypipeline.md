# QueryPipeline ядро репозитория (один раз на проект)

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
