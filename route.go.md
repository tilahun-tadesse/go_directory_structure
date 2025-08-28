package api

import (
"net/http"

    "my-go-project/internal/handlers"

    "github.com/gorilla/mux"

)

func SetupRoutes() \*mux.Router {
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
