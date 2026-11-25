# Cookie Helper

The Cookie Helper file contains functions that assist in working with cookies.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("cookie")>
```

## Available Functions

The following functions are available:

### set_cookie()

`set_cookie(name, value, expire, domain, path, prefix, secure, httpOnly, sameSite)`

- `name` (string): the cookie name.
- `value` (string, optional, default `""`): the cookie value.
- `expire` (numeric, optional, default `0`): the number of seconds until the cookie expires. If set to `0` the cookie lasts only as long as the browser is open.
- `domain` (string, optional, default `""`): the cookie domain (usually: `.yourdomain.com`).
- `path` (string, optional, default `/`): the cookie path.
- `prefix` (string, optional, default `""`): the cookie name prefix. If `""`, the default from `app/Config/Cookie.cfc` is used.
- `secure` (boolean, optional): whether to send the cookie only through HTTPS. If left out or `""`, the default from `app/Config/Cookie.cfc` is used.
- `httpOnly` (boolean, optional): whether to hide the cookie from JavaScript. If left out or `""`, the default from `app/Config/Cookie.cfc` is used.
- `sameSite` (string, optional): the value for the SameSite cookie parameter. If left out or `""`, the default from `app/Config/Cookie.cfc` is used.
- Returns nothing.

This helper function gives you friendlier syntax to set browser cookies. Refer to the [Response](../outgoing/response.md) page for a description of its use, as this function is an alias for the `setCookie()` method of the Response.

```cfm
<cfset set_cookie("remember", "yes", request.HOUR)>
```

!!! note
    This helper function just sets browser cookies on the shared response, the one that `service("response")` returns. So, if you create and return another response (for example, if you call `redirect()`), the cookies set here are not sent automatically.

!!! note "Difference from CodeIgniter"
    CodeIgniter takes `null` for `secure`, `httpOnly` and `sameSite` to ask for the default. CFML has no null, so an empty string stands for it.

!!! warning "Not ported yet"
    The first argument must be the cookie name. A map of all the parameters or a Cookie object as the first argument is not ported yet.

### get_cookie()

`get_cookie(index, xssClean, prefix)`

- `index` (string): the cookie name.
- `xssClean` (boolean, optional, default `false`): not used, see below.
- `prefix` (string, optional, default `""`): the cookie name prefix. If `""`, the default from `app/Config/Cookie.cfc` is used.
- Returns the cookie value, or an empty string if the cookie is not found.

This helper function gives you friendlier syntax to get browser cookies. Refer to the [IncomingRequest](../incoming/incomingrequest.md) page for a detailed description of its use, as this function acts very much like the `getCookie()` method of the request, except that it also prepends the `prefix` that you might have set in your `app/Config/Cookie.cfc` file.

```cfm
<cfset remember = get_cookie("remember")>
```

!!! note "Difference from CodeIgniter"
    CodeIgniter returns `null` when the cookie is not found. CFML has no null, so ColdIgniter returns an empty string. For the same reason there is no way to ask for "no prefix" with a null `prefix`: an empty `prefix` always means the default from `app/Config/Cookie.cfc`.

!!! warning "Not ported yet"
    XSS filtering is not ported yet, so the `xssClean` argument has no effect. Use [`esc()`](../general/common_functions.md#esc) with the correct context in the views.

### delete_cookie()

`delete_cookie(name, domain, path, prefix)`

- `name` (string): the cookie name.
- `domain` (string, optional, default `""`): the cookie domain (usually: `.yourdomain.com`).
- `path` (string, optional, default `/`): the cookie path.
- `prefix` (string, optional, default `""`): the cookie name prefix.
- Returns nothing.

Lets you delete a cookie. Unless you have set a custom path or other values, only the name of the cookie is needed.

```cfm
<cfset delete_cookie("name")>
```

This function is otherwise the same as `set_cookie()`, except that it does not have the `value` and `expire` parameters.

This too just sets browser cookies, for deleting the cookies, on the shared response that `service("response")` returns.

!!! note
    When you use `set_cookie()`, if the `value` is an empty string and the `expire` is `0`, the cookie is deleted. If the `value` is a non-empty string and the `expire` is `0`, the cookie lasts only as long as the browser is open.

You set the parameters one by one:

```cfm
<cfset delete_cookie(name, domain, path, prefix)>
```

### has_cookie()

`has_cookie(name, value, prefix)`

- `name` (string): the cookie name.
- `value` (string, optional, default `""`): the cookie value. If `""`, the value is not checked.
- `prefix` (string, optional, default `""`): the cookie prefix.
- Returns `true` or `false`.

Checks if a cookie exists by name in the shared response that `service("response")` returns. This is an alias of the `hasCookie()` method of the Response.

```cfm
<cfif has_cookie("remember")>
	<!--- the response already has the cookie --->
</cfif>
```
