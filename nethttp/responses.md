*Once Web Application with Everything in Go*

## Web responses

### http.ResponseWriter

* The type `http.ResponseWriter` is an interface type that represents the response being sent back to the client
* It defines methods to access and change the HTTP response status code and headers
* It also defines a method to write the response body
* This method ensures that the response also implements the `io.Writer` interface
* So all functions and methods working with `io.Writer` can also be used with `http.ResponseWriter`

### Example

```go
func (h *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/plain")
    w.Write([]byte("Hello, World!"))
    w.WriteHeader(http.StatusOK)
}
```

---

[   PREV   ](requests.md) | [   TOP   ](../README.md) | [   NEXT   ](buildingblocks.md)
