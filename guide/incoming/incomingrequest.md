# IncomingRequest Class

The IncomingRequest class is an object-oriented representation of an HTTP request from a client, like a browser. It extends the [Request](request.md) and [Message](message.md) classes, so it has all their methods, plus the methods listed below.

## Accessing the Request

An instance of the request class is already filled in for you if the current class is a descendant of `ColdIgniter\Controller`. You can reach it as a property of the controller:

```cfm
<cfset method = this.request_.getMethod()>
```

!!! note "Difference from CodeIgniter"
    The property is called `request_`, with an underscore, because `request` is a reserved name in CFML.

If you are not in a controller, but still need the request of the application, use the `request_()` function. It returns the shared request, the same as `service("request")`:

```cfm
<cfset req = request_()>
<cfset req = service("request")>
```

It is better to pass the request in as a dependency if the class is anything other than a controller, and keep it as a property of the class.

## Determining Request Type

A request can be of several types, for example an AJAX request. Check it with `isAJAX()`:

```cfm
<cfif this.request_.isAJAX()>
    <!--- an AJAX request --->
</cfif>
```

`isCLI()` also exists, but ColdIgniter only answers web requests, so it always returns false.

!!! note
    `isAJAX()` depends on the `X-Requested-With` header. Some ways of sending an XHR request from JavaScript, for example `fetch`, do not send it by default. See [AJAX Requests](../general/ajax.md).

### is()

The `is()` method accepts an HTTP method, `"ajax"` or `"json"`, and returns true or false. The argument is not case sensitive. Any other value throws an `InvalidArgumentException`.

```cfm
<cfif this.request_.is("put")>
    <!--- a PUT request --->
</cfif>
<cfif this.request_.is("json")>
    <!--- the Content-Type header holds application/json --->
</cfif>
```

### getMethod()

You can check the HTTP method of the request with `getMethod()`. It is always uppercase, for example `GET`:

```cfm
<cfset method = this.request_.getMethod()>
```

To get a lower case version, wrap the call in `LCase()`:

```cfm
<cfset method = LCase(this.request_.getMethod())>
```

You can also check if the request was made over an HTTPS connection with `isSecure()`:

```cfm
<cfif NOT this.request_.isSecure()>
    <!--- not a secure connection --->
</cfif>
```

!!! important
    The `X-Forwarded-Proto` and `Front-End-Https` headers are only used when the request comes from a trusted proxy that is set in `Config\App`. Otherwise they come from the client and could be forged.

!!! warning "Not ported yet"
    The check of the address of a trusted proxy always fails (see [Request Class](request.md)), so these two headers are never used for now.

## Retrieving Input

You can read the GET, POST, cookie and server values through the request object. The data is not filtered automatically. You get the raw input as it was sent.

A value that is not there is returned as an empty string, which ColdIgniter treats as null. Test for it with `is_null()`. An empty string that was sent is not null. You do not need to test whether an item exists first, so instead of:

```cfm
<cfif StructKeyExists(form, "something")>
    <cfset something = form.something>
<cfelse>
    <cfset something = "">
</cfif>
```

you can write:

```cfm
<cfset something = this.request_.getPost("something")>
```

### Getting Data

#### getGet()

Reads the GET (query string) values.

```cfm
<cfset city = this.request_.getGet("city")>
```

#### getPost()

Reads the POST values.

```cfm
<cfset name = this.request_.getPost("name")>
```

#### getCookie()

Reads the cookies.

```cfm
<cfset remember = this.request_.getCookie("remember")>
```

#### getServer()

Reads the server values. See the [Request Class](request.md).

#### getPostGet()

Reads from POST, and from GET if the value is not in POST. With no index it returns both sets combined, and POST wins when a name is in both.

#### getGetPost()

Reads from GET, and from POST if the value is not in GET. With no index it returns both sets combined, and GET wins when a name is in both.

#### getVar()

!!! important
    This method exists only for backward compatibility. Do not use it in new projects.

`getVar()` reads the request values in the order of the setting `request_order`. The default is `GP`, so the form and URL values are read, and cookies are left out unless the setting includes `C`. If the `Content-Type` header holds `application/json`, it returns the JSON data instead.

### Getting JSON Data

You can read the body of the request as a JSON text with `getJSON()`:

```cfm
<cfset data = this.request_.getJSON()>
```

!!! note
    `getJSON()` cannot tell whether the data is JSON before it reads it. Use it only if you expect JSON. Invalid JSON throws an `HTTPException`.

!!! note "Difference from CodeIgniter"
    A JSON object always becomes a map, so the first parameter (`assoc`) has no effect. The parameters for the depth and for the options are accepted, but only the option `JSON_THROW_ON_ERROR` (4194304) is read: with it, a failure throws the `JsonException` instead of the `HTTPException`.

### Getting Specific Data from JSON

Pass a name to `getJsonVar()` to get one value. You can use "dot" notation to reach a value that is not on the root level:

```cfm
<cfset email = this.request_.getJsonVar("customer.email")>
```

A value that is not there, and a JSON null, are returned as an empty string (null).

### Retrieving Raw Data (PUT, PATCH, DELETE)

You can read the body of the request as raw form data with `getRawInput()`. It decodes the text into a map:

```cfm
<cfset data = this.request_.getRawInput()>
```

Use `getRawInputVar()` to get one value from the raw input and filter it. It accepts the dot notation too:

```cfm
<cfset name = this.request_.getRawInputVar("name")>
```

### Filtering Input Data

You can pass a filter as the second argument of any of these methods, except `getJSON()` and `getRawInput()`. In `getJsonVar()` the filter is the third argument; the second argument is `assoc`, and a filter passed there is ignored. The filter is a constant such as `request.FILTER_VALIDATE_INT`. For the form of the flags, and for what a failed filter returns, see `fetchGlobal()` in the [Request Class](request.md).

```cfm
<cfset id = this.request_.getPost("id", request.FILTER_VALIDATE_INT)>
```

## Retrieving Headers

You can reach any header that was sent with the request through `headers()`. It returns a map. The key is the name of the header and the value is a Header object:

```cfm
<cfset headers = this.request_.headers()>
```

If you only need one header, pass its name to `header()`. The name is not case sensitive. If the header does not exist, you get an empty string (null):

```cfm
<cfset header = this.request_.header("Host")>
```

Use `hasHeader()` to see if the header was sent:

```cfm
<cfif this.request_.hasHeader("Accept")>
    <!--- ... --->
</cfif>
```

If you need the value of a header as one string, with all values on one line, use `getHeaderLine()`:

```cfm
<cfset line = this.request_.getHeaderLine("Accept")>
```

If you need the whole header, with the name and the values in one string, call `__toString()` on the Header object:

```cfm
<cfset text = this.request_.header("Accept").__toString()>
```

## The Request URL

You get a [URI](../libraries/uri.md) object for the current request with `getUri()`. Call `__toString()` on it to get the full URL of the request:

```cfm
<cfset uri = this.request_.getUri()>
<cfset fullUrl = uri.__toString()>
```

The object lets you read any part of the request on its own:

```cfm
<cfset host = uri.getHost()>
<cfset path = uri.getPath()>
<cfset segment = uri.getSegment(1)>
```

You can work with the current URI string, the path relative to your base URL, with `getPath()` of the request:

```cfm
<cfset path = this.request_.getPath()>
```

!!! warning "Not ported yet"
    `getRoutePath()` is not a method of the request. It exists on the site URI object that the request uses internally. Use `getPath()` of the request.

## Uploaded Files

All uploaded files are returned by `getFiles()`, a struct of `UploadedFile` objects, keyed by the field name. The files are read when you first ask for them.

```cfm
<cfset files = this.request_.getFiles()>
```

See [Working with Uploaded Files](../libraries/uploaded_files.md) for the details.

You can get one file, by the name given in the HTML file input:

```cfm
<cfset file = this.request_.getFile("userfile")>
```

You can get an array of the files that were uploaded with the same name in a multiple file upload:

```cfm
<cfset files = this.request_.getFileMultiple("userfile")>
```

!!! note
    The files correspond to the uploaded file fields of the form. Even if a user clicks submit and does not choose a file, the file still exists. Check that a file was really uploaded with the `isValid()` method of `UploadedFile`.

## Content Negotiation

You can negotiate content types with the request through `negotiate()`:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "application/json")>
<cfset ArrayAppend(supported, "text/html")>
<cfset type = this.request_.negotiate("media", supported)>
```

See [Content Negotiation](content_negotiation.md) for more details.

## Class Reference

!!! note
    In addition to the methods listed here, this class has the methods of the [Request Class](request.md) and the [Message Class](message.md).

### isCLI()

Returns true if the request came from the command line. In ColdIgniter it always returns false.

### isAJAX()

Returns true if the request is an AJAX request, otherwise false.

### isSecure()

Returns true if the request is an HTTPS request, otherwise false.

### getVar(index, filter, flags)

Backward compatibility only. See above.

### getGet(index, filter, flags)

`index` is the name of the value to look for, or an array of names. With no index, it returns all GET values. To get all values and filter them, pass an empty index and a filter. Pass an array of names to get several values as a map.

```cfm
<cfset city = this.request_.getGet("city")>
<cfset id = this.request_.getGet("id", request.FILTER_VALIDATE_INT)>
```

### getPost(index, filter, flags)

Like `getGet()`, but it reads the POST values.

### getPostGet(index, filter, flags)

Searches POST first, then GET. If you pass an array of names, you get a map, and each name is searched in that order. With no index, it returns both sets, and POST wins a conflict.

### getGetPost(index, filter, flags)

Searches GET first, then POST. If you pass an array of names, you get a map, and each name is searched in that order. With no index, it returns both sets, and GET wins a conflict.

### getCookie(index, filter, flags)

Like `getPost()`, but it reads the cookies.

!!! note
    Unlike the `get_cookie()` function of the [Cookie Helper](../helpers/cookie_helper.md), this method does not add the cookie prefix of your configuration.

### getServer(index, filter, flags)

Like `getPost()`, but it reads the server values.

### getUserAgent()

Returns the [User Agent](../libraries/user_agent.md) object of the request.

### getPath()

Returns the current URI path relative to the base URL.
