# HTTP Messages

The Message class gives you the parts of an HTTP message that requests and responses have in common: the message body, the protocol version, and tools for working with headers.

This class is the parent of the [Request Class](request.md) and of the Response class. You do not use it directly.

## Class Reference

The component is `ColdIgniter\HTTP\Message`. Where the CodeIgniter method returns null, the ColdIgniter method returns an empty string, which ColdIgniter treats as null.

### getBody()

Returns the current message body. If no body was set, it returns an empty string (null).

```cfm
<cfset body = this.request_.getBody()>
<cfif is_null(body)>
    <!--- no body --->
</cfif>
```

### setBody(data)

Sets the body of the message. It returns the message, so calls can be chained.

### appendBody(data)

Adds data to the end of the body. It returns the message.

### populateHeaders()

Reads the headers of the current HTTP request and stores them for later access. The [IncomingRequest Class](incomingrequest.md) calls it for you, so the headers of the current request are available.

!!! note "Difference from CodeIgniter"
    The names are stored as the web server gives them. ColdIgniter does not rewrite `HTTP_ACCEPT_LANGUAGE` into `Accept-Language`. Header names are not case sensitive when you read them, so `accept-language` and `Accept-Language` find the same header.

### headers()

Returns a map of all headers found or set. Each entry is a Header object, or an array of Header objects when a header was added more than once with `addHeader()`. `getHeaders()` does the same.

### header(name)

Returns one Header object. `name` is the header name, in any case. If several headers with this name exist, it returns an array of Header objects. If there is no such header, it returns an empty string (null). `getHeader()` does the same.

```cfm
<cfset header = this.request_.header("Accept-Language")>
<cfset header = this.request_.header("accept-language")>
```

A Header object has `getName()`, `getValue()` and `getValueLine()`. If the header has several values, `getValue()` returns an array of the values. `getValueLine()` returns the values as one string, joined with commas:

```cfm
<cfset line = this.request_.header("Accept-Language").getValueLine()>
```

!!! warning "Not ported yet"
    The second parameter of `header()`, a filter for the value, is not ported.

### hasHeader(name)

Returns true if the header exists, false if not.

### getHeaderLine(name)

Returns the value of the header as a string. If the header has several values, they are joined with commas. If the header does not exist, it returns an empty string.

```cfm
<cfset line = this.request_.getHeaderLine("Accept-Language")>
```

!!! note "Difference from CodeIgniter"
    If the header was added more than once with `addHeader()`, `getHeaderLine()` throws an `InvalidArgumentException`. Use `header()` and read each Header object instead.

### setHeader(name, value)

Sets the value of one header. `name` is not case sensitive. If the header does not exist, it is created. The value can be a string or an array of strings. It returns the message.

```cfm
<cfset this.response.setHeader("Cache-Control", "no-cache")>
```

### removeHeader(name)

Removes the header from the message. `name` is not case sensitive. It returns the message.

```cfm
<cfset this.response.removeHeader("Cache-Control")>
```

### appendHeader(name, value)

Adds a value to an existing header. It returns the message.

```cfm
<cfset this.response.appendHeader("Cache-Control", "max-age=60")>
```

!!! note "Difference from CodeIgniter"
    If the header does not exist yet, ColdIgniter sets it instead of failing. If the header was added more than once with `addHeader()`, it throws an `InvalidArgumentException`.

### prependHeader(name, value)

Adds a value in front of the values of an existing header. It returns the message.

```cfm
<cfset this.response.prependHeader("Cache-Control", "no-store")>
```

!!! note "Difference from CodeIgniter"
    If the header does not exist yet, ColdIgniter sets it instead of failing. If the header was added more than once with `addHeader()`, it throws an `InvalidArgumentException`.

### addHeader(name, value)

Adds a header (not a header value) with the same name as an existing one. Use it only when you must send several headers with the same name, for example `Set-Cookie`. If no header with this name exists, it is set as usual. It returns the message.

```cfm
<cfset this.response.addHeader("Set-Cookie", "foo=bar")>
<cfset this.response.addHeader("Set-Cookie", "baz=qux")>
```

### getProtocolVersion()

Returns the HTTP protocol version of the message. If none was set, it returns `1.1`.

### setProtocolVersion(version)

Sets the HTTP protocol version. Valid values are `1.0`, `1.1`, `2.0` and `3.0`. A text such as `HTTP/1.1` is also accepted. Any other value throws an `HTTPException`. It returns the message.

```cfm
<cfset this.response.setProtocolVersion("1.1")>
```
