package config

import (
"os"
)

type Config struct {
Port string
DBHost string
DBPort string
DBUser string
DBPass string
DBName string
}

func New() \*Config {
return &Config{
Port: getEnv("PORT", "8080"),
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
