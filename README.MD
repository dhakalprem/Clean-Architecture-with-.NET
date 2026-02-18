Clean Architecture with .NET (C#), ASP.NET Core and Entity Framework

Databases: PostgreSQL, MySQL, SQL Server, Oracle

Clean Architecture helps you build maintainable, testable, and scalable enterprise applications by separating concerns and enforcing dependency rules.

1️⃣ What is Clean Architecture?

Clean Architecture (popularized by Robert C. Martin) organizes code into layers where:

Core business logic is independent

Frameworks & databases are external

Dependencies always point inward

2️⃣ Layered Structure in .NET

Typical project structure in ASP.NET Core:

Solution
 ├── Domain
 ├── Application
 ├── Infrastructure
 └── WebAPI (Presentation)

🟢 1. Domain Layer (Core)

Entities

Value Objects

Domain Events

Interfaces (Repository contracts)

⚡ No dependency on ASP.NET Core, EF, or database.

Example:

public class Order
{
    public Guid Id { get; private set; }
    public decimal TotalAmount { get; private set; }

    public void UpdateAmount(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");

        TotalAmount = amount;
    }
}

🔵 2. Application Layer

Use Cases

DTOs

Interfaces (Repository, Unit of Work)

Business Rules

Example:

public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task AddAsync(Order order);
}

🟡 3. Infrastructure Layer

Implements:

Entity Framework Core

Database providers

External services

Uses:

Entity Framework Core

Example:

public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Order> GetByIdAsync(Guid id)
        => await _context.Orders.FindAsync(id);

    public async Task AddAsync(Order order)
        => await _context.Orders.AddAsync(order);
}

🔴 4. Presentation Layer (WebAPI)

Uses:

ASP.NET Core

Example Controller:

[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    private readonly IOrderRepository _repository;

    public OrdersController(IOrderRepository repository)
    {
        _repository = repository;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(Guid id)
    {
        var order = await _repository.GetByIdAsync(id);
        return Ok(order);
    }
}

3️⃣ Entity Framework Core (ORM)

Entity Framework Core supports multiple database providers:

Database	EF Core Provider
PostgreSQL	Npgsql
MySQL	Pomelo / Oracle provider
Microsoft SQL Server	Microsoft Provider
Oracle Database	Oracle EF Core provider
4️⃣ Database Configuration Example
SQL Server
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(configuration.GetConnectionString("Default")));

PostgreSQL
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(configuration.GetConnectionString("Default")));

MySQL
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));

5️⃣ Dependency Rule (Most Important)
WebAPI  →  Application  →  Domain
Infrastructure → Application & Domain


✔ Domain never depends on EF
✔ Application never depends on ASP.NET
✔ Infrastructure depends on everything
✔ Presentation depends only on Application

6️⃣ Benefits

✅ Testable (mock repositories)

✅ Database-agnostic

✅ Scalable for microservices

✅ Clear separation of concerns

✅ Easy to swap databases

7️⃣ Advanced Best Practices

Use CQRS pattern

Use MediatR for request handling

Apply FluentValidation

Use Unit of Work

Implement Repository Pattern carefully (avoid over-abstraction)

Use Migrations per environment

Use Docker for database isolation

8️⃣ When Using Different Databases
PostgreSQL

Strong JSON support

Great for analytics workloads

MySQL

Lightweight & widely used

SQL Server

Best integration with .NET ecosystem

Oracle

Enterprise-grade systems

9️⃣ Typical Enterprise Architecture Stack

ASP.NET Core Web API

EF Core

Clean Architecture

PostgreSQL / SQL Server

Docker

Azure / AWS hosting

CI/CD (GitHub Actions / Azure DevOps)