---
name: "Blazor .NET Web Application Development Guide"
description: "A comprehensive development guide for building modern web applications using Blazor, C#, ASP.NET Core, and Entity Framework Core with Visual Studio Enterprise integration"
category: "Full-Stack Framework"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "blazor",
    "dotnet",
    "csharp",
    "aspnet-core",
    "entity-framework",
    "visual-studio",
    "web-development",
  ]
lastUpdated: "2025-06-16"
---

# Blazor .NET Web Application Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing modern web applications using Blazor, C#, ASP.NET Core, and Entity Framework Core. The guide emphasizes efficient development workflows using Visual Studio Enterprise for running, debugging, and testing, while leveraging Cursor AI for code editing and AI-powered suggestions. It focuses on component-based UI development, proper separation of concerns, and following .NET conventions.

## Tech Stack

- **Frontend Framework**: Blazor Server or Blazor WebAssembly
- **Backend Framework**: ASP.NET Core 8.0+
- **Language**: C# 12+ with .NET 8+
- **Database**: Entity Framework Core with SQL Server/PostgreSQL
- **IDE**: Visual Studio Enterprise (for running/debugging/testing)
- **Code Editor**: Cursor AI (for editing and AI suggestions)
- **State Management**: Built-in Blazor state + advanced libraries (Fluxor/BlazorState)
- **Validation**: FluentValidation or DataAnnotations
- **Testing**: xUnit/NUnit/MSTest with Moq/NSubstitute
- **Caching**: IMemoryCache, Redis, or SQL Server Cache

## Development Environment Setup

### Workflow and Development Environment

- **Visual Studio Enterprise**: Used for running, debugging, and testing Blazor applications
- **Cursor AI**: Used for code editing, AI suggestions, and refactoring
- **Integration**: Seamless workflow between both tools for optimal development experience

### Development Requirements

- Visual Studio Enterprise 2022 (17.8+)
- .NET 8.0 SDK or later
- SQL Server Developer Edition or PostgreSQL
- Redis (optional, for distributed caching)
- Node.js 18+ (for client-side assets, if needed)

### Installation Steps

```bash
# 1. Create new Blazor project
dotnet new blazorserver -n MyBlazorApp
# or for WebAssembly
dotnet new blazorwasm -n MyBlazorApp

# 2. Navigate to project directory
cd MyBlazorApp

# 3. Add Entity Framework Core
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design

# 4. Add additional packages
dotnet add package FluentValidation.AspNetCore
dotnet add package Serilog.AspNetCore
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis

# 5. Restore packages
dotnet restore

# 6. Open in Visual Studio Enterprise
start MyBlazorApp.sln
```

## Project Structure

```
BlazorApp/
├── Program.cs                    # Application entry point
├── appsettings.json             # Configuration
├── Components/                  # Blazor components
│   ├── Layout/                 # Layout components
│   │   ├── MainLayout.razor
│   │   ├── NavMenu.razor
│   │   └── MainLayout.razor.css
│   ├── Pages/                  # Page components
│   │   ├── Home.razor
│   │   ├── Counter.razor
│   │   └── Weather.razor
│   └── Shared/                 # Shared components
│       ├── ErrorBoundary.razor
│       └── LoadingSpinner.razor
├── Data/                       # Data layer
│   ├── ApplicationDbContext.cs
│   ├── Models/
│   └── Repositories/
├── Services/                   # Business logic services
│   ├── IUserService.cs
│   └── UserService.cs
├── Models/                     # Domain models
├── ViewModels/                 # View models for complex components
├── Validators/                 # FluentValidation validators
├── wwwroot/                    # Static files
│   ├── css/
│   ├── js/
│   └── images/
└── Tests/                      # Test projects
    ├── UnitTests/
    └── IntegrationTests/
```

## Development Guidelines

### Blazor Code Style and Structure

- Write idiomatic and efficient Blazor and C# code following .NET conventions
- Use Razor Components appropriately for component-based UI development
- Prefer inline functions for smaller components but separate complex logic into code-behind or service classes
- Use async/await for non-blocking UI operations
- Utilize Blazor's built-in features for component lifecycle (OnInitializedAsync, OnParametersSetAsync)
- Use data binding effectively with @bind directive
- Leverage Dependency Injection for services in Blazor
- Structure components and services following Separation of Concerns principle

### Naming Conventions

- **PascalCase**: Component names, method names, public members, interfaces (prefixed with "I")
- **camelCase**: Private fields, local variables, parameters
- **UPPER_CASE**: Constants and configuration keys
- **kebab-case**: CSS classes and HTML attributes

```csharp
// Example: Proper naming conventions
public interface IUserService
{
    Task<User> GetUserByIdAsync(int userId);
}

public class UserService : IUserService
{
    private readonly ApplicationDbContext _context;
    private readonly ILogger<UserService> _logger;

    public UserService(ApplicationDbContext context, ILogger<UserService> logger)
    {
        _context = context;
        _logger = logger;
    }

    public async Task<User> GetUserByIdAsync(int userId)
    {
        return await _context.Users.FindAsync(userId);
    }
}
```

## Core Feature Implementation

### Blazor Component Development

#### Component Lifecycle and Data Binding

```razor
@page "/user-profile/{UserId:int}"
@using MyBlazorApp.Services
@inject IUserService UserService
@inject IJSRuntime JSRuntime

<PageTitle>User Profile</PageTitle>

@if (isLoading)
{
    <LoadingSpinner />
}
else if (user != null)
{
    <div class="user-profile">
        <h2>@user.Name</h2>
        <p>Email: @user.Email</p>

        <EditForm Model="user" OnValidSubmit="HandleValidSubmit">
            <DataAnnotationsValidator />
            <ValidationSummary />

            <div class="form-group">
                <label for="name">Name:</label>
                <InputText id="name" @bind-Value="user.Name" class="form-control" />
                <ValidationMessage For="@(() => user.Name)" />
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <InputText id="email" @bind-Value="user.Email" class="form-control" />
                <ValidationMessage For="@(() => user.Email)" />
            </div>

            <button type="submit" class="btn btn-primary" disabled="@isSubmitting">
                @if (isSubmitting)
                {
                    <span class="spinner-border spinner-border-sm" role="status"></span>
                    <span>Updating...</span>
                }
                else
                {
                    <span>Update Profile</span>
                }
            </button>
        </EditForm>
    </div>
}
else
{
    <div class="alert alert-danger">User not found.</div>
}

@code {
    [Parameter] public int UserId { get; set; }

    private User? user;
    private bool isLoading = true;
    private bool isSubmitting = false;

    protected override async Task OnInitializedAsync()
    {
        await LoadUserAsync();
    }

    protected override async Task OnParametersSetAsync()
    {
        if (UserId != user?.Id)
        {
            await LoadUserAsync();
        }
    }

    private async Task LoadUserAsync()
    {
        try
        {
            isLoading = true;
            user = await UserService.GetUserByIdAsync(UserId);
        }
        catch (Exception ex)
        {
            // Handle error appropriately
            await JSRuntime.InvokeVoidAsync("console.error", $"Error loading user: {ex.Message}");
        }
        finally
        {
            isLoading = false;
        }
    }

    private async Task HandleValidSubmit()
    {
        try
        {
            isSubmitting = true;
            await UserService.UpdateUserAsync(user!);
            await JSRuntime.InvokeVoidAsync("alert", "Profile updated successfully!");
        }
        catch (Exception ex)
        {
            await JSRuntime.InvokeVoidAsync("alert", $"Error updating profile: {ex.Message}");
        }
        finally
        {
            isSubmitting = false;
        }
    }
}
```

#### Component Performance Optimization

```csharp
// Use ShouldRender() to control component re-rendering
public partial class OptimizedComponent : ComponentBase
{
    [Parameter] public string Title { get; set; } = string.Empty;
    [Parameter] public List<Item> Items { get; set; } = new();

    private string previousTitle = string.Empty;
    private int previousItemsCount = 0;

    protected override bool ShouldRender()
    {
        // Only re-render if meaningful changes occurred
        bool shouldRender = Title != previousTitle || Items.Count != previousItemsCount;

        if (shouldRender)
        {
            previousTitle = Title;
            previousItemsCount = Items.Count;
        }

        return shouldRender;
    }

    protected override void OnParametersSet()
    {
        // Minimize work in OnParametersSet
        base.OnParametersSet();
    }
}

// Use EventCallback for efficient event handling
@code {
    [Parameter] public EventCallback<string> OnItemSelected { get; set; }

    private async Task HandleItemClick(string itemId)
    {
        // Pass only minimal data when triggering events
        await OnItemSelected.InvokeAsync(itemId);
    }
}
```

### Entity Framework Core Integration

#### DbContext Configuration

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using MyBlazorApp.Models;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure User entity
        modelBuilder.Entity<User>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Email).IsRequired().HasMaxLength(255);
            entity.HasIndex(e => e.Email).IsUnique();
        });

        // Configure relationships
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasOne(o => o.User)
                  .WithMany(u => u.Orders)
                  .HasForeignKey(o => o.UserId)
                  .OnDelete(DeleteBehavior.Cascade);
        });

        // Seed data
        modelBuilder.Entity<User>().HasData(
            new User { Id = 1, Name = "Admin User", Email = "admin@example.com" }
        );
    }
}
```

#### Repository Pattern Implementation

```csharp
// Data/Repositories/IUserRepository.cs
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
    Task<User?> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetAllAsync();
    Task<User> CreateAsync(User user);
    Task<User> UpdateAsync(User user);
    Task DeleteAsync(int id);
}

// Data/Repositories/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;

    public UserRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<User?> GetByIdAsync(int id)
    {
        return await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
    }

    public async Task<User?> GetByEmailAsync(string email)
    {
        return await _context.Users
            .FirstOrDefaultAsync(u => u.Email == email);
    }

    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Users
            .OrderBy(u => u.Name)
            .ToListAsync();
    }

    public async Task<User> CreateAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
        return user;
    }

    public async Task<User> UpdateAsync(User user)
    {
        _context.Users.Update(user);
        await _context.SaveChangesAsync();
        return user;
    }

    public async Task DeleteAsync(int id)
    {
        var user = await _context.Users.FindAsync(id);
        if (user != null)
        {
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
        }
    }
}
```

## Error Handling and Validation

### FluentValidation Implementation

```csharp
// Validators/UserValidator.cs
using FluentValidation;
using MyBlazorApp.Models;

public class UserValidator : AbstractValidator<User>
{
    public UserValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(2, 100).WithMessage("Name must be between 2 and 100 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(255).WithMessage("Email must not exceed 255 characters");
    }
}

// Register in Program.cs
builder.Services.AddScoped<IValidator<User>, UserValidator>();
```

### Error Boundary Implementation

```razor
@* Components/Shared/ErrorBoundary.razor *@
@inherits ErrorBoundaryBase

<div class="error-boundary">
    @if (CurrentException is not null)
    {
        <div class="alert alert-danger" role="alert">
            <h4 class="alert-heading">Something went wrong!</h4>
            <p>An error occurred while processing your request.</p>
            @if (Environment.IsDevelopment())
            {
                <hr>
                <p class="mb-0">
                    <strong>Exception:</strong> @CurrentException.Message<br>
                    <strong>Stack Trace:</strong><br>
                    <pre>@CurrentException.StackTrace</pre>
                </p>
            }
            <button class="btn btn-primary mt-3" @onclick="Recover">
                Try Again
            </button>
        </div>
    }
    else
    {
        @ChildContent
    }
</div>

@code {
    [Parameter] public RenderFragment? ChildContent { get; set; }

    protected override Task OnErrorAsync(Exception exception)
    {
        // Log the error
        Console.WriteLine($"Error boundary caught exception: {exception}");
        return Task.CompletedTask;
    }
}
```

## State Management

### Built-in Blazor State Management

```csharp
// Services/StateContainer.cs
public class StateContainer
{
    private string? _currentUser;
    private List<Notification> _notifications = new();

    public string? CurrentUser
    {
        get => _currentUser;
        set
        {
            _currentUser = value;
            NotifyStateChanged();
        }
    }

    public IReadOnlyList<Notification> Notifications => _notifications.AsReadOnly();

    public event Action? OnChange;

    public void AddNotification(Notification notification)
    {
        _notifications.Add(notification);
        NotifyStateChanged();
    }

    public void RemoveNotification(int id)
    {
        _notifications.RemoveAll(n => n.Id == id);
        NotifyStateChanged();
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}

// Register in Program.cs
builder.Services.AddScoped<StateContainer>();
```

### Advanced State Management with Fluxor

```csharp
// Install Fluxor package
// dotnet add package Fluxor.Blazor.Web

// State/UserState.cs
using Fluxor;

[FeatureState]
public record UserState
{
    public User? CurrentUser { get; init; }
    public bool IsLoading { get; init; }
    public string? ErrorMessage { get; init; }
}

public class UserStateFeature : Feature<UserState>
{
    public override string GetName() => "User";

    protected override UserState GetInitialState() => new()
    {
        CurrentUser = null,
        IsLoading = false,
        ErrorMessage = null
    };
}

// Actions/UserActions.cs
public record LoadUserAction(int UserId);
public record LoadUserSuccessAction(User User);
public record LoadUserFailureAction(string ErrorMessage);

// Effects/UserEffects.cs
[UsedImplicitly]
public class UserEffects
{
    private readonly IUserService _userService;

    public UserEffects(IUserService userService)
    {
        _userService = userService;
    }

    [EffectMethod]
    public async Task HandleLoadUser(LoadUserAction action, IDispatcher dispatcher)
    {
        try
        {
            var user = await _userService.GetUserByIdAsync(action.UserId);
            dispatcher.Dispatch(new LoadUserSuccessAction(user));
        }
        catch (Exception ex)
        {
            dispatcher.Dispatch(new LoadUserFailureAction(ex.Message));
        }
    }
}

// Reducers/UserReducers.cs
public static class UserReducers
{
    [ReducerMethod]
    public static UserState ReduceLoadUser(UserState state, LoadUserAction action) =>
        state with { IsLoading = true, ErrorMessage = null };

    [ReducerMethod]
    public static UserState ReduceLoadUserSuccess(UserState state, LoadUserSuccessAction action) =>
        state with { CurrentUser = action.User, IsLoading = false, ErrorMessage = null };

    [ReducerMethod]
    public static UserState ReduceLoadUserFailure(UserState state, LoadUserFailureAction action) =>
        state with { IsLoading = false, ErrorMessage = action.ErrorMessage };
}
```

## Caching Strategies

### In-Memory Caching

```csharp
// Services/CachedUserService.cs
public class CachedUserService : IUserService
{
    private readonly IUserService _userService;
    private readonly IMemoryCache _cache;
    private readonly ILogger<CachedUserService> _logger;
    private readonly TimeSpan _cacheExpiration = TimeSpan.FromMinutes(15);

    public CachedUserService(IUserService userService, IMemoryCache cache, ILogger<CachedUserService> logger)
    {
        _userService = userService;
        _cache = cache;
        _logger = logger;
    }

    public async Task<User?> GetUserByIdAsync(int userId)
    {
        string cacheKey = $"user_{userId}";

        if (_cache.TryGetValue(cacheKey, out User? cachedUser))
        {
            _logger.LogInformation("User {UserId} retrieved from cache", userId);
            return cachedUser;
        }

        var user = await _userService.GetUserByIdAsync(userId);
        if (user != null)
        {
            _cache.Set(cacheKey, user, _cacheExpiration);
            _logger.LogInformation("User {UserId} cached for {Expiration}", userId, _cacheExpiration);
        }

        return user;
    }

    public async Task<User> UpdateUserAsync(User user)
    {
        var updatedUser = await _userService.UpdateUserAsync(user);

        // Invalidate cache
        string cacheKey = $"user_{user.Id}";
        _cache.Remove(cacheKey);
        _logger.LogInformation("Cache invalidated for user {UserId}", user.Id);

        return updatedUser;
    }
}
```

### Distributed Caching with Redis

```csharp
// Services/DistributedCacheService.cs
public class DistributedCacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<DistributedCacheService> _logger;

    public DistributedCacheService(IDistributedCache cache, ILogger<DistributedCacheService> logger)
    {
        _cache = cache;
        _logger = logger;
    }

    public async Task<T?> GetAsync<T>(string key) where T : class
    {
        try
        {
            var cachedValue = await _cache.GetStringAsync(key);
            if (cachedValue != null)
            {
                return JsonSerializer.Deserialize<T>(cachedValue);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving from cache with key {Key}", key);
        }

        return null;
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan expiration) where T : class
    {
        try
        {
            var serializedValue = JsonSerializer.Serialize(value);
            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = expiration
            };

            await _cache.SetStringAsync(key, serializedValue, options);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error setting cache with key {Key}", key);
        }
    }

    public async Task RemoveAsync(string key)
    {
        try
        {
            await _cache.RemoveAsync(key);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error removing cache with key {Key}", key);
        }
    }
}
```

## Testing and Debugging

### Unit Testing with xUnit

```csharp
// Tests/UnitTests/UserServiceTests.cs
using Xunit;
using Moq;
using Microsoft.Extensions.Logging;
using MyBlazorApp.Services;
using MyBlazorApp.Data.Repositories;

public class UserServiceTests
{
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly Mock<ILogger<UserService>> _mockLogger;
    private readonly UserService _userService;

    public UserServiceTests()
    {
        _mockRepository = new Mock<IUserRepository>();
        _mockLogger = new Mock<ILogger<UserService>>();
        _userService = new UserService(_mockRepository.Object, _mockLogger.Object);
    }

    [Fact]
    public async Task GetUserByIdAsync_ValidId_ReturnsUser()
    {
        // Arrange
        var userId = 1;
        var expectedUser = new User { Id = userId, Name = "Test User", Email = "test@example.com" };
        _mockRepository.Setup(r => r.GetByIdAsync(userId)).ReturnsAsync(expectedUser);

        // Act
        var result = await _userService.GetUserByIdAsync(userId);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedUser.Id, result.Id);
        Assert.Equal(expectedUser.Name, result.Name);
        Assert.Equal(expectedUser.Email, result.Email);
    }

    [Fact]
    public async Task GetUserByIdAsync_InvalidId_ReturnsNull()
    {
        // Arrange
        var userId = 999;
        _mockRepository.Setup(r => r.GetByIdAsync(userId)).ReturnsAsync((User?)null);

        // Act
        var result = await _userService.GetUserByIdAsync(userId);

        // Assert
        Assert.Null(result);
    }
}
```

### Blazor Component Testing

```csharp
// Tests/UnitTests/ComponentTests/UserProfileTests.cs
using Bunit;
using Microsoft.Extensions.DependencyInjection;
using Moq;
using Xunit;
using MyBlazorApp.Components.Pages;
using MyBlazorApp.Services;

public class UserProfileTests : TestContext
{
    [Fact]
    public void UserProfile_LoadsUserData_DisplaysCorrectly()
    {
        // Arrange
        var mockUserService = new Mock<IUserService>();
        var testUser = new User { Id = 1, Name = "Test User", Email = "test@example.com" };
        mockUserService.Setup(s => s.GetUserByIdAsync(1)).ReturnsAsync(testUser);

        Services.AddSingleton(mockUserService.Object);

        // Act
        var component = RenderComponent<UserProfile>(parameters => parameters
            .Add(p => p.UserId, 1));

        // Assert
        Assert.Contains("Test User", component.Markup);
        Assert.Contains("test@example.com", component.Markup);
    }
}
```

## Security and Authentication

### ASP.NET Identity Integration

```csharp
// Program.cs - Configure Authentication
builder.Services.AddDefaultIdentity<IdentityUser>(options =>
{
    options.SignIn.RequireConfirmedAccount = false;
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;
})
.AddEntityFrameworkStores<ApplicationDbContext>();

// Configure authorization policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("UserOrAdmin", policy => policy.RequireRole("User", "Admin"));
});
```

### JWT Token Authentication

```csharp
// Services/JwtTokenService.cs
public class JwtTokenService
{
    private readonly IConfiguration _configuration;

    public JwtTokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(User user)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_configuration["Jwt:SecretKey"]!);

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
                new Claim(ClaimTypes.Name, user.Name),
                new Claim(ClaimTypes.Email, user.Email)
            }),
            Expires = DateTime.UtcNow.AddHours(24),
            SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
```

## Performance Optimization

### Blazor Server vs WebAssembly Optimization

```csharp
// For Blazor Server - Optimize SignalR connection
// Program.cs
builder.Services.AddSignalR(options =>
{
    options.MaximumReceiveMessageSize = 1024 * 1024; // 1MB
    options.EnableDetailedErrors = builder.Environment.IsDevelopment();
});

// For Blazor WebAssembly - Optimize bundle size
// Program.cs
builder.Services.AddScoped(sp => new HttpClient
{
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
});

// Use lazy loading for large components
@using Microsoft.AspNetCore.Components.Web.Virtualization

<Virtualize Items="@largeDataSet" Context="item">
    <ItemContent>
        <div>@item.Name</div>
    </ItemContent>
    <Placeholder>
        <div>Loading...</div>
    </Placeholder>
</Virtualize>
```

## API Documentation and Swagger

```csharp
// Program.cs - Configure Swagger
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo
        {
            Title = "Blazor API",
            Version = "v1",
            Description = "API for Blazor application"
        });

        // Include XML comments
        var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
        var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
        c.IncludeXmlComments(xmlPath);
    });
}

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Blazor API V1");
    });
}
```

## Best Practices Summary

### Development Workflow

- Use Visual Studio Enterprise for running, debugging, and testing
- Leverage Cursor AI for code editing and AI-powered suggestions
- Follow C# 10+ features like record types, pattern matching, and global usings
- Implement proper error handling for Blazor pages and API calls
- Use logging for error tracking and consider ErrorBoundary for UI-level errors

### Performance Guidelines

- Optimize Razor components by reducing unnecessary renders
- Use StateHasChanged() efficiently and ShouldRender() where appropriate
- Minimize the component render tree by avoiding re-renders unless necessary
- Use EventCallbacks for handling user interactions efficiently
- Implement proper caching strategies based on application requirements

### Security Considerations

- Implement Authentication and Authorization using ASP.NET Identity or JWT tokens
- Use HTTPS for all web communication
- Ensure proper CORS policies are implemented
- Validate all user inputs using FluentValidation or DataAnnotations
- Follow security best practices for sensitive data handling

This comprehensive guide provides a solid foundation for building scalable, maintainable Blazor applications with proper separation of concerns, performance optimization, and security considerations.
