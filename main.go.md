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
