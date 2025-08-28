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
