*Once Web Application with Everything in Go*

## The building blocks together

### Idea

* The cache server will be changed to a JSON cache server
* Each entry contains an ID and a raw content

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"

    "./pkg/httpx"
)

// JSONDoc describes one entry in the cache server.
type JSONDoc struct {
    ID      string          `json:"id"`
    Content json.RawMessage `json:"content"`
}

// JSONCacheServer provides a simple JSON in-memory cache server. Cache is 
// done via a map of string to JSONDoc. The JSONDoc contains the ID and a
// raw content. The sync.RWMutex is used to ensure that the cache is
// thread-safe.
type JSONCacheServer struct {
    mux   sync.RWMutex
    cache map[string]JSONDoc
}

// JSONnewCacheServer creates the cache server. It's simply needed to create
// the map of string to JSONDoc.
func JSONnewCacheServer() {
    return &JSONCacheServer{
        cache: make(map[string]JSONDoc),
    }
}

// ServeHTTPGet retrieves a JSO document out of cache.
func (s *JSONCacheServer) ServeHTTPGet(w http.ResponseWriter, r *http.Request) {
    s.mux.RLock()
    defer s.mux.RUnlock()

    ids := httpx.NewResourceIDs(r, "/api")
    if ids == nil {
        http.Error(w, "resource and ID are missing", http.StatusInvalidRequest)
        return
    }
    // Missing: Check of the resource name.
    id := ids[0].ID
    doc, ok := s.cache[id]
    if !ok {
        http.Error(w, "JSON document not found", http.StatusNotFound)
        return
    }
    err := https.WriteBody(w, httpx.ContentTypeJSON, doc) 
    if err != nil {
        log.Printf("Error writing body: %v", err)
    }
}

// ServeHTTPPost adds a JSON document to the cache.
func (s *JSONnewCacheServer) ServeHTTPPost(w http.ResponseWriter, r *http.Request) {
    s.mux.Lock()
    defer s.mux.Unlock()

    ids := httpx.NewResourceIDs(r, "/api")
    if ids == nil {
        http.Error(w, "resource is missing", http.StatusInvalidRequest)
        return
    }
    // Missing: Check of the resource name.
    var doc JSONDoc
    err := httpx.ReadBody(r, &doc)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    _, ok := s.cache[doc.ID]
    if ok {
        http.Error(w, "JSON document already exists", http.StatusConflict)
        return
    }
    s.cache[doc.ID] = doc
    w.WriteHeader(http.StatusCreated)
}

// ...

// main runs the cache server.
func main() {
    cs := httpx.NewMethodHandler(JSONnewCacheServer()) // Use the wrapper.
    err := http.ListenAndServe(":8080", cs)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[JSON](json.md) || [TOP](../README.md)
