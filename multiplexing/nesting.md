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

// NestedMux allows to nest handler following the pattern
// {prefix}/{resource}/{id}/{subresource}/{subresource-id}/...
type NestedMux struct {
	mu       sync.RWMutex
	prefix   string
	handlers map[string]http.Handler
}

func NewNestedMux(prefix string) *NestedMux {
	return &NestedMux{
		prefix:   prefix,
		handlers: make(map[string]http.Handler),
	}
}

func (mux *NestedMux) Handle(path string, h http.Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	mux.handlers[path] = h
}

// ServeHTTP implements http.Handler.
func (mux *NestedMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	mux.mu.RLock()
	defer mux.mu.RUnlock()

	ress := PathToResources(r, mux.prefix)
	path := ress.path()
	h, exists := mux.handlers[path]

	if !exists {
		h = http.NotFoundHandler()
	}

	h.ServeHTTP(w, r)
}
```

---

[<< MULTIPLEXING](multiplexing.md) ||  [TABLE OF CONTENTS](../README.md) || [BUILDING BLOCKS TOGETHER >>](buildingblocks.md)
