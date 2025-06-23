---
name: "Go Microservices Backend Development Guide"
description: "A comprehensive development guide for building scalable microservices using Go with Clean Architecture, OpenTelemetry observability, and modern backend development practices"
category: "Backend Service"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "go",
    "microservices",
    "backend",
    "clean-architecture",
    "opentelemetry",
    "grpc",
    "rest-api",
    "observability",
  ]
lastUpdated: "2025-06-16"
---

# Go Microservices Backend Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing scalable microservices using Go with Clean Architecture principles, OpenTelemetry observability, and modern backend development practices. The guide emphasizes idiomatic Go code, modular design, comprehensive testing, and robust observability across distributed systems.

## Tech Stack

- **Language**: Go 1.21+
- **Architecture**: Clean Architecture with Domain-Driven Design
- **API**: gRPC, REST (Gin/Echo), GraphQL
- **Database**: PostgreSQL, MongoDB, Redis
- **Message Queue**: RabbitMQ, Apache Kafka, NATS
- **Observability**: OpenTelemetry, Jaeger, Prometheus, Grafana
- **Testing**: Testify, GoMock, Ginkgo
- **Security**: JWT, OAuth2, TLS
- **Deployment**: Docker, Kubernetes, Helm

## Development Environment Setup

### Installation Requirements

- Go 1.21+
- Docker & Docker Compose
- Protocol Buffers compiler (protoc)
- golangci-lint for code quality
- OpenTelemetry Collector

### Installation Steps

```bash
# Install Go dependencies
go mod init myservice

# Core dependencies
go get github.com/gin-gonic/gin
go get google.golang.org/grpc
go get google.golang.org/protobuf
go get github.com/lib/pq
go get github.com/redis/go-redis/v9

# OpenTelemetry
go get go.opentelemetry.io/otel
go get go.opentelemetry.io/otel/trace
go get go.opentelemetry.io/otel/metric
go get go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp
go get go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc

# Testing
go get github.com/stretchr/testify
go get github.com/golang/mock/gomock
go get go.uber.org/mock/mockgen

# Development tools
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## Project Structure

```
microservice/
├── cmd/                          # Application entrypoints
│   ├── server/
│   │   └── main.go
│   └── migrate/
│       └── main.go
├── internal/                     # Core application logic
│   ├── domain/                   # Domain models and interfaces
│   │   ├── user.go
│   │   ├── repository.go
│   │   └── service.go
│   ├── usecase/                  # Business logic/use cases
│   │   ├── user_usecase.go
│   │   └── interfaces.go
│   ├── repository/               # Data access layer
│   │   ├── postgres/
│   │   ├── redis/
│   │   └── interfaces.go
│   ├── delivery/                 # Transport layer
│   │   ├── http/
│   │   ├── grpc/
│   │   └── middleware/
│   └── config/                   # Configuration
│       └── config.go
├── pkg/                          # Shared utilities
│   ├── logger/
│   ├── database/
│   ├── telemetry/
│   └── errors/
├── api/                          # API definitions
│   ├── proto/
│   ├── openapi/
│   └── generated/
├── configs/                      # Configuration files
│   ├── config.yaml
│   └── docker-compose.yml
├── test/                         # Test utilities and integration tests
│   ├── mocks/
│   ├── fixtures/
│   └── integration/
├── deployments/                  # Deployment configurations
│   ├── docker/
│   └── k8s/
├── docs/                         # Documentation
│   ├── ARCHITECTURE.md
│   └── API.md
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

## Core Architecture Principles

### Clean Architecture Implementation

```go
// internal/domain/user.go - Domain models and interfaces
package domain

import (
    "context"
    "time"
)

// User represents the core business entity
type User struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

// UserRepository defines the contract for data access
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    GetByID(ctx context.Context, id string) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id string) error
    List(ctx context.Context, limit, offset int) ([]*User, error)
}

// UserService defines business logic interface
type UserService interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*User, error)
    GetUser(ctx context.Context, id string) (*User, error)
    UpdateUser(ctx context.Context, id string, req UpdateUserRequest) (*User, error)
    DeleteUser(ctx context.Context, id string) error
    ListUsers(ctx context.Context, limit, offset int) ([]*User, error)
}

// Request/Response models
type CreateUserRequest struct {
    Email string `json:"email" validate:"required,email"`
    Name  string `json:"name" validate:"required,min=2,max=100"`
}

type UpdateUserRequest struct {
    Name string `json:"name" validate:"min=2,max=100"`
}
```

### Use Case Implementation

```go
// internal/usecase/user_usecase.go - Business logic layer
package usecase

import (
    "context"
    "fmt"
    "time"

    "github.com/google/uuid"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"

    "myservice/internal/domain"
    "myservice/pkg/errors"
)

type userUseCase struct {
    userRepo domain.UserRepository
    tracer   trace.Tracer
}

// NewUserUseCase creates a new user use case with dependency injection
func NewUserUseCase(userRepo domain.UserRepository) domain.UserService {
    return &userUseCase{
        userRepo: userRepo,
        tracer:   otel.Tracer("user-usecase"),
    }
}

func (u *userUseCase) CreateUser(ctx context.Context, req domain.CreateUserRequest) (*domain.User, error) {
    ctx, span := u.tracer.Start(ctx, "CreateUser")
    defer span.End()

    span.SetAttributes(
        attribute.String("user.email", req.Email),
        attribute.String("user.name", req.Name),
    )

    // Validate business rules
    if err := u.validateCreateUserRequest(req); err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("validation failed: %w", err)
    }

    // Check if user already exists
    existingUser, err := u.userRepo.GetByEmail(ctx, req.Email)
    if err != nil && !errors.IsNotFound(err) {
        span.RecordError(err)
        return nil, fmt.Errorf("failed to check existing user: %w", err)
    }
    if existingUser != nil {
        err := errors.NewConflictError("user with email already exists")
        span.RecordError(err)
        return nil, err
    }

    // Create new user
    user := &domain.User{
        ID:        uuid.New().String(),
        Email:     req.Email,
        Name:      req.Name,
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }

    if err := u.userRepo.Create(ctx, user); err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("failed to create user: %w", err)
    }

    span.SetAttributes(attribute.String("user.id", user.ID))
    return user, nil
}

func (u *userUseCase) GetUser(ctx context.Context, id string) (*domain.User, error) {
    ctx, span := u.tracer.Start(ctx, "GetUser")
    defer span.End()

    span.SetAttributes(attribute.String("user.id", id))

    user, err := u.userRepo.GetByID(ctx, id)
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("failed to get user: %w", err)
    }

    return user, nil
}

func (u *userUseCase) validateCreateUserRequest(req domain.CreateUserRequest) error {
    if req.Email == "" {
        return errors.NewValidationError("email is required")
    }
    if req.Name == "" {
        return errors.NewValidationError("name is required")
    }
    if len(req.Name) < 2 || len(req.Name) > 100 {
        return errors.NewValidationError("name must be between 2 and 100 characters")
    }
    return nil
}
```

## Repository Implementation

### PostgreSQL Repository with OpenTelemetry

```go
// internal/repository/postgres/user_repository.go
package postgres

import (
    "context"
    "database/sql"
    "fmt"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"

    "myservice/internal/domain"
    "myservice/pkg/errors"
)

type userRepository struct {
    db     *sql.DB
    tracer trace.Tracer
}

func NewUserRepository(db *sql.DB) domain.UserRepository {
    return &userRepository{
        db:     db,
        tracer: otel.Tracer("user-repository"),
    }
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    ctx, span := r.tracer.Start(ctx, "UserRepository.Create")
    defer span.End()

    span.SetAttributes(
        attribute.String("user.id", user.ID),
        attribute.String("user.email", user.Email),
    )

    query := `
        INSERT INTO users (id, email, name, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5)`

    _, err := r.db.ExecContext(ctx, query,
        user.ID, user.Email, user.Name, user.CreatedAt, user.UpdatedAt)
    if err != nil {
        span.RecordError(err)
        return fmt.Errorf("failed to insert user: %w", err)
    }

    return nil
}

func (r *userRepository) GetByID(ctx context.Context, id string) (*domain.User, error) {
    ctx, span := r.tracer.Start(ctx, "UserRepository.GetByID")
    defer span.End()

    span.SetAttributes(attribute.String("user.id", id))

    query := `
        SELECT id, email, name, created_at, updated_at
        FROM users
        WHERE id = $1`

    var user domain.User
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID, &user.Email, &user.Name, &user.CreatedAt, &user.UpdatedAt)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, errors.NewNotFoundError("user not found")
        }
        span.RecordError(err)
        return nil, fmt.Errorf("failed to get user: %w", err)
    }

    return &user, nil
}

func (r *userRepository) GetByEmail(ctx context.Context, email string) (*domain.User, error) {
    ctx, span := r.tracer.Start(ctx, "UserRepository.GetByEmail")
    defer span.End()

    span.SetAttributes(attribute.String("user.email", email))

    query := `
        SELECT id, email, name, created_at, updated_at
        FROM users
        WHERE email = $1`

    var user domain.User
    err := r.db.QueryRowContext(ctx, query, email).Scan(
        &user.ID, &user.Email, &user.Name, &user.CreatedAt, &user.UpdatedAt)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, errors.NewNotFoundError("user not found")
        }
        span.RecordError(err)
        return nil, fmt.Errorf("failed to get user by email: %w", err)
    }

    return &user, nil
}

func (r *userRepository) List(ctx context.Context, limit, offset int) ([]*domain.User, error) {
    ctx, span := r.tracer.Start(ctx, "UserRepository.List")
    defer span.End()

    span.SetAttributes(
        attribute.Int("limit", limit),
        attribute.Int("offset", offset),
    )

    query := `
        SELECT id, email, name, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
        LIMIT $1 OFFSET $2`

    rows, err := r.db.QueryContext(ctx, query, limit, offset)
    if err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("failed to list users: %w", err)
    }
    defer rows.Close()

    var users []*domain.User
    for rows.Next() {
        var user domain.User
        if err := rows.Scan(&user.ID, &user.Email, &user.Name,
            &user.CreatedAt, &user.UpdatedAt); err != nil {
            span.RecordError(err)
            return nil, fmt.Errorf("failed to scan user: %w", err)
        }
        users = append(users, &user)
    }

    if err := rows.Err(); err != nil {
        span.RecordError(err)
        return nil, fmt.Errorf("rows iteration error: %w", err)
    }

    span.SetAttributes(attribute.Int("users.count", len(users)))
    return users, nil
}
```

## HTTP Delivery Layer

### REST API with Gin and OpenTelemetry

```go
// internal/delivery/http/user_handler.go
package http

import (
    "net/http"
    "strconv"

    "github.com/gin-gonic/gin"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"

    "myservice/internal/domain"
    "myservice/pkg/errors"
)

type UserHandler struct {
    userService domain.UserService
    tracer      trace.Tracer
}

func NewUserHandler(userService domain.UserService) *UserHandler {
    return &UserHandler{
        userService: userService,
        tracer:      otel.Tracer("user-handler"),
    }
}

func (h *UserHandler) CreateUser(c *gin.Context) {
    ctx, span := h.tracer.Start(c.Request.Context(), "UserHandler.CreateUser")
    defer span.End()

    var req domain.CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        span.RecordError(err)
        c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request body"})
        return
    }

    span.SetAttributes(
        attribute.String("user.email", req.Email),
        attribute.String("user.name", req.Name),
    )

    user, err := h.userService.CreateUser(ctx, req)
    if err != nil {
        span.RecordError(err)
        h.handleError(c, err)
        return
    }

    span.SetAttributes(attribute.String("user.id", user.ID))
    c.JSON(http.StatusCreated, user)
}

func (h *UserHandler) GetUser(c *gin.Context) {
    ctx, span := h.tracer.Start(c.Request.Context(), "UserHandler.GetUser")
    defer span.End()

    id := c.Param("id")
    span.SetAttributes(attribute.String("user.id", id))

    user, err := h.userService.GetUser(ctx, id)
    if err != nil {
        span.RecordError(err)
        h.handleError(c, err)
        return
    }

    c.JSON(http.StatusOK, user)
}

func (h *UserHandler) ListUsers(c *gin.Context) {
    ctx, span := h.tracer.Start(c.Request.Context(), "UserHandler.ListUsers")
    defer span.End()

    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "10"))
    offset, _ := strconv.Atoi(c.DefaultQuery("offset", "0"))

    span.SetAttributes(
        attribute.Int("limit", limit),
        attribute.Int("offset", offset),
    )

    users, err := h.userService.ListUsers(ctx, limit, offset)
    if err != nil {
        span.RecordError(err)
        h.handleError(c, err)
        return
    }

    span.SetAttributes(attribute.Int("users.count", len(users)))
    c.JSON(http.StatusOK, gin.H{"users": users})
}

func (h *UserHandler) handleError(c *gin.Context, err error) {
    switch {
    case errors.IsNotFound(err):
        c.JSON(http.StatusNotFound, gin.H{"error": err.Error()})
    case errors.IsValidation(err):
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
    case errors.IsConflict(err):
        c.JSON(http.StatusConflict, gin.H{"error": err.Error()})
    default:
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Internal server error"})
    }
}
```

## Testing Strategy

### Unit Testing with Table-Driven Tests

```go
// internal/usecase/user_usecase_test.go
package usecase

import (
    "context"
    "testing"
    "time"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "go.uber.org/mock/gomock"

    "myservice/internal/domain"
    "myservice/pkg/errors"
    "myservice/test/mocks"
)

func TestUserUseCase_CreateUser(t *testing.T) {
    tests := []struct {
        name           string
        request        domain.CreateUserRequest
        setupMocks     func(*mocks.MockUserRepository)
        expectedError  string
        expectedResult bool
    }{
        {
            name: "successful user creation",
            request: domain.CreateUserRequest{
                Email: "test@example.com",
                Name:  "Test User",
            },
            setupMocks: func(repo *mocks.MockUserRepository) {
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "test@example.com").
                    Return(nil, errors.NewNotFoundError("user not found"))
                repo.EXPECT().
                    Create(gomock.Any(), gomock.Any()).
                    Return(nil)
            },
            expectedResult: true,
        },
        {
            name: "user already exists",
            request: domain.CreateUserRequest{
                Email: "existing@example.com",
                Name:  "Test User",
            },
            setupMocks: func(repo *mocks.MockUserRepository) {
                existingUser := &domain.User{
                    ID:    "existing-id",
                    Email: "existing@example.com",
                    Name:  "Existing User",
                }
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "existing@example.com").
                    Return(existingUser, nil)
            },
            expectedError: "user with email already exists",
        },
        {
            name: "invalid email",
            request: domain.CreateUserRequest{
                Email: "",
                Name:  "Test User",
            },
            expectedError: "email is required",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()

            mockRepo := mocks.NewMockUserRepository(ctrl)
            if tt.setupMocks != nil {
                tt.setupMocks(mockRepo)
            }

            useCase := NewUserUseCase(mockRepo)
            ctx := context.Background()

            result, err := useCase.CreateUser(ctx, tt.request)

            if tt.expectedError != "" {
                assert.Error(t, err)
                assert.Contains(t, err.Error(), tt.expectedError)
                assert.Nil(t, result)
            } else {
                assert.NoError(t, err)
                if tt.expectedResult {
                    assert.NotNil(t, result)
                    assert.Equal(t, tt.request.Email, result.Email)
                    assert.Equal(t, tt.request.Name, result.Name)
                    assert.NotEmpty(t, result.ID)
                }
            }
        })
    }
}

func TestUserUseCase_GetUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := mocks.NewMockUserRepository(ctrl)
    useCase := NewUserUseCase(mockRepo)
    ctx := context.Background()

    expectedUser := &domain.User{
        ID:        "test-id",
        Email:     "test@example.com",
        Name:      "Test User",
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }

    mockRepo.EXPECT().
        GetByID(ctx, "test-id").
        Return(expectedUser, nil)

    result, err := useCase.GetUser(ctx, "test-id")

    assert.NoError(t, err)
    assert.Equal(t, expectedUser, result)
}
```

### Integration Testing

```go
// test/integration/user_integration_test.go
package integration

import (
    "bytes"
    "context"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/suite"

    "myservice/internal/domain"
    httpdelivery "myservice/internal/delivery/http"
    "myservice/internal/usecase"
    "myservice/test/fixtures"
)

type UserIntegrationTestSuite struct {
    suite.Suite
    router      *gin.Engine
    userHandler *httpdelivery.UserHandler
    cleanup     func()
}

func (suite *UserIntegrationTestSuite) SetupSuite() {
    // Setup test database and dependencies
    db, cleanup := fixtures.SetupTestDB()
    suite.cleanup = cleanup

    userRepo := fixtures.NewTestUserRepository(db)
    userService := usecase.NewUserUseCase(userRepo)
    suite.userHandler = httpdelivery.NewUserHandler(userService)

    // Setup router
    gin.SetMode(gin.TestMode)
    suite.router = gin.New()
    suite.setupRoutes()
}

func (suite *UserIntegrationTestSuite) TearDownSuite() {
    if suite.cleanup != nil {
        suite.cleanup()
    }
}

func (suite *UserIntegrationTestSuite) setupRoutes() {
    v1 := suite.router.Group("/api/v1")
    {
        users := v1.Group("/users")
        {
            users.POST("", suite.userHandler.CreateUser)
            users.GET("/:id", suite.userHandler.GetUser)
            users.GET("", suite.userHandler.ListUsers)
        }
    }
}

func (suite *UserIntegrationTestSuite) TestCreateUser() {
    createReq := domain.CreateUserRequest{
        Email: "integration@example.com",
        Name:  "Integration Test User",
    }

    body, _ := json.Marshal(createReq)
    req := httptest.NewRequest(http.MethodPost, "/api/v1/users", bytes.NewBuffer(body))
    req.Header.Set("Content-Type", "application/json")

    w := httptest.NewRecorder()
    suite.router.ServeHTTP(w, req)

    assert.Equal(suite.T(), http.StatusCreated, w.Code)

    var response domain.User
    err := json.Unmarshal(w.Body.Bytes(), &response)
    assert.NoError(suite.T(), err)
    assert.Equal(suite.T(), createReq.Email, response.Email)
    assert.Equal(suite.T(), createReq.Name, response.Name)
    assert.NotEmpty(suite.T(), response.ID)
}

func TestUserIntegrationTestSuite(t *testing.T) {
    suite.Run(t, new(UserIntegrationTestSuite))
}
```

## OpenTelemetry Observability

### Telemetry Setup and Configuration

```go
// pkg/telemetry/telemetry.go
package telemetry

import (
    "context"
    "fmt"
    "time"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/exporters/prometheus"
    "go.opentelemetry.io/otel/metric"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/sdk/metric"
    "go.opentelemetry.io/otel/sdk/resource"
    "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.17.0"
)

type Config struct {
    ServiceName    string
    ServiceVersion string
    Environment    string
    JaegerEndpoint string
}

func InitTelemetry(ctx context.Context, cfg Config) (func(), error) {
    // Create resource
    res, err := resource.New(ctx,
        resource.WithAttributes(
            semconv.ServiceName(cfg.ServiceName),
            semconv.ServiceVersion(cfg.ServiceVersion),
            semconv.DeploymentEnvironment(cfg.Environment),
        ),
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create resource: %w", err)
    }

    // Setup tracing
    tracerProvider, err := setupTracing(ctx, res, cfg.JaegerEndpoint)
    if err != nil {
        return nil, fmt.Errorf("failed to setup tracing: %w", err)
    }

    // Setup metrics
    meterProvider, err := setupMetrics(ctx, res)
    if err != nil {
        return nil, fmt.Errorf("failed to setup metrics: %w", err)
    }

    // Set global providers
    otel.SetTracerProvider(tracerProvider)
    otel.SetMeterProvider(meterProvider)
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))

    return func() {
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()

        if err := tracerProvider.Shutdown(ctx); err != nil {
            fmt.Printf("Error shutting down tracer provider: %v\n", err)
        }
        if err := meterProvider.Shutdown(ctx); err != nil {
            fmt.Printf("Error shutting down meter provider: %v\n", err)
        }
    }, nil
}

func setupTracing(ctx context.Context, res *resource.Resource, endpoint string) (*trace.TracerProvider, error) {
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(jaeger.WithEndpoint(endpoint)))
    if err != nil {
        return nil, err
    }

    tp := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
        trace.WithResource(res),
        trace.WithSampler(trace.AlwaysSample()),
    )

    return tp, nil
}

func setupMetrics(ctx context.Context, res *resource.Resource) (*metric.MeterProvider, error) {
    exporter, err := prometheus.New()
    if err != nil {
        return nil, err
    }

    mp := metric.NewMeterProvider(
        metric.WithResource(res),
        metric.WithReader(exporter),
    )

    return mp, nil
}
```

### Middleware for HTTP Instrumentation

```go
// internal/delivery/middleware/telemetry.go
package middleware

import (
    "time"

    "github.com/gin-gonic/gin"
    "go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/metric"
)

type TelemetryMiddleware struct {
    requestDuration metric.Float64Histogram
    requestCounter  metric.Int64Counter
}

func NewTelemetryMiddleware() (*TelemetryMiddleware, error) {
    meter := otel.Meter("http-middleware")

    requestDuration, err := meter.Float64Histogram(
        "http_request_duration_seconds",
        metric.WithDescription("Duration of HTTP requests in seconds"),
    )
    if err != nil {
        return nil, err
    }

    requestCounter, err := meter.Int64Counter(
        "http_requests_total",
        metric.WithDescription("Total number of HTTP requests"),
    )
    if err != nil {
        return nil, err
    }

    return &TelemetryMiddleware{
        requestDuration: requestDuration,
        requestCounter:  requestCounter,
    }, nil
}

func (tm *TelemetryMiddleware) Handler() gin.HandlerFunc {
    return gin.HandlerFunc(func(c *gin.Context) {
        start := time.Now()

        // Use OpenTelemetry Gin middleware for automatic tracing
        otelgin.Middleware("user-service")(c)

        c.Next()

        duration := time.Since(start).Seconds()
        status := c.Writer.Status()

        // Record metrics
        labels := []attribute.KeyValue{
            attribute.String("method", c.Request.Method),
            attribute.String("route", c.FullPath()),
            attribute.Int("status_code", status),
        }

        tm.requestDuration.Record(c.Request.Context(), duration, metric.WithAttributes(labels...))
        tm.requestCounter.Add(c.Request.Context(), 1, metric.WithAttributes(labels...))
    })
}
```

## Error Handling and Resilience

### Custom Error Types

```go
// pkg/errors/errors.go
package errors

import (
    "errors"
    "fmt"
)

// Error types
type ErrorType string

const (
    ErrorTypeValidation ErrorType = "VALIDATION"
    ErrorTypeNotFound   ErrorType = "NOT_FOUND"
    ErrorTypeConflict   ErrorType = "CONFLICT"
    ErrorTypeInternal   ErrorType = "INTERNAL"
)

type AppError struct {
    Type    ErrorType `json:"type"`
    Message string    `json:"message"`
    Code    string    `json:"code,omitempty"`
    Details any       `json:"details,omitempty"`
}

func (e *AppError) Error() string {
    return e.Message
}

// Constructor functions
func NewValidationError(message string) *AppError {
    return &AppError{
        Type:    ErrorTypeValidation,
        Message: message,
    }
}

func NewNotFoundError(message string) *AppError {
    return &AppError{
        Type:    ErrorTypeNotFound,
        Message: message,
    }
}

func NewConflictError(message string) *AppError {
    return &AppError{
        Type:    ErrorTypeConflict,
        Message: message,
    }
}

func NewInternalError(message string) *AppError {
    return &AppError{
        Type:    ErrorTypeInternal,
        Message: message,
    }
}

// Type checking functions
func IsValidation(err error) bool {
    var appErr *AppError
    return errors.As(err, &appErr) && appErr.Type == ErrorTypeValidation
}

func IsNotFound(err error) bool {
    var appErr *AppError
    return errors.As(err, &appErr) && appErr.Type == ErrorTypeNotFound
}

func IsConflict(err error) bool {
    var appErr *AppError
    return errors.As(err, &appErr) && appErr.Type == ErrorTypeConflict
}

func IsInternal(err error) bool {
    var appErr *AppError
    return errors.As(err, &appErr) && appErr.Type == ErrorTypeInternal
}
```

## Best Practices Summary

### Development Guidelines

- **Write idiomatic Go code** following standard conventions and patterns
- **Apply Clean Architecture** with clear separation between layers
- **Use interface-driven development** with explicit dependency injection
- **Prefer composition over inheritance** with small, purpose-specific interfaces
- **Write short, focused functions** with single responsibility
- **Handle errors explicitly** using wrapped errors for traceability
- **Avoid global state** and use constructor functions for dependency injection

### Testing Strategy

- **Write comprehensive unit tests** using table-driven patterns
- **Mock external interfaces** cleanly using generated or handwritten mocks
- **Separate fast unit tests** from slower integration and E2E tests
- **Ensure test coverage** for every exported function with behavioral checks
- **Use parallel execution** where appropriate to speed up test runs

### Observability and Monitoring

- **Use OpenTelemetry** for distributed tracing, metrics, and structured logging
- **Propagate context** across all service boundaries (HTTP, gRPC, DB, external APIs)
- **Record important attributes** like request parameters, user ID, and error messages
- **Monitor key metrics** including request latency, throughput, error rate, and resource usage
- **Use structured logging** with JSON format for better observability tool integration

### Security and Resilience

- **Apply input validation** rigorously, especially on external inputs
- **Use secure defaults** for JWT, cookies, and configuration settings
- **Implement retries, exponential backoff, and timeouts** on all external calls
- **Use circuit breakers and rate limiting** for service protection
- **Isolate sensitive operations** with clear permission boundaries

### Performance and Concurrency

- **Use goroutines safely** with proper synchronization mechanisms
- **Implement goroutine cancellation** using context propagation
- **Minimize allocations** and profile before optimizing
- **Use benchmarks** to track performance regressions
- **Guard shared state** with channels or sync primitives

This comprehensive guide provides a solid foundation for building scalable, maintainable Go microservices with proper observability, testing, and modern development practices.
