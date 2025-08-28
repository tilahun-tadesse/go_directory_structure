# Go Project Structure

## Directory Structure

```
my-go-project/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handlers/
│   │   └── user.go
│   ├── models/
│   │   └── user.go
│   └── config/
│       └── config.go
├── pkg/
│   └── utils/
│       └── validator.go
├── api/
│   └── routes.go
├── web/
│   ├── static/
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   └── templates/
│       └── index.html
├── scripts/
│   └── build.sh
├── docs/
│   └── README.md
├── tests/
│   └── user_test.go
├── .env
├── .gitignore
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

## File Contents

### go.mod

```go
module my-go-project

go 1.21

require (
    github.com/gorilla/mux v1.8.0
    github.com/joho/godotenv v1.5.1
    github.com/lib/pq v1.10.9
)
```

### cmd/server/main.go

```go
package main

import (
    "log"
    "net/http"
    "os"

    "my-go-project/api"
    "my-go-project/internal/config"

    "github.com/joho/godotenv"
)

func main() {
    // Load environment variables
    if err := godotenv.Load(); err != nil {
        log.Println("No .env file found")
    }

    // Initialize configuration
    cfg := config.New()

    // Setup routes
    router := api.SetupRoutes()

    // Start server
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    log.Printf("Server starting on port %s", port)
    log.Fatal(http.ListenAndServe(":"+port, router))
}
```

### internal/config/config.go

```go
package config

import (
    "os"
)

type Config struct {
    Port     string
    DBHost   string
    DBPort   string
    DBUser   string
    DBPass   string
    DBName   string
}

func New() *Config {
    return &Config{
        Port:   getEnv("PORT", "8080"),
        DBHost: getEnv("DB_HOST", "localhost"),
        DBPort: getEnv("DB_PORT", "5432"),
        DBUser: getEnv("DB_USER", "postgres"),
        DBPass: getEnv("DB_PASS", ""),
        DBName: getEnv("DB_NAME", "myapp"),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}
```

### internal/models/user.go

```go
package models

import (
    "time"
)

type User struct {
    ID        int       `json:"id" db:"id"`
    Username  string    `json:"username" db:"username"`
    Email     string    `json:"email" db:"email"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

type CreateUserRequest struct {
    Username string `json:"username" validate:"required,min=3,max=20"`
    Email    string `json:"email" validate:"required,email"`
}
```

### internal/handlers/user.go

```go
package handlers

import (
    "encoding/json"
    "net/http"

    "my-go-project/internal/models"
    "my-go-project/pkg/utils"
)

func CreateUser(w http.ResponseWriter, r *http.Request) {
    var req models.CreateUserRequest

    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }

    if err := utils.ValidateStruct(req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // TODO: Save user to database

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(map[string]string{"message": "User created successfully"})
}

func GetUsers(w http.ResponseWriter, r *http.Request) {
    // TODO: Fetch users from database
    users := []models.User{
        {ID: 1, Username: "john_doe", Email: "john@example.com"},
        {ID: 2, Username: "jane_smith", Email: "jane@example.com"},
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}
```

### pkg/utils/validator.go

```go
package utils

import (
    "fmt"
    "reflect"
    "strings"
)

func ValidateStruct(s interface{}) error {
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)

    for i := 0; i < v.NumField(); i++ {
        field := v.Field(i)
        fieldType := t.Field(i)

        validateTag := fieldType.Tag.Get("validate")
        if validateTag == "" {
            continue
        }

        if err := validateField(field, fieldType.Name, validateTag); err != nil {
            return err
        }
    }

    return nil
}

func validateField(field reflect.Value, fieldName, validateTag string) error {
    rules := strings.Split(validateTag, ",")

    for _, rule := range rules {
        switch {
        case rule == "required":
            if field.IsZero() {
                return fmt.Errorf("%s is required", fieldName)
            }
        case strings.HasPrefix(rule, "min="):
            // Add minimum length validation
        case strings.HasPrefix(rule, "max="):
            // Add maximum length validation
        case rule == "email":
            // Add email validation
        }
    }

    return nil
}
```

### api/routes.go

```go
package api

import (
    "net/http"

    "my-go-project/internal/handlers"

    "github.com/gorilla/mux"
)

func SetupRoutes() *mux.Router {
    r := mux.NewRouter()

    // API routes
    api := r.PathPrefix("/api/v1").Subrouter()

    // User routes
    api.HandleFunc("/users", handlers.GetUsers).Methods("GET")
    api.HandleFunc("/users", handlers.CreateUser).Methods("POST")

    // Health check
    r.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    }).Methods("GET")

    // Static files
    r.PathPrefix("/static/").Handler(http.StripPrefix("/static/",
        http.FileServer(http.Dir("web/static/"))))

    return r
}
```

### tests/user_test.go

```go
package tests

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "my-go-project/api"
    "my-go-project/internal/models"
)

func TestCreateUser(t *testing.T) {
    router := api.SetupRoutes()

    user := models.CreateUserRequest{
        Username: "testuser",
        Email:    "test@example.com",
    }

    jsonData, _ := json.Marshal(user)
    req := httptest.NewRequest("POST", "/api/v1/users", bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")

    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    if w.Code != http.StatusCreated {
        t.Errorf("Expected status code %d, got %d", http.StatusCreated, w.Code)
    }
}

func TestGetUsers(t *testing.T) {
    router := api.SetupRoutes()

    req := httptest.NewRequest("GET", "/api/v1/users", nil)
    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("Expected status code %d, got %d", http.StatusOK, w.Code)
    }
}
```

### .env

```env
PORT=8080
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASS=mypassword
DB_NAME=myapp
JWT_SECRET=your-secret-key
```

### .gitignore

```
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool
*.out

# Go workspace file
go.work

# Environment variables
.env
.env.local

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Log files
*.log

# Build directory
build/
dist/
```

### Makefile

```makefile
.PHONY: build run test clean

APP_NAME=my-go-project
BUILD_DIR=build

build:
	go build -o $(BUILD_DIR)/$(APP_NAME) cmd/server/main.go

run:
	go run cmd/server/main.go

test:
	go test -v ./...

test-coverage:
	go test -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out

clean:
	rm -rf $(BUILD_DIR)
	rm -f coverage.out

install:
	go mod download
	go mod tidy

docker-build:
	docker build -t $(APP_NAME) .

docker-run:
	docker run -p 8080:8080 $(APP_NAME)
```

### README.md

````markdown
# My Go Project

A Go web application with REST API endpoints.

## Getting Started

### Prerequisites

- Go 1.21 or higher
- PostgreSQL (optional)

### Installation

1. Clone the repository
2. Install dependencies:
   ```bash
   make install
   ```
````

3. Copy environment variables:

   ```bash
   cp .env.example .env
   ```

4. Run the application:
   ```bash
   make run
   ```

### API Endpoints

- `GET /health` - Health check
- `GET /api/v1/users` - Get all users
- `POST /api/v1/users` - Create a new user

### Testing

Run tests:

```bash
make test
```

Run tests with coverage:

```bash
make test-coverage
```

### Building

Build the application:

```bash
make build
```

The binary will be created in the `build/` directory.

````

### scripts/build.sh
```bash
#!/bin/bash

set -e

APP_NAME="my-go-project"
BUILD_DIR="build"

echo "Building $APP_NAME..."

# Create build directory if it doesn't exist
mkdir -p $BUILD_DIR

# Build for current platform
go build -o $BUILD_DIR/$APP_NAME cmd/server/main.go

echo "Build completed: $BUILD_DIR/$APP_NAME"

# Optional: Build for multiple platforms
echo "Building for multiple platforms..."

PLATFORMS="linux/amd64 windows/amd64 darwin/amd64"

for platform in $PLATFORMS; do
    GOOS=${platform%/*}
    GOARCH=${platform#*/}

    output="$BUILD_DIR/${APP_NAME}-${GOOS}-${GOARCH}"
    if [ $GOOS = "windows" ]; then
        output+='.exe'
    fi

    env GOOS=$GOOS GOARCH=$GOARCH go build -o $output cmd/server/main.go
    echo "Built: $output"
done

echo "All builds completed!"
````

## Directory Structure Explanation

- **`cmd/`**: Main applications (entry points)
- **`internal/`**: Private application code (not importable by other projects)
- **`pkg/`**: Library code that can be used by external applications
- **`api/`**: API-related code (routes, middleware)
- **`web/`**: Web assets (templates, static files)
- **`scripts/`**: Build and deployment scripts
- **`tests/`**: Test files
- **`docs/`**: Documentation

This structure follows Go community standards and provides a solid foundation for a scalable Go application.
