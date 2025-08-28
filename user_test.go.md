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

func TestCreateUser(t \*testing.T) {
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

func TestGetUsers(t \*testing.T) {
router := api.SetupRoutes()

    req := httptest.NewRequest("GET", "/api/v1/users", nil)
    w := httptest.NewRecorder()
    router.ServeHTTP(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("Expected status code %d, got %d", http.StatusOK, w.Code)
    }

}
