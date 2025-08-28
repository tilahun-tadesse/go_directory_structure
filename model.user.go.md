package models

import (
"time"
)

type User struct {
ID int `json:"id" db:"id"`
Username string `json:"username" db:"username"`
Email string `json:"email" db:"email"`
CreatedAt time.Time `json:"created_at" db:"created_at"`
UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

type CreateUserRequest struct {
Username string `json:"username" validate:"required,min=3,max=20"`
Email string `json:"email" validate:"required,email"`
}
