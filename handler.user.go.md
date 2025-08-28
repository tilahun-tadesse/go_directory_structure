package handlers

import (
"encoding/json"
"net/http"

    "my-go-project/internal/models"
    "my-go-project/pkg/utils"

)

func CreateUser(w http.ResponseWriter, r \*http.Request) {
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

func GetUsers(w http.ResponseWriter, r \*http.Request) {
// TODO: Fetch users from database
users := []models.User{
{ID: 1, Username: "john_doe", Email: "john@example.com"},
{ID: 2, Username: "jane_smith", Email: "jane@example.com"},
}

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)

}
