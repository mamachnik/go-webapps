*Once Web Application with Everything in Go*

## The building blocks together

### Idea

* Register the RESTful API for users

### Example

```go
package main

import (
    "net/http"

    "./pkg/httpx"
    "./pkg/user"
)

func main() { 
    mux := http.NewServeMux()
    apimux := httpx.NewNestedMux("/api/v1")

    apimux.Handle("users", httpx.MethodWrapper(user.NewUsersHandler()))
    apimux.Handle("users/addresses", httpx.MethodWrapper(user.NewUsersAddressesHandler()))v
    apimux.Handle("users/contracts", httpx.MethodWrapper(user.NewUsersContractsHandler()))

    // ...

    mux.Handle("/api/v1/", apimux)

    // ...

    http.ListenAndServe(":8080", mux)
}
```

---

[NESTING](nesting.md) || [TOP](../README.md)
