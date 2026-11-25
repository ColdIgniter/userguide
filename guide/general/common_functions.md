# Global Functions and Constants

ColdIgniter provides a few functions and variables that are globally defined, and are available to you at any point. These do not require loading any additional libraries or helpers.

## Global Functions

!!! note "Difference from CodeIgniter"
    ColdFusion MX 6.1 reserves the names of some scopes, so a few functions have a name with a trailing underscore: `request_()` for `request()`, `session_()` for `session()` and `cookie_()` for `cookie()`.

### Service Accessors

!!! warning "Not ported yet"
    The function `view_cell()` does not exist, because view cells are not ported.

#### cookie_()

`cookie_(name, value, options)` is a convenience function that creates a new `Cookie` instance. The name has a trailing underscore because `cookie` is a scope in CFML. `options` is a structure with the settings of the cookie:

```cfm
<cfset options = StructNew()>
<cfset options.expires = 3600>
<cfset rememberCookie = cookie_("remember_me", "yes", options)>
```

For more details, see [Cookies](../libraries/cookies.md).

#### cookies()

`cookies(cookies, getGlobal)` returns the `CookieStore` instance. `getGlobal` is `true` by default, and then the global cookie store held by the response is returned. With `false` it builds a new store with the cookies you pass in `cookies`:

```cfm
<cfset store = cookies()>
```

For more details, see [Cookies](../libraries/cookies.md).

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

Valid context values: `html` (the default), `js`, `css`, `url`, `attr` and `raw`.

You can pass the name of the character encoding as the third argument. An empty string means UTF-8. The `url` context uses it to encode the data. For the other contexts the encoding is only checked and kept.

```cfm
<cfoutput><a href="/search?q=#esc(q, "url")#">#esc(title)#</a></cfoutput>
```

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

#### response()

Returns the shared response object. This function is a wrapper for `service("response")`.

#### single_service()

`single_service(name, param, ...)` provides easy access to any of the [Services](../concepts/services.md) defined in the system. It works like `service()` but always returns a new instance of the class. It gives an empty string when no service has that name. See [single_service()](../concepts/services.md#single_service).

#### timer()

`timer(name, callable)` provides a quick method to benchmark parts of your application. Without a name, it returns the shared timer. With only a name, it starts the timer of that name, or stops it if it is already running. With a callable too, the call is recorded under the name, and the result of the call is returned. A callable is a list of two items, an object and a method name, or the string `"Class::method"`. It is called with no arguments:

```cfm
<cfset callable = ArrayNew(1)>
<cfset ArrayAppend(callable, reportService)>
<cfset ArrayAppend(callable, "build")>
<cfset result = timer("build_report", callable)>
```

The timer is the same as `service("timer")`. The Benchmarking page is not written yet.

#### view()

`view(name, data, options)` grabs the current renderer and tells it to render the specified view. It simply provides a convenience method that can be used in controllers and libraries. It returns the output of the view, and does not escape the data: that must be handled by the developer.

Currently, these options are read from the `options` structure:

- `saveData` specifies that data will persist between multiple calls to `view()` within the same request. If you do not want the data to be persisted, specify false.
- `cache` specifies the number of seconds to cache the view for. See [Caching Views](../outgoing/views.md#caching-views).
- `cache_name` specifies the cache ID to use. By default the name of the view is used.

```cfm
<cfset data = StructNew()>
<cfset data.title = "My Blog">
<cfset options = StructNew()>
<cfset options.saveData = true>
<cfset output = view("blog_template", data, options)>
```

For more details, see the [Views](../outgoing/views.md) and [View Renderer](../outgoing/view_renderer.md) pages.

### Miscellaneous Functions

#### app_timezone()

Returns the timezone the application has been set to display dates in. This is set in `app/Config/App.cfc`:

```cfm
<cfset timezone = app_timezone()>
```

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
    The function does not send the redirect itself. It throws `ColdIgniter.HTTP.Exceptions.RedirectException`, and ColdIgniter catches it and sends the redirect, so it also works when you call `force_https()` from a controller. To force HTTPS for the whole site, set `forceGlobalSecureRequests` to `true` in **app/Config/App.cfc**; the required `forcehttps` filter then sends the redirect. See [Controller Filters](../incoming/filters.md).

#### function_usable()

`function_usable(functionName)` returns `true` if a function exists and can be used. It always gives `false` for an empty name or a name that is not a function.

```cfm
<cfif function_usable("mycustomfunction")>...</cfif>
```

#### is_cli()

Returns `true` if the application is run through the command line and `false` if not. ColdIgniter is run by the web server only, so it gives `false` on every web request. It only gives `true` when both `REMOTE_ADDR` and `REQUEST_METHOD` are empty.

#### is_really_writable()

`is_really_writable(file)` checks whether a file or a folder can be written to. On Windows it write-tests, by creating a test file in a folder or opening a file for append. Elsewhere it uses `canWrite()` of the Java file object. It gives `true` or `false`.

```cfm
<cfif is_really_writable(request.WRITEPATH & "cache")>...</cfif>
```

#### is_windows()

`is_windows(mock)` returns `true` if the application is running on Windows, `false` otherwise. Pass `true` or `false` to mock the answer, for tests. The mock is kept for the rest of the request. Pass an empty string to clear it:

```cfm
<cfset is_windows(true)>
<cfset ignored = is_windows("")>
```

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

When you pass a [route name](../incoming/routing.md#named-routes) or `Controller::method`, `redirect("name")` finds the route with [reverse routing](../incoming/routing.md#reverse-routing). It is the same as `redirect().route("name")`:

```cfm
<cfreturn redirect("named_route")>
```

#### remove_invisible_characters()

`remove_invisible_characters(str, urlEncoded)` prevents inserting null characters between ASCII characters, like `Java\0script`. It returns the sanitized string. `urlEncoded` (default `true`) tells whether to remove URL-encoded characters as well.

```cfm
<cfset cleaned = remove_invisible_characters(input)>
```

#### request_()

Returns the shared request object. This function is a wrapper for `service("request")`.

#### route_to()

`route_to(method, param, ...)` generates a route for you based on a controller::method combination or a route name, taking parameters into effect if provided. It returns a route path (URI path relative to baseURL).

The first argument is either a `Controller::method` or the name of a route. Parameters for the route come next:

```cfm
<!--- the route is defined as: --->
<cfset routes.get("users/(:num)/gallery/(:num)", "Galleries::showUserGallery/$1/$2")>

<cfoutput><a href="#route_to("Galleries::showUserGallery", 15, 12)#">View Gallery</a></cfoutput>
<!--- result: /users/15/gallery/12 --->
```

With a route name:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["as"] = "user_gallery">
<cfset routes.get("users/(:num)/gallery/(:num)", "Galleries::showUserGallery/$1/$2", routeOptions)>

<cfoutput><a href="#route_to("user_gallery", 15, 12)#">View Gallery</a></cfoutput>
```

If the route has a `{locale}` placeholder, the locale can be given as the last parameter:

```cfm
<cfset routes.get("{locale}/users/(:num)", "Users::show/$1")>
<cfset path = route_to("Users::show", 15, "en")>
<!--- result: /en/users/15 --->
```

!!! important
    `route_to()` returns a route path, not a full URI. It is relative to the base URL. To get a full URI, use [`url_to()`](../helpers/url_helper.md#url_to) of the URL Helper.

See [Reverse Routing](../incoming/routing.md#reverse-routing) and [Named Routes](../incoming/routing.md#named-routes).

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
