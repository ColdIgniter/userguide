# Request Class

The request class is an object-oriented representation of an HTTP request. It is meant to work for both incoming requests, such as a request to the application from a browser, and outgoing requests, such as a request the application sends to another service. The class gives the functions both cases need. Each case has its own class that extends it and adds what it needs. In practice you will use those classes.

See the [IncomingRequest Class](incomingrequest.md) for more usage details.

## Class Reference

The component is `ColdIgniter\HTTP\Request`. It extends `OutgoingRequest`, which extends the [Message](message.md) class.

### getIPAddress()

Returns the IP address of the current user, as the web server reports it (`REMOTE_ADDR`):

```cfm
<cfset ip = this.request_.getIPAddress()>
```

!!! warning "Not ported yet"
    The setting for trusted proxy addresses is not used yet, so a client address sent in a proxy header is never used, and an address that is not valid is not replaced by `0.0.0.0`.

### isValidIP(ip, which)

!!! warning "Not ported yet"
    `isValidIP()` is not ported. It is deprecated in CodeIgniter.

### getMethod()

Returns the HTTP request method of the current request, for example `GET` or `POST`. If a form was sent with a spoofed method (see [HTTP Method Spoofing](methodspoofing.md)), it returns the spoofed method.

```cfm
<cfset method = this.request_.getMethod()>
```

### setMethod(method)

Sets the request method. It is used when spoofing the request. It returns the request.

### withMethod(method)

!!! warning "Not ported yet"
    `withMethod()` is not ported. Use `setMethod()`.

### getServer(index, filter, flags)

Returns a value from the server variables, or an empty string (null) if it is not found. It works like `getPost()`, `getGet()` and `getCookie()` of the [IncomingRequest Class](incomingrequest.md):

```cfm
<cfset method = this.request_.getServer("REQUEST_METHOD")>
```

To get several values, pass an array of the names.

The `filter` is a filter constant, such as `request.FILTER_VALIDATE_INT`. The `flags` are a number, or a map with the keys `flags` and `options`.

### getEnv(index, filter, flags)

This method is deprecated. It has the same arguments as `getServer()`. Read environment values with the `env()` function instead.

### setGlobal(name, value)

Sets the value of a global by hand. `name` is one of `cookie`, `get`, `post`, `request` or `server`. It returns the request.

### fetchGlobal(name, index, filter, flags)

Fetches one or more values from a global, like cookies, get or post values. `name` is one of `cookie`, `get`, `post`, `request` or `server`. `index` is a name, an array of names, or empty to get all values. You can filter the value by passing a filter and flags.

!!! note "Difference from CodeIgniter"
    A value that is not there is returned as an empty string (null), and it is not filtered. When a filter fails, the result is the text `false`. On ColdFusion MX 6.1, `0 EQ false` is true, so do not test a failure with `EQ`. Use `IsBoolean(x) AND NOT IsNumeric(x) AND NOT x`.
