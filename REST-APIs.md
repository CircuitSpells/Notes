# REST APIs

REST: REpresentational State Transfer.

REST is not a standard; it's an architectural style: a named set of constraints that produce specific properties.

REST constraints:

- client-server
- stateless
- cache
- uniform interface
- layered system
- code-on-demand (optional)

## HTTP Methods

- GET, POST, PUT, DELETE, PATCH.
- HEAD: request a resource but only get header data without the response body.
- OPTIONS: lists the supported HTTP methods on a resource.

you can decorate GET methods with HEAD attribute. This prevents the response body from being transmitted back to the client:

```C#
[HttpGet]
[HttpHead]
public async Task<IActionResult> GetHabits([FromQuery] HabitParams habitParams)
{
    // get habits from the database

    return Ok(habits);
}
```

OPTIONS example:

```C#
[HttpOptions]
public IActionResult Options()
{
    Response.Headers.Add("Allow", "GET, POST, HEAD, OPTIONS");
    return Ok();
}
```

PUT vs PATCH:

- PUT replaces a resource.
- PATCH is a partial update.
- both can optionally support upsert (update an existing record if the specified value already exists, or insert a new record if the specified record doesn’t exist).
  - the downside is that the client has to then provide the resource id, which moves the responsibility of generating a resource id to the client. For this reason, supporting upsert is generally not recommended.

A note on PATCH: consider if it is really necessary for your API; there is additional overhead for implementing support for JSON Patch Documents for both the server and the client. Updates can be done with PUT much more easily.

## HTTP Qualities

Safety: an HTTP method is safe if it has no intended side-effects on the server.

Idempotence: an HTTP method is idempotent if multiple requests have the same effect as a single request.

| Method | Safe | Idempotent | Cacheable |
| ------ | ---- | ---------- | --------- |
| GET    | Y    | Y          | Y         |
| POST   | N    | N          | N         |
| PUT    | N    | Y          | N         |
| DELETE | N    | Y          | N         |
| PATCH  | N    | N          | N         |

note: PATCH is not idempotent because it can, for example, add an element to a collection. Sending the request multiple times would result in multiple elements being added.

REST is not a standard, and therefore exceptions are allowable. For example, if your GET request runs into scenarios where the URL length is too long due to too many query parameters, it is acceptable to use a POST request with a request body instead.

## HTTP Status Codes

Most popular status codes:

- 1xx (Informational)
  - 100 Continue
    - check if a server is available for handling a specific request.
- 2xx (Successful)
  - 200 OK
    - may or may not contain a response body.
  - 201 Created
    - the most correct status code for creating a new resource. Response should contain a location header pointing to a URI containing the newly created resource.
  - 202 Accepted
    - often used for asynchronous processing for long-running requests. Ideally comes with a location header containing the location to poll for the status of the long-running process.
  - 204 No Content
    - should contain an empty body.
- 3xx (Redirection)
  - 301 Moved Permanently
    - used when you want to access a resource at a given URI, but it is no longer available there.
  - 302 Found
    - used when the URI has been temporarily changed.
  - 304 Not Modified
    - used to implement support for ETags and the If-None-Match header.
- 4xx (Client error)
  - 400 Bad Request
    - an invalid request was sent to the API.
  - 401 Unauthorized
    - returned when a user is UNAUTHENTICATED.
  - 403 Forbidden
    - returned when a user is UNAUTHORIZED.
  - 404 Not Found
    - the requested resource cannot be found.
  - 409 Conflict
    - used for when there are concurrent operations on a resource, and the resource has already been modified by some other concurrent request.
  - 415 Unsupported Media Type
    - a media query is sent with a media type that the server doesn't recognize.
  - 422 Unprocessable Content
    - used for a request that is syntactically correct, but for some reason (such as a business rule), you are unable to process the request. Unless there is a specific reason to do so, use a 400 status code instead.
  - 429 Too Many Requests
    - client hit the rate limit.
- 5xx (Server error)
  - 500 Internal Server Error
    - the request is correct but something went wrong on the server.
  - 503 Service Unavailable
    - the server is not ready to handle the request, usually due to being overloaded or down for maintenance.
