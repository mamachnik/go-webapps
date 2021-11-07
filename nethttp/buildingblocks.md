*Once Web Application with Everything in Go*

## The building blocks together

### Idea

* In-memory Cache Server to store named byte sequences
* HTTP Server to serve the cache

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"
)

// CacheServer provides a simple in-memory cache server. Cache is done
// via a map of string to []byte. The sync.RWMutex is used to ensure that
// the cache is thread-safe.
type CacheServer struct {
    mux   sync.RWMutex
    cache map[string][]byte
}

// newCacheServer creates the cache server. It's simply needed to create
// the map of string to []byte.
func newCacheServer() {
    return &CacheServer{
        cache: make(map[string][]byte),
    }
}

// ServeHTTP implements the http.Handler interface. It only handles
// the two HTTP methods: GET and POST. The work itself is done by
// private helper methods.
func (cs *CacheServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Map HTTP methods to individual methods.
    switch r.Method { 
    case http.HandleGet:
        cs.handleGet(w, r)
    case http.HandlePost:
        cs.handlePost(w, r)
    default:
        w.WriteHeader(http.StatusMethodNotAllowed)
    }
}

// handleGet handles GET requests. If the path can be found in the cache,
// its data is returned. Otherwise, the response is set to 404. Only
// read lock is needed.
func (cs *CacheServer) handleGet(w http.ResponseWriter, r *http.Request) {
    cs.mux.RLock()
    defer cs.mux.RUnlock()

    // Look if path is known.
    if data, ok := cs.cache[r.URL.Path]; ok {
        w.Write(data)
        w.WriteHeader(http.StatusOK)
    } else {
        w.WriteHeader(http.StatusNotFound)
    }
}

// handlePost handles POST requests. It does not matter if the path is known.
func (cs *CacheServer) handlePost(w http.ResponseWriter, r *http.Request) {
    cs.mux.Lock()
    defer cs.mux.Unlock()

    // Simply write/update the variable.
    cs.cache[r.URL.Path] = r.Body
    w.WriteHeader(http.StatusOK)
}

// main runs the cache server.
func main() {
    cs := newCacheServer()
    err := http.ListenAndServe(":8080", cs)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[   PREV   ](responses.md) | [   TOP   ](../README.md)
