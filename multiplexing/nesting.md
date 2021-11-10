*Once Web Application with Everything in Go*

## Nesting of handlers

### Requirements in RESTful APIs

* Requests like `GET /api/v1/users/1/addresses/5` show the usage of resources and subresources
* Pattern is `{prefix}/{resource}/{id}/{subresource}/{subresource-id}/...`
* The type `http.ServeMux` does not handle a direct matching as names as the part of the path are changing
* An own multiplexer is needed to handle the requests

### Extend the `httpx` package

```go
// file: nesting.go

package httpx

import (
    "net/http"
    "strings"
    "sync"
)

// NestedMux allows to nest handler following the pattern
// {prefix}/{resource}/{id}/{subresource}/{subresource-id}/...
type NestedMux struct {
    mu        sync.RWMutex
    prefix    string
    resources map[string]http.Handler
    handler   http.Handler
}

// NewNestedMux creates an empty nested multiplexer.
func NewNestedMux(prefix string) *NestedMux {
    return &NestedMux{
        prefix:    prefix,
        resources: make(map[string]*NestedMux),
    }
}

// Handle registers the handler for the given resource name. Nested names are separated by a slash.
func (mux *NestedMux) Handle(name string, h http.Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()

    parts := strings.Split(name, "/")
    switch len(parts) {
    case 0:
        panic("empty resource name")
    case 1:
        submux, exists := mux.resources[parts[0]]
        if exists {
            panic("resource already exists")
        }
        submux.handler = h
    default:
        submux, exist := mux.resources[parts[0]]
        if !exists {
            panic("resource does not exist")
        }
        submux.Handle(strings.Join(parts[1:], "/"), h)
    }
}

// ServeHTTP implements http.Handler.
func (mux *NestedMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    ids := ParseResourceIDs(r.URL.Path, mux.prefix)

    h := mux.retrieveHandler(ids)
    h.ServeHTTP(w, r)
}

// retrieveHandler retrieves the handler based on the resource IDs.
func (mux *NestedMux) retrieveHandler(ids ResourceIDs) http.Handler {
    mux.mu.RLock()
    defer mux.mu.RUnlock()

    switch len(ids) {
    case 0:
        return mux.handler
    case 1:
        submux, exists := mux.resources[ids[0].ID]
        if !exists {
            return http.NotFoundHandler()
        }
        return submux.handler
    default:
        submux, exists := mux.resources[ids[0].ID]
        if !exists {
            return http.NotFoundHandler()
        }
        return submux.retrieveHandler(ids[1:])
    }
}
```

---

[MULTIPLEXING](multiplexing.md) ||  [TOP](../README.md) || [BUILDING BLOCKS TOGETHER](buildingblocks.md)
