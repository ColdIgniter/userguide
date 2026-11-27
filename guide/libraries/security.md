# Security

The Security Class contains methods that help protect your site against Cross-Site Request Forgery attacks.

## Loading the Library

If your only interest in loading the library is to handle CSRF protection, then you will never need to load it, as it runs as a filter and has no manual interaction.

If you find a case where you do need direct access though, you may load it through the global function `service()`:

```cfm
<cfset security = service("security")>
```

## Cross-Site Request Forgery (CSRF)

!!! warning
    The CSRF Protection is only available for **POST/PUT/PATCH/DELETE** requests. Requests for other methods are not protected.

### Prerequisite

When you use ColdIgniter's CSRF protection, you still need to code as the following. Otherwise, the CSRF protection may be bypassed.

#### When Auto-Routing is Disabled

Do one of the following:

1. Do not use `routes.add()`, and use HTTP verbs in routes.
2. Check the request method in the controller method before processing.

E.g.:

```cfm
<cfif NOT this.request_.is("post")>
	<cfreturn this.response.setStatusCode(405).setBody("Method Not Allowed")>
</cfif>
```

#### When Auto-Routing is Enabled

1. Check the request method in the controller method before processing.

E.g.:

```cfm
<cfif NOT this.request_.is("post")>
	<cfreturn this.response.setStatusCode(405).setBody("Method Not Allowed")>
</cfif>
```

### Config for CSRF

#### CSRF Protection Methods

!!! warning
    If you use [Session](sessions.md), be sure to use Session based CSRF protection. Cookie based CSRF protection will not prevent Same-site attacks.

By default, the Cookie based CSRF Protection is used. It is [Double Submit Cookie](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#double-submit-cookie) on OWASP Cross-Site Request Forgery Prevention Cheat Sheet.

You can also use Session based CSRF Protection. It is [Synchronizer Token Pattern](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#synchronizer-token-pattern).

You can set to use the Session based CSRF protection by editing the following config parameter value in **app/Config/Security.cfc**:

```cfm
<cfset this.csrfProtection = "session">
```

!!! note "Difference from CodeIgniter"
    A boolean in **app/Config/Security.cfc** is the string `"true"` or `"false"`.

The other settings of **app/Config/Security.cfc** are `tokenName` (`csrf_test_name`), `headerName` (`X-CSRF-TOKEN`), `cookieName` (`csrf_cookie_name`) and `expires` (`7200` seconds, for the cookie).

#### Token Randomization

To mitigate compression side-channel attacks like [BREACH](https://en.wikipedia.org/wiki/BREACH), and prevent an attacker from guessing the CSRF tokens, you can configure token randomization (off by default).

If you enable it, a random mask is added to the token and used to scramble it.

You can enable it by editing the following config parameter value in **app/Config/Security.cfc**:

```cfm
<cfset this.tokenRandomize = "true">
```

#### Token Regeneration

Tokens may be either regenerated on every submission (default) or kept the same throughout the life of the Session or CSRF cookie.

The default regeneration of tokens provides stricter security, but may result in usability concerns as other tokens become invalid (back/forward navigation, multiple tabs/windows, asynchronous actions, etc). You may alter this behavior by editing the following config parameter value in **app/Config/Security.cfc**:

```cfm
<cfset this.regenerate = "false">
```

!!! warning
    If you use Cookie based CSRF protection, and `redirect()` after the submission, you must call `withCookies()` to send the regenerated CSRF cookie. See [Response](../outgoing/response.md) for details.

!!! note
    You can regenerate the CSRF token manually with the `generateHash()` method of the Security class.

#### Redirection on Failure

When a request fails the CSRF validation check, by default, the user is redirected to the previous page in production environment, or a `SecurityException` is thrown in other environments.

!!! note
    In production environment, when you use HTML forms, it is recommended to enable this redirection for a better user experience.

If you want to make it redirect to the previous page, set the following config parameter value to `"true"` in **app/Config/Security.cfc**:

```cfm
<cfset this.redirect = "true">
```

The shipped file sets `redirect` to `"true"` when `request.ENVIRONMENT` is `production`, and to `"false"` in the other environments.

When redirected, an `error` flash message is set and can be displayed to the end user with the following code in your view:

```cfm
<cfoutput>#session().getFlashdata("error")#</cfoutput>
```

This provides a nicer experience than simply crashing.

Even when the redirect value is `true`, AJAX calls will not redirect, but will throw a `SecurityException`.

### Enable CSRF Protection

You can enable CSRF protection by altering your **app/Config/Filters.cfc** and enabling the `csrf` filter globally:

```cfm
<cfset this.globals = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.globals["before"] = ArrayNew(1)>
<cfset ArrayAppend(this.globals["before"], "csrf")>
<cfset this.globals["after"] = ArrayNew(1)>
```

Select URIs can be whitelisted from CSRF protection (for example API endpoints expecting externally POSTed content). You can add these URIs by adding them as exceptions in the filter:

```cfm
<cfset this.globals["before"] = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset csrfOptions = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset csrfOptions["except"] = "api/record/save">
<cfset this.globals["before"]["csrf"] = csrfOptions>
```

Regular expressions are also supported (case-insensitive). A slash is a slash and an asterisk matches any characters, and the pattern must match the whole URI path:

```cfm
<cfset csrfOptions["except"] = "api/record/[0-9]+">
```

See [Filters](../incoming/filters.md#except-for-a-few-uris), also for a list of patterns.

It is also possible to enable the CSRF filter only for specific methods:

```cfm
<cfset this.methods = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset postFilters = ArrayNew(1)>
<cfset ArrayAppend(postFilters, "csrf")>
<cfset this.methods["POST"] = postFilters>
```

!!! note "Difference from CodeIgniter"
    CodeIgniter warns that `methods` filters can be bypassed through Auto Routing (Legacy). ColdIgniter does not have it. Auto Routing (Improved) runs only a controller method with the HTTP verb of the request in its name.

### HTML Forms

If you use the [form helper](../helpers/form_helper.md), then `form_open()` will automatically insert a hidden csrf field in your forms.

!!! note
    To use auto-generation of CSRF field, you need to turn CSRF filter on to the form page. In most cases it is requested using the `GET` method.

If not, then you can use the always available `csrf_token()` and `csrf_hash()` functions:

```cfm
<cfoutput><input type="hidden" name="#csrf_token()#" value="#csrf_hash()#"></cfoutput>
```

Additionally, you can use the `csrf_field()` method to generate this hidden input field for you:

```cfm
<!--- Generates: <input type="hidden" name="{csrf_token}" value="{csrf_hash}"> --->
<cfoutput>#csrf_field()#</cfoutput>
```

When sending a JSON request the CSRF token can also be passed as one of the parameters. The next way to pass the CSRF token is a special Http header that's name is available by `csrf_header()` function.

Additionally, you can use the `csrf_meta()` method to generate this handy meta tag for you:

```cfm
<!--- Generates: <meta name="{csrf_header}" content="{csrf_hash}"> --->
<cfoutput>#csrf_meta()#</cfoutput>
```

### The Order of Token Sent by Users

The order of checking the availability of the CSRF token is as follows:

1. The POST data
2. HTTP header
3. The raw body of the request, as JSON - bear in mind that this approach is the slowest one since we have to parse the JSON
4. The raw body of the request, as form-encoded data - for PUT, PATCH, and DELETE type of requests

!!! note "Difference from CodeIgniter"
    CodeIgniter decodes and encodes the JSON body again to remove the token, which changes its formatting. ColdIgniter cuts the token member out of the text of the body, together with the comma next to it, and keeps the rest of the body as it was written. JSON has no types here, so a number or `true` in the token member counts as the text it is written with, and is refused as a token.

!!! note
    To keep the JSON body unchanged anyway, send the token in the `X-CSRF-TOKEN` header and leave the CSRF field out of the body.

## Other Helpful Methods

You will never need to use most of the methods in the Security class directly. The following are methods that you might find helpful that are not related to the CSRF protection.

### sanitizeFilename()

Tries to sanitize filenames in order to prevent directory traversal attempts and other security threats, which is particularly useful for files that were supplied via user input. The first parameter is the path to sanitize.

If it is acceptable for the user input to include relative paths, e.g., **file/in/some/approved/folder.txt**, you can set the second optional parameter, `relativePath` to `true`.

```cfm
<cfset path = service("security").sanitizeFilename(this.request_.getPost("filepath"))>
```

This method is an alias for the `sanitize_filename()` function from the Security helper.

!!! warning "Not ported yet"
    `CheckPhpIni`, the class that CodeIgniter uses to report the settings of the language runtime, is not ported.
