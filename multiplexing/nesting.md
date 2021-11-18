*Once Web Application with Everything in Go*

## Nesting of handlers

### Requirements in RESTful APIs

* Requests like `GET /api/v1/users/1/addresses/5` show the usage of resources and subresources
* Pattern is `{prefix}/{resource}/{id}/{subresource}/{subresource-id}/...`
* The type `http.ServeMux` does not handle a direct matching as names as the part of the path are changing
* An own multiplexer is needed to handle the requests

### Extend the `httpx` package

* Separation of URI path into prefix, resources, and IDs

```go
// file: uri.go

// Resource identifies a resource in a URI path by name and ID.
type Resource struct {
	Name string
	ID   string
}

// Resources is a number or resources in a URI path.
type Resources []Resource

// path returns the number of resource names concatenated with slashes
// like they are stored in the nested multiplexer.
func (ress Resources) path() string {
	names := make([]string, len(ress))
	for i, res := range ress {
		names[i] = res.Name
	}
	return strings.Join(names, "/")
}

// PathToResources parses a new Resource from a URI path.
func PathToResources(r *http.Request, prefix string) Resources {
	// Remove prefix with and without trailing slash.
	prefix = strings.TrimSuffix(prefix, "/")
	trimmed := strings.TrimPrefix(r.URL.Path, prefix)
	trimmed = strings.TrimPrefix(trimmed, "/")
	// Now split the path.
	parts := strings.Split(trimmed, "/")
	if len(parts) == 0 {
		return nil
	}
	var ress Resources
	var name string
	for i, part := range parts {
		switch {
		case i%2 == 0:
			name = part
		case i%2 == 1:
			ress = append(ress, Resource{name, part})
			name = ""
		}
	}
	if name != "" {
		ress = append(ress, Resource{name, ""})
	}
	return ress
}
```

* Multiplexing of requests path to handlers based on resource names

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
