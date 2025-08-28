# Go Project Directory Structure - Complete Guide

## 1. `cmd/` Directory

**Purpose**: Contains main application entry points (executable programs)

### What goes here:

- **Main applications** - Each subdirectory represents a different executable
- **Entry points** - Files with `main()` function that start your programs
- **Application bootstrapping** - Initial setup, configuration loading, dependency injection

### Naming Convention:

- Each subdirectory should be named after the executable it produces
- Common names: `server`, `cli`, `worker`, `migrate`, `admin`

### Example Structure:

```
cmd/
├── server/          # Web server application
│   └── main.go     # HTTP server entry point
├── cli/            # Command-line tool
│   └── main.go     # CLI application entry point
├── worker/         # Background job processor
│   └── main.go     # Worker process entry point
├── migrate/        # Database migration tool
│   └── main.go     # Migration runner
└── admin/          # Admin panel/tools
    └── main.go     # Admin application entry point
```

### Example Files:

```go
// cmd/server/main.go
package main

import (
    "log"
    "myapp/internal/server"
    "myapp/internal/config"
)

func main() {
    cfg := config.Load()
    srv := server.New(cfg)
    log.Fatal(srv.Start())
}

// cmd/cli/main.go
package main

import (
    "flag"
    "myapp/internal/commands"
)

func main() {
    var action = flag.String("action", "", "Action to perform")
    flag.Parse()

    commands.Execute(*action)
}

// cmd/worker/main.go
package main

import (
    "myapp/internal/worker"
    "myapp/internal/queue"
)

func main() {
    q := queue.New()
    w := worker.New(q)
    w.Start()
}
```

---

## 2. `internal/` Directory

**Purpose**: Private application code that cannot be imported by other projects

### What goes here:

- **Business logic** - Core application functionality
- **Handlers** - HTTP request handlers, gRPC handlers
- **Services** - Business service layer
- **Repository** - Data access layer
- **Models** - Domain models and entities
- **Config** - Configuration structures and loading

### Naming Convention:

- Use clear, descriptive names based on functionality
- Group by domain or layer (handlers, services, models, etc.)

### Example Structure:

```
internal/
├── handlers/        # HTTP/gRPC request handlers
│   ├── user.go     # User-related HTTP handlers
│   ├── auth.go     # Authentication handlers
│   └── product.go  # Product handlers
├── services/        # Business logic layer
│   ├── user.go     # User business logic
│   ├── auth.go     # Authentication service
│   └── email.go    # Email service
├── repository/      # Data access layer
│   ├── user.go     # User database operations
│   ├── product.go  # Product database operations
│   └── postgres/   # Database-specific implementations
│       └── user.go
├── models/          # Domain models/entities
│   ├── user.go     # User model
│   ├── product.go  # Product model
│   └── auth.go     # Authentication models
├── config/          # Configuration management
│   └── config.go   # Config struct and loading
├── middleware/      # HTTP middleware
│   ├── auth.go     # Authentication middleware
│   ├── cors.go     # CORS middleware
│   └── logging.go  # Request logging
├── database/        # Database connection and migrations
│   ├── connection.go
│   └── migrations/
├── queue/           # Message queue implementations
│   └── redis.go
└── worker/          # Background job processors
    └── email.go
```

### Example Files:

```go
// internal/handlers/user.go
package handlers

type UserHandler struct {
    userService *services.UserService
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {}
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {}

// internal/services/user.go
package services

type UserService struct {
    userRepo repository.UserRepository
}

func (s *UserService) CreateUser(user *models.User) error {}
func (s *UserService) GetUserByID(id int) (*models.User, error) {}

// internal/models/user.go
package models

type User struct {
    ID       int       `json:"id" db:"id"`
    Username string    `json:"username" db:"username"`
    Email    string    `json:"email" db:"email"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}
```

---

## 3. `pkg/` Directory

**Purpose**: Library code that can be used by external applications (public API)

### What goes here:

- **Utility functions** - Helper functions used across applications
- **Shared libraries** - Code that other projects might import
- **Generic components** - Reusable components not specific to your business logic
- **SDK/Client libraries** - APIs for other developers to use

### Naming Convention:

- Use generic, descriptive names
- Think about what other projects might want to import
- Common names: `utils`, `logger`, `validator`, `http`, `auth`

### Example Structure:

```
pkg/
├── utils/           # General utility functions
│   ├── string.go   # String manipulation utilities
│   ├── time.go     # Time formatting utilities
│   └── hash.go     # Hashing utilities
├── logger/          # Logging utilities
│   └── logger.go   # Structured logging setup
├── validator/       # Validation utilities
│   └── validator.go # Input validation helpers
├── http/            # HTTP utilities
│   ├── client.go   # HTTP client helpers
│   └── response.go # Response formatting
├── auth/            # Authentication utilities
│   ├── jwt.go      # JWT token utilities
│   └── password.go # Password hashing
├── database/        # Database utilities
│   └── paginate.go # Pagination helpers
└── errors/          # Error handling utilities
    └── errors.go   # Custom error types
```

### Example Files:

```go
// pkg/utils/string.go
package utils

func ToCamelCase(s string) string {}
func ToSnakeCase(s string) string {}
func Slugify(s string) string {}

// pkg/logger/logger.go
package logger

func New(level string) *Logger {}
func (l *Logger) Info(msg string, fields ...Field) {}

// pkg/validator/validator.go
package validator

func ValidateEmail(email string) bool {}
func ValidatePassword(password string) error {}
```

---

## 4. `api/` Directory

**Purpose**: API definitions, routes, and API-related code

### What goes here:

- **Route definitions** - HTTP route setup
- **API versioning** - Different API versions
- **OpenAPI/Swagger specs** - API documentation
- **Middleware setup** - API-specific middleware
- **API schemas** - Request/response schemas

### Example Structure:

```
api/
├── v1/              # API version 1
│   ├── routes.go   # V1 route definitions
│   ├── handlers.go # V1-specific handlers
│   └── schemas.go  # V1 request/response schemas
├── v2/              # API version 2
│   ├── routes.go
│   └── handlers.go
├── middleware/      # API middleware
│   ├── ratelimit.go
│   └── version.go
├── openapi/         # API documentation
│   ├── v1.yaml     # OpenAPI spec for v1
│   └── v2.yaml
└── routes.go        # Main route setup
```

---

## 5. `web/` Directory

**Purpose**: Web assets and templates

### What goes here:

- **Static files** - CSS, JavaScript, images
- **Templates** - HTML templates
- **Frontend assets** - Compiled frontend code

### Example Structure:

```
web/
├── static/          # Static assets
│   ├── css/
│   │   ├── main.css
│   │   └── admin.css
│   ├── js/
│   │   ├── main.js
│   │   └── components.js
│   ├── images/
│   │   ├── logo.png
│   │   └── icons/
│   └── fonts/
├── templates/       # HTML templates
│   ├── layout/
│   │   ├── base.html
│   │   └── header.html
│   ├── pages/
│   │   ├── home.html
│   │   ├── login.html
│   │   └── dashboard.html
│   └── components/
│       ├── navbar.html
│       └── footer.html
└── dist/            # Compiled frontend assets
    ├── bundle.js
    └── styles.css
```

---

## 6. `scripts/` Directory

**Purpose**: Build, deployment, and utility scripts

### What goes here:

- **Build scripts** - Compilation and packaging
- **Deployment scripts** - Deployment automation
- **Database scripts** - Migration and seeding
- **Development tools** - Development helpers

### Example Structure:

```
scripts/
├── build.sh         # Build application
├── deploy.sh        # Deployment script
├── test.sh          # Run tests
├── migrate.sh       # Database migrations
├── seed.sh          # Database seeding
├── docker/          # Docker-related scripts
│   ├── build.sh
│   └── push.sh
└── dev/             # Development scripts
    ├── setup.sh     # Development environment setup
    └── watch.sh     # File watching for development
```

---

## 7. `configs/` Directory

**Purpose**: Configuration files

### What goes here:

- **Environment configs** - Different environment settings
- **Application configs** - App-specific configuration
- **External service configs** - Third-party service configurations

### Example Structure:

```
configs/
├── config.yaml      # Default configuration
├── development.yaml # Development settings
├── production.yaml  # Production settings
├── staging.yaml     # Staging settings
├── database.yaml    # Database configurations
└── services.yaml    # External service configs
```

---

## 8. `docs/` Directory

**Purpose**: Documentation

### What goes here:

- **API documentation** - REST API docs
- **Architecture docs** - System design documentation
- **Deployment guides** - How to deploy
- **Development guides** - How to contribute

### Example Structure:

```
docs/
├── api/             # API documentation
│   ├── endpoints.md
│   └── authentication.md
├── architecture/    # System architecture
│   ├── overview.md
│   └── database.md
├── deployment/      # Deployment guides
│   ├── docker.md
│   └── kubernetes.md
├── development/     # Development guides
│   ├── setup.md
│   └── contributing.md
└── images/          # Documentation images
    └── architecture.png
```

---

## 9. `test/` or `tests/` Directory

**Purpose**: Test files and test utilities

### What goes here:

- **Integration tests** - End-to-end tests
- **Test fixtures** - Test data
- **Test utilities** - Testing helper functions
- **Mocks** - Mock implementations

### Example Structure:

```
test/
├── integration/     # Integration tests
│   ├── api_test.go
│   └── database_test.go
├── fixtures/        # Test data
│   ├── users.json
│   └── products.sql
├── mocks/           # Mock implementations
│   ├── user_service.go
│   └── database.go
├── utils/           # Test utilities
│   ├── setup.go    # Test setup helpers
│   └── assert.go   # Custom assertions
└── e2e/             # End-to-end tests
    └── user_flow_test.go
```

---

## 10. `migrations/` Directory

**Purpose**: Database migrations

### What goes here:

- **Up migrations** - Schema changes
- **Down migrations** - Schema rollbacks
- **Seed data** - Initial data

### Example Structure:

```
migrations/
├── 001_create_users_table.up.sql
├── 001_create_users_table.down.sql
├── 002_add_email_index.up.sql
├── 002_add_email_index.down.sql
└── seeds/
    ├── users.sql
    └── products.sql
```

---

## Naming Best Practices

### File Naming:

- Use **snake_case** for file names: `user_service.go`, `auth_handler.go`
- Be descriptive: `email_validator.go` not `validator.go`
- Group related files: `user.go`, `user_test.go`, `user_mock.go`

### Package Naming:

- Use **lowercase**, single words when possible
- Avoid underscores: `userservice` not `user_service`
- Make it descriptive: `handlers`, `services`, `models`

### Directory Naming:

- Use **lowercase**
- Use plurals for collections: `handlers/`, `models/`, `services/`
- Use descriptive names that indicate purpose

This structure provides clear separation of concerns, makes code easy to find, and follows Go community standards for maintainable, scalable applications.
