# HTTP Responses

The Response class extends the [HTTP Message Class](../incoming/message.md) with methods only appropriate for
a server responding to the client that called it.

## Working with the Response

A Response class is instantiated for you and passed into your controllers. It can
be accessed through `this.response`. It is the same instance that
`service("response")` returns. We call it the global response instance.

Many times you will not need to touch the class directly, since ColdIgniter takes care of
sending the headers and the body for you. This is great if the page successfully created the content it was asked to.

When things go wrong, or you need to send very specific status codes back, it's there for you.

!!! note "Difference from CodeIgniter"
    There is no global `response()` function. Use `this.response` in a controller, or `service("response")` anywhere else.
    The body is not written by the Response itself: `send()` leaves it for the front controller, which writes it to the client.

### Setting the Output

When you need to set the output of the script directly, and not rely on ColdIgniter to automatically get it, you
do it manually with the `setBody` method. This is usually used in conjunction with setting the status code of
the response:

```cfm
<cfset this.response.setStatusCode(404).setBody(body)>
```

The reason phrase ('OK', 'Created', 'Moved Permanently') will be automatically added, but you can add custom reasons
as the second parameter of the `setStatusCode()` method:

```cfm
<cfset this.response.setStatusCode(404, "Nope. Not here.")>
```

!!! warning "Not ported yet"
    `setJSON()` and `setXML()`, which turn a struct into JSON or XML and set the content type (**system/HTTP/Response.php**), are not ported. For a JSON or XML body in a controller that extends `ResourceController`, use `respond()`, see [API Responses](api_responses.md). Otherwise set the body and the content type by hand with `setBody()` and `setContentType()`.

### Setting Headers

#### setHeader()

Often, you will need to set headers to be set for the response. The Response class makes this very simple to do,
with the `setHeader()` method.

The first parameter is the name of the header. The second parameter is the value,
which can be either a string or a list of values that will be combined correctly when sent to the client.

```cfm
<!--- Sets a header "Content-Type: text/html" --->
<cfset this.response.setHeader("Content-Type", "text/html")>

<!--- If the header exists, it is replaced --->
<cfset this.response.setHeader("Cache-Control", "no-cache")>
```

Using these functions instead of writing the headers yourself allows you to ensure that no headers are sent
prematurely, causing errors.

!!! note
    This method just sets headers to the response instance. So, if you create
    and return another response instance (e.g., if you call `redirect()`),
    the headers set here will not be sent automatically.

#### appendHeader()

If the header exists and can have more than one value, you may use the `appendHeader()` and `prependHeader()`
methods to add the value to the end or beginning of the values list, respectively. The first parameter is the name
of the header, while the second is the value to append or prepend.

```cfm
<cfset this.response.appendHeader("Cache-Control", "max-age=3600")>
<cfset this.response.prependHeader("Cache-Control", "must-revalidate")>
```

#### removeHeader()

Headers can be removed from the response with the `removeHeader()` method, which takes the header name as the only
parameter. This is not case-sensitive.

```cfm
<cfset this.response.removeHeader("Cache-Control")>
```

## Redirect

If you want to create a redirect, use the `redirect()` function.
It returns a `RedirectResponse` instance. It is a different instance from the
global response instance that `service("response")` returns.

!!! warning
    If you set Cookies or Response Headers before you call `redirect()`,
    they are set to the global response instance, and they are not automatically
    copied to the `RedirectResponse` instance. To send them, you need to call
    the `withCookies()` or `withHeaders()` method manually.

!!! important
    If you want to redirect, an instance of `RedirectResponse` must
    be returned in a method of the [Controller](../incoming/controllers.md) or
    the [Controller Filter](../incoming/filters.md).
    If you forget to return the `RedirectResponse`, no redirection will occur.

### Redirect to a URI path

When you want to pass a URI path (relative to baseURL), use `redirect().to()`:

```cfm
<cffunction name="toIndex" access="public" returntype="any" output="no">
	<cfreturn redirect().to("admin/home")>
</cffunction>
```

!!! note
    If there is a fragment in your URL that you want to remove, you can
    use the `method` parameter: `redirect().to("admin/home", "", "refresh")`.
    The second parameter is the status code, and an empty string means the default.

### Redirect to a Defined Route

!!! warning "Not ported yet"
    `redirect().route()` and `redirect("route_name")` are not ported (reverse routing, **system/Router/RouteCollection.php**). They always throw an `HTTPException` with "The route for ... cannot be found". Use `redirect().to()` with the path.

### Redirect Back

When you want to redirect back, use `redirect().back()`:

```cfm
<cffunction name="goBack" access="public" returntype="any" output="no">
	<cfreturn redirect().back()>
</cffunction>
```

!!! note
    `redirect().back()` is not the same as browser "back" button.
    It takes a visitor to "the last page viewed during the Session" when the Session is available.
    If the Session hasn't been loaded, or is otherwise unavailable, then a sanitized version of the HTTP referer will be used.

### Redirect with Cookies

If you set Cookies before you call `redirect()`, they are set to the global
response instance, and they are not automatically copied to the `RedirectResponse`
instance.

To send the Cookies, you need to call the `withCookies()` method manually.

```cfm
<cfset this.response.setCookie("login_time", GetTickCount())>
<cfset this.response.setCookie("login_id", user.id)>

<cfreturn redirect().to("/").withCookies()>
```

### Redirect with Headers

If you set Response Headers before you call `redirect()`, they are set to the
global response instance, and they are not automatically copied to the
`RedirectResponse` instance.

To send the Headers, you need to call the `withHeaders()` method manually.

```cfm
<cfset this.response.setHeader("X-Example", "value")>

<cfreturn redirect().to("/").withHeaders()>
```

### Redirect Status Code

The default HTTP status code for GET requests is 302. However, when using HTTP/1.1
or later, 303 is used for POST/PUT/DELETE requests and 307 for all other requests.

You can specify the status code:

```cfm
<cfreturn redirect().to("/", 301)>
```

If you don't know HTTP status code for redirection, it is recommended to read
[Redirections in HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections).

## Force File Download

!!! warning "Not ported yet"
    `download()` and the `DownloadResponse` class (`setFileName()`, `inline()`, **system/HTTP/Response.php**) are not ported. To send a file by hand, set the `Content-Disposition` and `Content-Type` headers and the body yourself.

### Open File in Browser

!!! warning "Not ported yet"
    See [Force File Download](#force-file-download): `DownloadResponse::inline()` is not ported.

## HTTP Caching

Built into the HTTP specification are tools help the client (often the web browser) cache the results. Used correctly,
this can lead to a huge performance boost to your application because it will tell the client that they don't need
to contact the server at all since nothing has changed. And you can't get faster than that.

This is handled through the `Cache-Control` and `ETag` headers. This guide is not the proper place for a thorough
introduction to all of the cache headers power, but you can get a good understanding over at
[Google Developers](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching).

By default, all response objects sent through ColdIgniter have HTTP caching turned off. The options and exact
circumstances are too varied for us to be able to create a good default other than turning it off.

!!! warning "Not ported yet"
    `setCache()` (and `setLastModified()`, **system/HTTP/Response.php**) are not ported. Set the headers yourself, for example:

```cfm
<cfset this.response.setHeader("Cache-Control", "public, max-age=3600")>
<cfset this.response.setHeader("ETag", etag)>
```

## Class Reference

!!! note
    In addition to the methods listed here, this class inherits the methods from the
    [Message Class](../incoming/message.md).

The methods inherited from the Message Class are:

* `getBody`
* `setBody`
* `appendBody`
* `populateHeaders`
* `headers`
* `header`
* `getHeaders`
* `getHeader`
* `hasHeader`
* `getHeaderLine`
* `setHeader`
* `removeHeader`
* `appendHeader`
* `prependHeader`
* `addHeader`
* `getProtocolVersion`
* `setProtocolVersion`

### getStatusCode

`getStatusCode()`

- Returns the current HTTP status code for this response (number)

Returns the currently status code for this response. If no status code has been set, an `HTTPException`
will be thrown:

```cfm
<cftry>
	<cfset statusCode = this.response.getStatusCode()>
	<cfcatch type="ColdIgniter.HTTP.Exceptions.HTTPException">
		<!--- no status code has been set --->
	</cfcatch>
</cftry>
```

### setStatusCode

`setStatusCode(code [, reason])`

- `code` (number): the HTTP status code
- `reason` (string): an optional reason phrase
- Returns the current Response instance

Sets the HTTP status code that should be sent with this response:

```cfm
<cfset this.response.setStatusCode(404)>
```

The reason phrase will be automatically generated based upon the official lists. If you need to set your own
for a custom status code, you can pass the reason phrase as the second parameter:

```cfm
<cfset this.response.setStatusCode(230, "Tardis initiated")>
```

A code outside 100 to 599, or an unknown code without a reason, throws an `HTTPException`.

### getReasonPhrase

`getReasonPhrase()`

- Returns the current reason phrase (string)

Returns the current reason phrase for this response. If no status has been set, an empty string is returned:

```cfm
<cfoutput>#this.response.getReasonPhrase()#</cfoutput>
```

### setDate

!!! warning "Not ported yet"
    `setDate()` is not ported, and no `Date` header is sent (**system/HTTP/Response.php**).

### setContentType

`setContentType(mime [, charset])`

- `mime` (string): the content type this response represents
- `charset` (string): the character set this response uses, `UTF-8` by default
- Returns the current response instance

Sets the content type this response represents:

```cfm
<cfset this.response.setContentType("text/plain")>
```

By default, the method sets the character set to `UTF-8`. If you need to change this, you can
pass the character set as the second parameter:

```cfm
<cfset this.response.setContentType("text/plain", "x-pig-latin")>
```

### noCache

`noCache()`

- Returns the current response instance

Sets the `Cache-Control` header to turn off all HTTP caching. This is the default setting
of all response messages:

```cfm
<cfset this.response.noCache()>
```

### setCache

!!! warning "Not ported yet"
    `setCache()` is not ported.

### setLastModified

!!! warning "Not ported yet"
    `setLastModified()` is not ported.

### send

`send()`

- Returns the current response instance

Tells the response to send everything back to the client. This will first send the headers,
followed by the response body. For the main application response, you do not need to call
this as it is handled automatically by ColdIgniter.

### setCookie

`setCookie(name [, value [, expire [, domain [, path [, prefix [, secure [, httponly [, samesite]]]]]]]])`

- `name` (struct, Cookie or string): cookie name, *or* a struct of all of the parameters available to this method, *or* an instance of `ColdIgniter\Cookie\Cookie`
- `value` (string): cookie value
- `expire` (number): cookie expiration time in seconds. If set to `0` the cookie will only last as long as the browser is open
- `domain` (string): cookie domain
- `path` (string): cookie path
- `prefix` (string): cookie name prefix. If left empty, the default value from **app/Config/Cookie.cfc** will be used
- `secure` (boolean): whether to only transfer the cookie through HTTPS. If left out or empty, the default value from **app/Config/Cookie.cfc** will be used
- `httponly` (boolean): whether to only make the cookie accessible for HTTP requests (no JavaScript). If left out or empty, the default value from **app/Config/Cookie.cfc** will be used
- `samesite` (string): the value for the SameSite cookie parameter. If left out or empty, the default value from **app/Config/Cookie.cfc** will be used
- Returns the current response instance

Sets a cookie containing the values you specify to the Response instance.

There are two ways to
pass information to this method so that a cookie can be set: Struct
Method, and Discrete Parameters:

**Struct Method**

Using this method, a struct is passed as the first
parameter:

```cfm
<cfset cookie = StructNew()>
<cfset cookie["name"] = "The Cookie Name">
<cfset cookie["value"] = "The Value">
<cfset cookie["expire"] = "86500">
<cfset cookie["domain"] = ".some-domain.com">
<cfset cookie["path"] = "/">
<cfset cookie["prefix"] = "myprefix_">
<cfset cookie["secure"] = true>
<cfset cookie["httponly"] = false>
<cfset cookie["samesite"] = "Lax">

<cfset this.response.setCookie(cookie)>
```

Only the `name` and `value` are required. To delete a cookie set it with the
`value` blank.

The `expire` is set in **seconds**, which will be added to the current
time. Do not include the time, but rather only the number of seconds
from *now* that you wish the cookie to be valid. If the `expire` is
set to zero the cookie will only last as long as the browser is open.

!!! note
    But if the `value` is set to empty string and the `expire` is set to `0`,
    the cookie will be deleted.

For site-wide cookies regardless of how your site is requested, add your
URL to the `domain` starting with a period, like this: `.your-domain.com`

The `path` is usually not needed since the method sets a root path.

The `prefix` is only needed if you need to avoid name collisions with
other identically named cookies for your server.

The `secure` flag is only needed if you want to make it a secure cookie
by setting it to `true`.

The `samesite` value controls how cookies are shared between domains and sub-domains.
Allowed values are `"None"`, `"Lax"`, `"Strict"` or a blank string.

**Discrete Parameters**

If you prefer, you can set the cookie by passing data using individual
parameters:

```cfm
<cfset this.response.setCookie(name, value, expire, domain, path, prefix, secure, httponly, samesite)>
```

!!! note "Difference from CodeIgniter"
    CodeIgniter's `null` is an empty string here, and means "take the default from the config". A `false` for `secure` or `httponly` is kept as `false`.

### deleteCookie

`deleteCookie([name [, domain [, path [, prefix]]]])`

- `name` (string): cookie name (a struct of parameters is also accepted)
- `domain` (string): cookie domain
- `path` (string): cookie path
- `prefix` (string): cookie name prefix

Delete an existing cookie.

!!! note
    This also just sets browser cookie for deleting the cookie.

Only the `name` is required.

The `prefix` is only needed if you need to avoid name collisions with
other identically named cookies for your server.

Provide a `prefix` if cookies should only be deleted for that subset.
Provide a `domain` name if cookies should only be deleted for that domain.
Provide a `path` name if cookies should only be deleted for that path.

If any of the optional parameters are empty, then the same-named
cookie will be deleted across all that apply.

Example:

```cfm
<cfset this.response.deleteCookie("name")>
```

### hasCookie

`hasCookie(name [, value [, prefix]])`

- `name` (string): cookie name
- `value` (string): cookie value
- `prefix` (string): cookie name prefix
- Returns true or false

Checks to see if the Response has a specified cookie or not.

Only the `name` is required. If a `prefix` is specified, it will be prepended to the cookie name.

If no `value` is given, the method just checks for the existence of the named cookie.
If a `value` is given, then the method checks that the cookie exists, and that it
has the prescribed value.

Example:

```cfm
<cfif this.response.hasCookie("my_cookie", "cookie_value")>
	<!--- the response will set this cookie --->
</cfif>
```

### getCookie

`getCookie([name [, prefix]])`

- `name` (string): cookie name
- `prefix` (string): cookie name prefix
- Returns the `Cookie`, a struct of `Cookie` objects, or an empty string (no cookie)

Returns the named cookie, if found, or an empty string.

If no `name` is given, returns all the `Cookie` objects of the response.

Example:

```cfm
<cfset cookie = this.response.getCookie("name")>
```

### getCookies

`getCookies()`

- Returns the cookies of the response (a struct of `Cookie` objects)

Returns all cookies currently set within the Response instance.
These are any cookies that you have specifically specified to set during the current
request only.
