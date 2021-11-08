*Once Web Application with Everything in Go*

## Handling HTTP methods

### HTTP methods are verbs

* HTTP requests contain different methods, such as GET, POST, PUT, DELETE, etc.
* The methods are used to indicate the desired action to be performed on the resource
* So one task is to analyse the methods and decide which one is the most suitable for your application
* We've seen how to do a `switch` statement to handle the methods and to handle

```go
switch r.Method {
case http.MethodGet:
    // Handle GET.
    h.handleGet(w, r)
case http.MethodPost:
    // Handle POST.
    h.handlePost(w, r)
default:
    // Handle other methods or return error.
}
```

### But why repeat this for every handler?

* Let's create our own helper package and use the power of interfaces to handle the methods
* So we first create our HTTP helper package `pkg/httpx`

```go
// File: httpx.go

// Package httpx contains helper functions for the daily work with HTTP.
package httpx
``` 

* And now create our method wrapper `pkg/httpx/methods.go`
* It defines individual interfaces for each HTTP method
* The wrapper contains a `http.Handler` interface
* Its `ServeHTTP` distributes the requests to the handler methods based on a type switch

```go
// File: methods.go

package httpx

import (
    "net/http"
)

type GetHandler interface {
    ServeHTTPGet(w http.ResponseWriter, r *http.Request)
}

type PostHandler interface {
    ServeHTTPPost(w http.ResponseWriter, r *http.Request)
}

type PutHandler interface {
    ServeHTTPPut(w http.ResponseWriter, r *http.Request)
}

type DeleteHandler interface {
    ServeHTTPDelete(w http.ResponseWriter, r *http.Request)
}

type DefaultHandler interface {
    ServeHTTPDefault(w http.ResponseWriter, r *http.Request)
}

// ...

// MethodHandler wraps a http.Handler implementing also individual httpx handler
// interface. It distributes the requests to the handler methods based on a type
// switch In case of no matching method a http.ErrMethodNotAllowed is returned.
type MethodHandler struct {
    h http.Handler
}

func NewMethodHandler(h http.Handler) *MethodHandler {
    return &MethodHandler{h: h}
}

func (h *MethodHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        if h, ok := h.h.(GetHandler); ok {
            h.ServeHTTPGet(w, r)
            return
        }
    case http.MethodPost:
        if h, ok := h.h.(PostHandler); ok {
            h.ServeHTTPPost(w, r)
            return
        }
    case http.MethodPut:
        if h, ok := h.h.(PutHandler); ok {
            h.ServeHTTPPut(w, r)
            return
        }
    case http.MethodDelete:
        if h, ok := h.h.(DeleteHandler); ok {
            h.ServeHTTPDelete(w, r)
            return
        }
    // ...
    default:
        if h, ok := h.h.(DefaultHandler); ok {
            h.ServeHTTPDefault(w, r)
            return
        }
    }
    http.Error(w, http.StatusText(http.StatusMethodNotAllowed), http.StatusMethodNotAllowed)
}
```

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"

    "./pkg/httpx"
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

// ServeHTTPGet handles GET requests. If the path can be found in the cache,
// its data is returned. Otherwise, the response is set to 404. Only
// read lock is needed.
func (cs *CacheServer) ServeHTTPGet(w http.ResponseWriter, r *http.Request) {
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

// ...

// main runs the cache server.
func main() {
    cs := httpx.NewMethodHandler(newCacheServer()) // Use the wrapper.
    err := http.ListenAndServe(":8080", cs)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[   TOP   ](../README.md) | [   NEXT   ](crud.md)
