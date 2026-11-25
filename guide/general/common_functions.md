# Global Functions and Constants

ColdIgniter provides a few functions and variables that are globally defined, and are available to you at any point. These do not require loading any additional libraries or helpers.

## Global Functions

!!! note "Difference from CodeIgniter"
    ColdFusion MX 6.1 reserves the names of some scopes, so a few functions have a name with a trailing underscore: `request_()` for `request()` and `session_()` for `session()`. The function `response()` does not exist (see below).

### Service Accessors

!!! warning "Not ported yet"
    These functions do not exist: `cookie()`, `cookies()`, `response()`, `timer()`, `view_cell()` and `single_service()`. Use `service("response")` and `service("timer")` for the shared instances, and `service("name")` for any other service. For a model, use `model()`.

#### cache()

Takes an optional key, the cache name of the item to retrieve from cache. If no key is given, it returns the cache engine instance. If a key is given, it returns the value stored in the cache under the key, or an empty string if no value is found.

```cfm
<cfset foo = cache("foo")>
<cfif foo EQ "">
    <cfset foo = "bar">
    <cfset cache().save("foo", foo, 300)>
</cfif>

<!--- you can also use the cache engine like any other object --->
<cfset cache().save("foo", "bar", 300)>
```

!!! note "Difference from CodeIgniter"
    There is no null, so a cached empty string cannot be told from a miss.

#### config()

`config(name, getShared)` is a simpler way of getting config instances from [Factories](../concepts/factories.md). `name` is the config class name, and `getShared` (default `true`) tells whether to return the shared instance. It returns an empty string when the class is not found.

See [Configuration](configuration.md#config) for details.

#### env()

`env(key, default)` is used to retrieve values that have previously been set to the environment, or to return a default value if it is not found. It turns the words `true` and `false` into booleans, and the words `empty` and `null` into an empty string.

It is especially useful in conjunction with **.env** files for setting values that are specific to the environment itself, like database settings, API keys, etc. See [Configuration](configuration.md#getting-variables).

#### esc()

`esc(data, context, encoding)` escapes data for inclusion in web pages, to help prevent XSS attacks.

If `data` is a string, then it simply escapes and returns it. If `data` is an array or a structure, then it loops over it, escaping each value and returning the copy.

Valid context values: `html` (the default), `js`, `url`, `attr` and `raw`.

!!! warning "Not ported yet"
    The context `css` and the `encoding` argument are not ported, because the Escaper library is not ported. The context `js` is a small stand-in for it.

#### helper()

`helper(filename)` loads a helper file. The argument is the name of the helper file to load, or an array of names. For full details, see the [Helper Functions](helpers.md) page.

#### lang()

`lang(line, args, locale)` retrieves text from the language files. `line` is the language filename and the key of the text to retrieve, `args` holds the values to substitute for placeholders, and `locale` is a different locale to be used instead of the current locale. It returns the text, or an array of texts.

For more information, see [Localization](../outgoing/localization.md).

!!! warning "Not ported yet"
    The formatting of the placeholders is not ported (it uses the intl extension in CodeIgniter).

#### model()

`model(name, getShared, conn)` is a simpler way of getting model instances from [Factories](../concepts/factories.md). `name` is the model class name, and `getShared` (default `true`) tells whether to return the shared instance. `conn` is an optional database connection that is passed to the constructor of the model. It returns an empty string when the class is not found.

See [Factories](../concepts/factories.md#model) for details.

#### old()

`old(key, default, escape)` provides a simple way to access "old input data" from submitting a form. `key` is the name of the old form data to check for, `default` is the value to return if the key does not exist, and `escape` is an `esc()` context, or `false` to disable it.

```cfm
<cfoutput>
<input type="text" name="name" value="#old("name")#">
</cfoutput>
```

!!! note
    If you are using the `set_value()`, `set_select()`, `set_checkbox()`, and `set_radio()` functions in the [form helper](../helpers/form_helper.md), this feature is built-in. You only need to use this function when not using the form helper.

#### session_()

`session_(key)` provides a convenient way to access the session class and to retrieve a stored value. It returns the session object when no key is given, else the value found in the session for the key. For more information, see the [Sessions](../libraries/sessions.md) page.

```cfm
<cfset session_().set("foo", "bar")>
<cfset foo = session_("foo")>
```

#### view()

`view(name, data, options)` grabs the current renderer and tells it to render the specified view. It simply provides a convenience method that can be used in controllers and libraries. It returns the output of the view, and does not escape the data: that must be handled by the developer.

Currently, these options are read from the `options` structure:

- `saveData` specifies that data will persist between multiple calls to `view()` within the same request. If you do not want the data to be persisted, specify false.

```cfm
<cfset data = StructNew()>
<cfset data.title = "My Blog">
<cfset options = StructNew()>
<cfset options.saveData = true>
<cfset output = view("blog_template", data, options)>
```

!!! warning "Not ported yet"
    The option `cache` (caching a view for a number of seconds) is not ported.

For more details, see the [Views](../outgoing/views.md) and [View Renderer](../outgoing/view_renderer.md) pages.

### Miscellaneous Functions

!!! warning "Not ported yet"
    These functions do not exist: `app_timezone()`, `function_usable()`, `is_cli()`, `is_really_writable()` and `is_windows()`.

#### csp_script_nonce()

Returns the nonce attribute for a script tag. For example: `nonce="Eskdikejidojdk978Ad8jf"`. It returns an empty string when the content security policy is off. See [Content Security Policy](../outgoing/csp.md).

#### csp_style_nonce()

Returns the nonce attribute for a style tag. For example: `nonce="Eskdikejidojdk978Ad8jf"`. It returns an empty string when the content security policy is off. See [Content Security Policy](../outgoing/csp.md).

#### csrf_token()

Returns the name of the current CSRF token.

#### csrf_header()

Returns the name of the header for the current CSRF token.

#### csrf_hash()

Returns the current CSRF hash value.

#### csrf_field()

Returns a hidden input with the CSRF information already inserted. An optional argument is the `id` of the input:

```html
<input type="hidden" name="{csrf_token}" value="{csrf_hash}">
```

#### csrf_meta()

Returns a meta tag with the CSRF information already inserted. An optional argument is the `id` of the tag:

```html
<meta name="{csrf_header}" content="{csrf_hash}">
```

#### db_connect()

`db_connect(db, getShared)` grabs the current database connection instance, or creates it. `db` is a group name, a structure of settings or a connection, and `getShared` tells whether to return the shared connection. See [Connecting to your Database](../database/connecting.md).

#### force_https()

`force_https(duration)` checks to see if the page is currently being accessed via HTTPS. If it is, then nothing happens. If it is not, it sets the HTTP Strict Transport Security header, which instructs modern browsers to automatically modify any HTTP requests to HTTPS requests for `duration` seconds (default 31536000), and builds a redirect to the same URI through HTTPS.

!!! note "Difference from CodeIgniter"
    The function does not send the redirect itself. It throws `ColdIgniter.HTTP.Exceptions.RedirectException`, which only the `forcehttps` filter catches. If you call `force_https()` directly from a controller, nobody catches the error and the visitor gets an error page (status 500). To force HTTPS, set `forceGlobalSecureRequests` to `true` in **app/Config/App.cfc**; the required `forcehttps` filter then sends the redirect. See [Controller Filters](../incoming/filters.md).

#### log_message()

`log_message(level, message, context)` logs a message using the log handlers defined in **app/Config/Logger.cfc**. It returns nothing.

Level can be one of the following values: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, or `debug`.

Context can be used to substitute values in the message string. For full details, see the [Logging Information](logging.md) page.

#### redirect()

`redirect()` returns a redirect response object, allowing you to easily create redirects. Call it without an argument and use `to()`:

```cfm
<cfreturn redirect().to("/")>
```

See [HTTP Responses](../outgoing/response.md).

!!! warning "Not ported yet"
    `redirect("name")` and `route()` of the redirect response need reverse routing, which is not ported, so they always throw an error. Use `redirect().to()` with a URI.

#### remove_invisible_characters()

`remove_invisible_characters(str, urlEncoded)` prevents inserting null characters between ASCII characters, like `Java\0script`. It returns the sanitized string. `urlEncoded` (default `true`) tells whether to remove URL-encoded characters as well.

```cfm
<cfset cleaned = remove_invisible_characters(input)>
```

#### request_()

Returns the shared request object. This function is a wrapper for `service("request")`.

#### route_to()

`route_to(method, param, ...)` generates a route for you based on a controller::method combination or a route name, taking parameters into effect if provided. It returns a route path (URI path relative to baseURL).

!!! warning "Not ported yet"
    Reverse routing is not ported, so `route_to()` finds no route. Build links with `site_url()` of the [URL Helper](../helpers/url_helper.md).

#### service()

`service(name, param, ...)` provides easy access to any of the [Services](../concepts/services.md) defined in the system. This will always return a shared instance of the class, so no matter how many times this is called during a single request, only one class instance will be created.

```cfm
<cfset logger = service("logger")>
```

#### slash_item()

`slash_item(item)` fetches a config file item (of `Config\App`) with a slash appended (if not empty). An item that does not exist gives an empty string.

#### stringify_attributes()

`stringify_attributes(attributes, js)` converts a string, an array or a structure of attributes to a string. `js` is `true` if values do not need quotes (JavaScript-style).

```cfm
<cfset attributes = StructNew()>
<cfset attributes.class = "wide">
<cfset attributes.id = "main">
<cfoutput><div#stringify_attributes(attributes)#></cfoutput>
```

## Global Constants

The following constants are always available anywhere within your application.

!!! note "Difference from CodeIgniter"
    A constant is a value in the `request` scope. `APPPATH` is read as `request.APPPATH`.

### Core Constants

#### APPPATH

The path to the **app** directory.

#### ROOTPATH

The path to the project root directory. Just above `APPPATH`.

#### SYSTEMPATH

The path to the **system** directory.

#### FCPATH

The path to the directory that holds the front controller.

#### WRITEPATH

The path to the **writable** directory.

### Time Constants

#### SECOND

Equals 1.

#### MINUTE

Equals 60.

#### HOUR

Equals 3600.

#### DAY

Equals 86400.

#### WEEK

Equals 604800.

#### MONTH

Equals 2592000.

#### YEAR

Equals 31536000.

#### DECADE

Equals 315360000.
