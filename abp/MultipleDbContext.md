# How to Use Multiple DbContexts in a Single Query Execution

## Introduction
It's common to have multiple DbContexts in a single application, especially when your application is modular. However, querying from different repositories, each associated with a different DbContext, can lead to this error:

```csharp
System.InvalidOperationException: 'Cannot use multiple context instances within a single query execution. Ensure the query uses a single context instance.'
```

This error occurs because EF Core does not support querying from multiple DbContexts in a single query execution.

In this article, we will explore a solution to query from multiple DbContext instances within a single execution in the ABP Framework with EF Core. Specifically, we will focus on a scenario where we have two DbContext instances: one for an ExamsModule and another for the main application (LMSApp). Notably, both DbContext instances in our example connect to the same database.

Additionally, it’s important to note that this workaround is not the only solution. Another approach involves replacing the DbContext and injecting it directly where needed in the host application. This approach is detailed in the [ABP Framework documentation](https://docs.abp.io/en/abp/latest/Entity-Framework-Core-Migrations#entityframeworkcore-project).

## Real-World Example
Let's say we have two DbContexts: one for the `ExamsModule` and another for the host application `LMSApp`. We want to get the `Course` from the LMSApp DbContext and `Exam` details from the ExamsModule DbContext in a single query execution.

### LMS Application
```csharp
public class Course<Guid>
{
    public string Title { get; set; }
    public string Description { get; set; }
}

public interface ILMSDbContext : IEfCoreDbContext
{
    DbSet<Course> Courses { get; set; }
}

public class LMSDbContext : AbpDbContext<LMSDbContext>, ILMSDbContext
{
    public DbSet<Course> Courses { get; set; }
}

public interface ICourseRepository : IRepository<Course, Guid>
{
    // Add custom methods here
}

public class CourseRepository : EfCoreRepository<LMSDbContext, Course, Guid>, ICourseRepository
{
    public CourseRepository(IDbContextProvider<LMSDbContext> dbContextProvider) : base(dbContextProvider)
    {
    }

    // Implement custom methods here
}
```

### Exams Module
```csharp
public class Exam<Guid>
{
    public string Title { get; set; }
    public DateTime Date { get; set; }
}

public interface IExamDbContext : IEfCoreDbContext
{
    DbSet<Exam> Exams { get; set; }
}

public class ExamDbContext : AbpDbContext<ExamDbContext>, IExamDbContext
{
    public DbSet<Exam> Exams { get; set; }
}

public interface IExamRepository : IRepository<Exam, Guid>
{
    // Add custom methods here
}

public class ExamRepository : EfCoreRepository<ExamDbContext, Exam, Guid>, IExamRepository
{
    public ExamRepository(IDbContextProvider<ExamDbContext> dbContextProvider) : base(dbContextProvider)
    {
    }

    // Implement custom methods here
}
```

### Querying from Multiple DbContexts in an Application Service in the Host Application
```csharp
public class CourseExamAppService : ApplicationService
{
    private readonly ICourseRepository _courseRepository;
    private readonly IExamRepository _examRepository;

    public CourseExamAppService(ICourseRepository courseRepository, IExamRepository examRepository)
    {
        _courseRepository = courseRepository;
        _examRepository = examRepository;
    }

    public async Task<CourseExamDto> GetCourseExamAsync(Guid courseId, Guid examId)
    {
        var course = await _courseRepository.GetAsync(courseId);
        var exam = await _examRepository.GetAsync(examId);

        return new CourseExamDto
        {
            CourseTitle = course.Title,
            CourseDescription = course.Description,
            ExamTitle = exam.Title,
            ExamDate = exam.Date
        };
    }
}
```

In the above code, we are querying from two different repositories, each in a different DbContext. This will throw the error mentioned earlier.

## Solution

To solve this issue, we need to abstract the exams repository and implement it using the LMSApp DbContext. This ensures that the two repositories we use will be in the same DbContext (LMSApp DbContext).

### Updated Exam Repository
```csharp
// Abstract Exam Repository to use in the host application
public class AbstractExamRepository<TDbContext> : EfCoreRepository<TDbContext, Exam, Guid>, IExamRepository
    where TDbContext : IExamDbContext
{
    public AbstractExamRepository(IDbContextProvider<TDbContext> dbContextProvider) : base(dbContextProvider)
    {
    }

    // Implement custom methods here
}

// Concrete Exam Repository to use in the ExamsModule
public class ExamRepository : AbstractExamRepository<ExamDbContext>
{
    public ExamRepository(IDbContextProvider<ExamDbContext> dbContextProvider) : base(dbContextProvider)
    {
    }
}
```

### AppDbContext
```csharp
public class LMSDbContext : AbpDbContext<LMSDbContext>, ILMSDbContext, IExamDbContext
{
    public DbSet<Course> Courses { get; set; }
    public DbSet<Exam> Exams { get; set; }
}

public interface ILMSExamRepository : IExamRepository
{
}

public class LMSExamRepository : AbstractExamRepository<LMSDbContext>, ILMSExamRepository
{
    public LMSExamRepository(IDbContextProvider<LMSDbContext> dbContextProvider) : base(dbContextProvider)
    {
    }
}
```

Then you need to add the new repository to the dependency injection container in the host application.

```csharp
public override void ConfigureServices(ServiceConfigurationContext context)
{
    // context.Services.AddScoped<ILMSExamRepository, LMSExamRepository>();
    context.Services.AddAbpDbContext<LMSDbContext>(options =>
    {
        options.AddDefaultRepositories(includeAllEntities: true);
        options.AddRepository<Exam, LMSExamRepository>();
    });
}
```

Now, you can use the `ILMSExamRepository` in the `CourseExamAppService` to query from both repositories in a single query execution.

```csharp
public class CourseExamAppService : ApplicationService
{
    private readonly ICourseRepository _courseRepository;
    private readonly ILMSExamRepository _examRepository;

    public CourseExamAppService(ICourseRepository courseRepository, ILMSExamRepository examRepository)
    {
        _courseRepository = courseRepository;
        _examRepository = examRepository;
    }

    public async Task<CourseExamDto> GetCourseExamAsync(Guid courseId, Guid examId)
    {
        var course = await _courseRepository.GetAsync(courseId);
        var exam = await _examRepository.GetAsync(examId);

        return new CourseExamDto
        {
            CourseTitle = course.Title,
            CourseDescription = course.Description,
            ExamTitle = exam.Title,
            ExamDate = exam.Date
        };
    }
}
```

## Conclusion
In this article, we saw how to query from multiple DbContexts in a single query execution in the ABP Framework. We abstracted the repository and implemented it using the host application DbContext, allowing us to query from multiple repositories in a single query execution.

This article was written by [Ahmad Nidal](https://community.abp.io/members/ahmadnidal.dr@gmail.com) with team instructions, especially from [Qais Al-Khateeb](https://community.abp.io/members/qais.alkhateeb@devnas-jo.com) and [Suhaib Musa](https://community.abp.io/members/suhaibmousa032@gmail.com).
