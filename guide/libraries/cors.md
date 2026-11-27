# Cross-Origin Resource Sharing (CORS)

Cross-Origin Resource Sharing (CORS) is an HTTP-header based security mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources.

CORS works by adding headers to HTTP requests and responses to indicate whether the requested resource can be shared across different origins, helping to prevent malicious attacks like cross-site request forgery (CSRF) and data theft.

If you are not familiar with CORS and CORS headers, please read the [MDN documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers).

ColdIgniter provides the CORS filter and helper class.

## Configuring CORS

### Setting Default Config

CORS can be configured by **app/Config/Cors.cfc**.

At a minimum, the following items in the `default` property must be set:

- `allowedOrigins`: List explicitly the Origin(s) you want to allow.
- `allowedHeaders`: List explicitly the HTTP headers you want to allow.
- `allowedMethods`: List explicitly the HTTP methods you want to allow.

!!! warning
    Based on the principle of least privilege, only the minimum necessary Origin, Methods, and Headers should be allowed.

If you send credentials (e.g., cookies) with a cross-origin request, set `supportsCredentials` to `true`.

The `default` property is a struct. It has these keys, with these defaults:

| Key | Type | Default |
|---|---|---|
| `allowedOrigins` | array of strings | empty array |
| `allowedOriginsPatterns` | array of strings | empty array |
| `supportsCredentials` | boolean | `false` |
| `allowedHeaders` | array of strings | empty array |
| `exposedHeaders` | array of strings | empty array |
| `allowedMethods` | array of strings | empty array |
| `maxAge` | number of seconds | `7200` |

`allowedOriginsPatterns` holds regular expressions without delimiters, each matched against the whole origin, for example `https://\w+\.example\.com`.

```cfm
<cfset this.default["allowedOrigins"] = ListToArray("https://app.example.com")>
<cfset this.default["allowedHeaders"] = ListToArray("Authorization,Content-Type")>
<cfset this.default["allowedMethods"] = ListToArray("GET,POST,PUT,DELETE")>
```

A config that is wrong makes the framework throw an error of the type `ColdIgniter.Exceptions.ConfigException`.

### Enabling CORS

To enable CORS, you need to do two things:

1. Specify the `cors` filter to routes that you permit CORS.
2. Add **OPTIONS** routes for CORS Preflight Requests.

#### Set against Routes

You can set the `cors` filter to routes with **app/Config/Routes.cfm**, by the `filter` option of each route.

E.g.,

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = "cors">

<cfset routes.resource("product", routeOptions)>

<cfset routes.options("product", "Product::options", routeOptions)>
<cfset routes.options("product/(:any)", "Product::options", routeOptions)>
```

Don't forget to add OPTIONS routes for Preflight Requests. Because Controller Filters (except for Required Filters) do not work if the route does not exist.

The CORS filter handles all Preflight Requests, so the controller methods for the OPTIONS routes are not normally called. Implement them to process normal non-preflight OPTIONS requests, if necessary.

!!! warning "Not ported yet"
    CodeIgniter sets the filter on a group of routes, and the OPTIONS routes can be closures. ColdIgniter has no `group()` and a route handler cannot be a function, see [Routing](../incoming/routing.md). Set the filter on each route, and point the OPTIONS routes to a controller method, as above.

#### Set in Config\Filters

Alternatively, you can set the `cors` filter to URI paths in **app/Config/Filters.cfc**.

E.g.,

```cfm
<cfset corsUris = StructNew()>
<cfset corsBefore = ArrayNew(1)>
<cfset ArrayAppend(corsBefore, "api/*")>
<cfset corsUris["before"] = corsBefore>
<cfset corsAfter = ArrayNew(1)>
<cfset ArrayAppend(corsAfter, "api/*")>
<cfset corsUris["after"] = corsAfter>
<cfset this.filters["cors"] = corsUris>
```

Don't forget to add OPTIONS routes for Preflight Requests. Because Controller Filters (except for Required Filters) do not work if the route does not exist.

E.g.,

```cfm
<cfset routes.options("api/(:any)", "Api::options")>
```

The CORS filter handles all Preflight Requests, so the controller method for the OPTIONS routes is not normally called.

!!! note
    The `cors` alias is set in **app/Config/Filters.cfc** already. It is the class `ColdIgniter\Filters\Cors`.

#### Checking Routes and Filters

!!! warning "Not ported yet"
    The `spark routes` command is not ported, because ColdIgniter has no command line. Read **app/Config/Routes.cfm** and **app/Config/Filters.cfc** to check the routes and the filters.

### Setting Another Config

If you want to use a different configuration than the default configuration, add a property to **app/Config/Cors.cfc**.

For example, add the `api` property.

```cfm
<cfset this.api = StructNew()>
<cfset this.api["allowedOrigins"] = ListToArray("https://app.example.com")>
<cfset this.api["allowedOriginsPatterns"] = ArrayNew(1)>
<cfset this.api["supportsCredentials"] = true>
<cfset this.api["allowedHeaders"] = ListToArray("Authorization,Content-Type")>
<cfset this.api["exposedHeaders"] = ArrayNew(1)>
<cfset this.api["allowedMethods"] = ListToArray("GET,POST,PUT,DELETE")>
<cfset this.api["maxAge"] = 7200>
```

The property name (`api` in the above example) will become the configuration name.

Then, specify the property name as the filter argument like `cors:api`:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = "cors:api">

<cfset routes.resource("user", routeOptions)>

<cfset routes.options("user", "User::options", routeOptions)>
<cfset routes.options("user/(:any)", "User::options", routeOptions)>
```

You can also use [Filter Arguments](../incoming/filters.md#filter-arguments).

## Class Reference

### Cors

The class is `ColdIgniter\HTTP\Cors`. Its constructor takes a struct of the config, a `Config\Cors` instance, or nothing. Missing keys take the defaults above. The method `factory(configName)` makes an instance by the name of a property of `Config\Cors`, with `default` as the default name. Call it on `static_("ColdIgniter\HTTP\Cors")`.

#### addResponseHeaders()

`addResponseHeaders(request_, response)`

- `request_` (IncomingRequest): the request instance.
- `response` (Response): the response instance.
- Returns the response instance.

Adds response headers for CORS.

#### handlePreflightRequest()

`handlePreflightRequest(request_, response)`

- `request_` (IncomingRequest): the request instance.
- `response` (Response): the response instance.
- Returns the response instance.

Handles Preflight Requests. The response gets the status code 204.

#### isPreflightRequest()

`isPreflightRequest(request_)`

- `request_` (IncomingRequest): the request instance.
- Returns `true` if it is a Preflight Request.

Checks if the request is a Preflight Request: an OPTIONS request that has the header `Access-Control-Request-Method`.

!!! note "Difference from CodeIgniter"
    The request arguments are named `request_`, as `request` is a scope in CFML.
