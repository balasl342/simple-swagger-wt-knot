
# Swagger UI Setup with Knot Framework

This guide explains how to set up Swagger UI with a Go project using the `knot` framework, and how to serve your generated `swagger.json` file.

## Prerequisites

- Go 1.20 or later
- `knot.v1` package
- Swagger documentation generated using `swag` or another Swagger generator tool

## Steps to Set Up Swagger UI

### Step 1: Generate Swagger Documentation

If you haven't already generated the Swagger documentation (`swagger.json`), use the following command to generate it (assuming you're using `swag`):

```bash
swag init
```

This will create a `docs/` folder with a `swagger.json` file.

### Step 2: Download Swagger UI

Download the Swagger UI from the official Swagger UI GitHub repository:

1. Go to [Swagger UI Releases](https://github.com/swagger-api/swagger-ui/releases).
2. Download the latest release.
3. Extract the `dist` folder and rename it to `swagger-ui`.
4. Copy the `swagger-ui` folder to the root of your project.

### Step 3: Set Up Knot Framework

Set up your Go project to serve Swagger UI with `knot.v1` and `net/http`.

Your `main.go` file should look like this:

```go
package main

import (
    "net/http"
    _ "swagger-ui/docs" // Import the generated docs

    "github.com/eaciit/knot/knot.v1"
)

// Serve static files and Swagger JSON using http.Handler
func swaggerHandler() http.Handler {
    return http.StripPrefix("/swagger/", http.FileServer(http.Dir("./swagger-ui")))
}

func swaggerJSONHandler() http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        http.ServeFile(w, r, "./docs/swagger.json")
    })
}

func main() {
    ks := new(knot.Server)
    ks.Address = "localhost:8080"

    // Register other handlers
    ks.Route("/hello", HelloHandler)

    // Serve Swagger UI and swagger.json
    http.Handle("/swagger/", swaggerHandler())
    http.Handle("/swagger.json", swaggerJSONHandler())

    // Start the server
    go func() {
        http.ListenAndServe(":8080", nil)
    }()
    ks.Listen()
}

// Example handler
func HelloHandler(r *knot.WebContext) interface{} {
    return map[string]string{"message": "Hello, World!"}
}
```

### Step 4: Test Swagger UI

1. Run your Go application:

```bash
go run main.go
```

2. Open your browser and navigate to `http://localhost:8080/swagger/` to see the Swagger UI.

If everything is set up correctly, you should see your API documentation listed in the Swagger UI.

## Notes

- Ensure that the `swagger.json` file is being served correctly by visiting `http://localhost:8080/swagger.json`.
- Make sure the paths in `index.html` are pointing to `/swagger.json` (if needed).
