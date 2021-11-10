*Once Web Application with Everything in Go*

## URI handling with the httpx package

### Parsing request URI paths

* Add a type analyzing the URI path
* Shall be able to handle prefixes and nested paths

```go
// file: uri.go

package httpx

import (
    "net/http"
    "strings"
)

// ResourceID is a type that can be used to identify a resource.
type ResourceID struct {
    Name string
    ID   string
}

// ResourceIDs is a type that can be used to identify multiple resources.
type ResourceIDs []ResourceID

// NewResourceID creates a new ResourceID from a URI path.
func NewResourceIDs(r http.Request, prefix string) ResourceIDs {
    trimmed := strings.TrimPrefix(r.URL.Path, prefix)
    parts := strings.Split(trimmed, "/")
    if len(parts) == 0 {
        return nil
    }
    var ids ResourceIDs
    var name string
    for i, part := range parts {
        switch {
        case part == "":
            continue
        case i % 2 == 0:
            name = part
        case i % 2 == 1:
            ids = append(ids, ResourceID{name, part})
            name = ""
        }
    }
    if name != "" {
        ids = append(ids, ResourceID{name, ""})
    }
    return ids
}
``` 

---

[CRUD](crud.md) ||  [TOP](../README.md) || [JSON](json.md)
