*Once Web Application with Everything in Go*

## RESTul APIs and CRUD

### CRUD

* CRUD methods are used to create, read, update and delete resources
* HTTP methods (also called verbs) are used to indicate the CRUD method
    * POST: Create a new resource
    * GET: Read a resource
    * PUT: Update a resource
    * DELETE: Delete a resource
* RESTful APIs are idempotent, meaning that they can be called multiple times without changing the resource

### More HTTP methods

* Other HTTP methods are used to perform other operations on resources
* They are handled by the HTTP methods HEAD, PATCH, OPTIONS, TRACE and CONNECT of RESTful APIs:
    * HEAD: Used to check if the resource exists and to get the resource's metadata
    * PATCH: Can be used to update a resource partially
    * OPTIONS: Used to check if the resource exists and to get the resource's metadata
    * TRACE: Used for diagnostics and debugging
    * CONNECT: Used to connect to a remote resource

### URI format and components

* URI is the Uniform Resource Identifier
* It is composed of the following parts:
    * Scheme: The protocol used to access the resource
    * Host: The domain name or IP address of the resource
    * Path: The path of the resource
    * Query: The query string of the resource
* The path is used to identify the resource
* It contains the resource's name and the resource's ID
* Names and IDs can be nested
* The path also may have a prefix like `/api/v1`
* The resource's name is always written in lowercase and plural

### Example requests

    // Create a new user.
    POST /api/v1/users

    // Read a user.
    GET /api/v1/users/1

    // Read all addresses of a user.
    GET /api/v1/users/1/addresses

    // Read one address of a user.
    GET /api/v1/users/1/addresses/5

    // Update a user.
    PUT /api/v1/users/1

    // Delete a resource.
    DELETE /api/v1/users/1

---

[METHODS](methods.md) ||  [TOP](../README.md) || [URI](uri.md)
