# Controller Filters

Controller Filters let you run actions before or after the controllers run. Unlike [events](../extending/events.md), you can choose the URIs or routes the filters apply to. Before filters can change the request. After filters can act on the response, and even change it. This gives you a lot of flexibility.

Some common tasks for filters are:

- Protecting incoming requests against CSRF
- Restricting areas of your site by role
- Rate limiting for some endpoints
- Showing a "Down for Maintenance" page
- Automatic content negotiation
- and more

## Creating a Filter

A filter is a component that extends `ColdIgniter\Filters\FilterInterface`. It has two methods, `before()` and `after()`, which hold the code that runs before and after the controller. Your filter must define both methods, but they can be empty if you do not need them. If one is missing, a `FilterException` is thrown. A skeleton filter in `app/Filters/MyFilter.cfc` looks like this:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Filters\FilterInterface">

	<cffunction name="before" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">

		<!--- do something here --->
		<cfreturn "">
	</cffunction>

	<cffunction name="after" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="response" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">

		<!--- do something here --->
		<cfreturn "">
	</cffunction>

</cfcomponent>
```

!!! note "Difference from CodeIgniter"
    The arguments are named `request_` and `arguments_`, with an underscore, because `request` and `arguments` are reserved names in CFML. `arguments_` is an empty string (null) when the filter has no arguments, or a list of strings when it has.

### Before Filters

#### Replacing Request

From any filter you can return the request object. It replaces the current request, so changes you make stay in place when the controller runs.

#### Stopping Later Filters

When you have a series of filters, you may want to stop the later filters after one of them. You do this by returning **any non-empty** string. If a before filter returns an empty result, the controller and the later filters still run.

An exception to the non-empty rule is the request object. Returning it does not stop anything. It only replaces the current request.

!!! note "Difference from CodeIgniter"
    A before filter that returns a string stops the later filters, but the controller still runs. A boolean value, or the text `true` or `false`, counts as empty.

#### Returning Response

Before filters run before your controller, so you may want to stop the controller from running. This is usually done to redirect, as in this example:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Filters\FilterInterface">

	<cffunction name="before" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">

		<cfif is_null(session_("userId"))>
			<cfreturn redirect().to("login")>
		</cfif>

		<cfreturn "">
	</cffunction>

	<cffunction name="after" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="response" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">

		<cfreturn "">
	</cffunction>

</cfcomponent>
```

If a response is returned, it is sent to the client and the request ends. This is useful for rate limiting an API. See the [Throttler](../libraries/throttler.md) for an example.

### After Filters

After filters are almost the same as before filters, except that you can only return the response object, and you cannot stop the request. This lets you change the final output, or just do something with it. You could use it to be sure certain security headers are set, to cache the final output, or to filter the output with a bad words filter.

## Configuring Filters

There are two ways to set when filters run. One is in `app/Config/Filters.cfc`. The other is in `app/Config/Routes.cfm`. To apply filters to defined routes, use the routes file and see [URI Routing](routing.md).

!!! note
    The safest way to apply filters is to apply them to routes. ColdIgniter has no auto routing, so only the routes you define can be reached.

### app/Config/Filters.cfc

The file `app/Config/Filters.cfc` has these properties, which set when the filters run.

!!! warning
    Always add `*` at the end of a URI pattern in the filter settings, because a controller method may be reachable by more URLs than you think.

!!! note "Difference from CodeIgniter"
    The arrays of the original are maps and lists. In this file a map is made with `CreateObject("java", "java.util.LinkedHashMap").init()` and a list with `ArrayNew(1)`, as in the file that comes with the application. Keep the case of the keys: the aliases are looked up with the exact case.

#### aliases

The `aliases` map gives a simple name to one filter class, or to a list of filter classes:

```cfm
<cfset this.aliases = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.aliases["csrf"] = "ColdIgniter\Filters\CSRF">
```

Aliases are required. If you use a name in the settings that is not an alias, an error is thrown. Class names are not accepted here. They work only in the `filter` option of a route (see [URI Routing](routing.md)). Using aliases makes it easy to swap the class that is used. This is useful if you change your authentication system, because you only change the class of the filter.

You can join several filters into one alias, so a complex set of filters is easy to apply:

```cfm
<cfset apiPrep = ArrayNew(1)>
<cfset ArrayAppend(apiPrep, "App\Filters\Negotiate")>
<cfset ArrayAppend(apiPrep, "App\Filters\ApiAuth")>
<cfset this.aliases["api-prep"] = apiPrep>
```

Define as many aliases as you need.

#### required

The `required` section holds the **required filters**. They are special filters that run on every request. They run before and after the other kinds of filters described below.

!!! note
    The required filters always run. If the route does not exist, only the before filters run.

Think about how many you use here, since many filters on every request can slow the application. The filters set by default give functions of the framework. If you remove them, those functions stop working. See [Provided Filters](#provided-filters).

Add an alias to the `before` list or to the `after` list:

```cfm
<cfset this.required = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.required["before"] = ArrayNew(1)>
<cfset ArrayAppend(this.required["before"], "forcehttps")> <!--- Force Global Secure Requests --->
<cfset ArrayAppend(this.required["before"], "pagecache")> <!--- Web Page Caching --->
<cfset this.required["after"] = ArrayNew(1)>
<cfset ArrayAppend(this.required["after"], "pagecache")> <!--- Web Page Caching --->
<cfset ArrayAppend(this.required["after"], "toolbar")> <!--- DebugToolbar --->
```

#### globals

The `globals` section holds the filters that run on every valid request. Think about how many you use here, for the same reason as above.

Add an alias to the `before` list or to the `after` list:

```cfm
<cfset this.globals = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.globals["before"] = ArrayNew(1)>
<cfset ArrayAppend(this.globals["before"], "csrf")>
<cfset this.globals["after"] = ArrayNew(1)>
```

##### Except for a Few URIs

Sometimes you want a filter on almost every request, but not on a few. A common case is the CSRF filter: you may need to let third-party sites call one or two URIs, and keep the rest protected.

For this, make `before` (or `after`) a map. Each key is an alias. The value is a map with the key `except`, and a URI path (relative to the base URL) to match:

```cfm
<cfset this.globals["before"] = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset csrfOptions = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset csrfOptions["except"] = "api/*">
<cfset this.globals["before"]["csrf"] = csrfOptions>
```

Anywhere you can use a URI path in the filter settings, you can use an asterisk (`*`) as a wildcard that matches all characters after it. The match is not case sensitive. In this example, any URI path that starts with `api/` is exempt from CSRF protection, and the forms of the site stay protected.

If you need several URI paths, use a list of patterns:

```cfm
<cfset except = ArrayNew(1)>
<cfset ArrayAppend(except, "foo/*")>
<cfset ArrayAppend(except, "bar/*")>
<cfset csrfOptions["except"] = except>
```

#### methods

!!! warning
    ColdIgniter has no auto routing, so only the routes you define can be reached.

You can apply filters to all requests of one HTTP method, like `POST`, `GET` or `PUT`. The key is the method in uppercase. The value is a list of aliases to run:

```cfm
<cfset this.methods = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset postFilters = ArrayNew(1)>
<cfset ArrayAppend(postFilters, "invalidchars")>
<cfset ArrayAppend(postFilters, "csrf")>
<cfset this.methods["POST"] = postFilters>
```

!!! note
    Unlike `globals` and `filters`, these only run as before filters.

!!! warning "Not ported yet"
    The special method `CLI` has no use here, because ColdIgniter only answers web requests.

#### filters

This map holds filter aliases. For each alias you give a map with a `before` list and an `after` list. Each holds the URI path patterns the filter applies to:

```cfm
<cfset this.filters = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset fooUris = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset fooBefore = ArrayNew(1)>
<cfset ArrayAppend(fooBefore, "admin/*")>
<cfset fooUris["before"] = fooBefore>
<cfset fooAfter = ArrayNew(1)>
<cfset ArrayAppend(fooAfter, "users/*")>
<cfset fooUris["after"] = fooAfter>
<cfset this.filters["foo"] = fooUris>
```

##### Filter Arguments

You can pass arguments to a filter when you set it in `filters`. Write them after the alias, with a colon, and separate them with commas:

```cfm
<cfset adminUris = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset adminBefore = ArrayNew(1)>
<cfset ArrayAppend(adminBefore, "admin/*")>
<cfset adminUris["before"] = adminBefore>
<cfset this.filters["group:admin,superadmin"] = adminUris>
```

In this example, when the URI matches `admin/*`, the list `admin,superadmin` is passed as `arguments_` to the `before()` method of the `group` filter. Each argument is a string.

### Filter Execution Order

Filters run in this order:

- **Before filters**: required, globals, methods, filters, route
- **After filters**: route, filters, globals, required

!!! note "Difference from CodeIgniter"
    ColdIgniter always uses this order. The setting `oldFilterOrder` is not ported.

## Confirming Filters

!!! warning "Not ported yet"
    The command `filter:check` of CodeIgniter needs the command line tool `spark`, which ColdIgniter does not have. To see the filters of the current route, see [Getting Active Filters for the Current Route](routing.md).

## Provided Filters

The filters that come with ColdIgniter are:

- `cors`: see [CORS](../libraries/cors.md)
- `csrf`: see [Security](../libraries/security.md)
- `toolbar`: the debug toolbar
- `honeypot`: see [Honeypot](../libraries/honeypot.md)
- `invalidchars`
- `secureheaders`
- `forcehttps`
- `pagecache`: see [Caching](../general/caching.md)

!!! note
    The filters run in the order they have in the config file. The debug toolbar filter is meant to run last, so it can see everything the other filters do.

!!! warning "Not ported yet"
    The `performance` filter (PerformanceMetrics) is not an alias in ColdIgniter yet.

### ForceHTTPS

This filter gives the "Force Global Secure Requests" feature. If you set `forceGlobalSecureRequests` to true in `app/Config/App.cfc`, every request to the application must use a secure connection (HTTPS). If the request is not secure, the user is redirected to the secure version of the page, and the HTTP Strict Transport Security (HSTS) header is set.

### PerformanceMetrics

!!! warning "Not ported yet"
    The PerformanceMetrics filter is not ported, so the pseudo-variables `{elapsed_time}` and `{memory_usage}` are not provided by a filter.

### InvalidChars

This filter stops user input (GET, POST, cookies and the request body) that holds any of these:

- invalid UTF-8 characters
- control characters, except the line break and the tab

### SecureHeaders

This filter adds HTTP response headers that make your application more secure.

To change the headers, make your own component that extends `ColdIgniter\Filters\SecureHeaders` and set its `headers` map. Then point the `secureheaders` alias in `app/Config/Filters.cfc` at your component:

```cfm
<cfset this.aliases["secureheaders"] = "App\Filters\SecureHeaders">
```

To learn about secure headers, see the [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/).
