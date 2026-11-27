# URI Routing

How to map an address to a controller and a method.

## What is URI Routing?

URI Routing connects a URI to the method of a controller.

ColdIgniter has two kinds of routing. One is **Defined Route Routing**, and the other is **Auto Routing**. With Defined Route Routing, you define the routes yourself, which allows flexible URLs. Auto Routing routes HTTP requests from conventions and runs the matching controller method, so there is no need to define routes by hand.

First, let's look at Defined Route Routing. If you want to use Auto Routing, see [Auto Routing (Improved)](auto_routing_improved.md).

!!! note "Difference from CodeIgniter"
    ColdIgniter has only Auto Routing (Improved). Auto Routing (Legacy) is not ported.

## Setting Routing Rules

Routing rules are in the file `app/Config/Routes.cfm`. The variable `routes` is the route collection. It lets you set your own routing rules. Routes can use placeholders or regular expressions.

When you set a route, you choose a method that matches an HTTP verb (the request method). If you expect a GET request, use `get()`:

```cfm
<cfset routes.get("/", "Home::index")>
```

A route takes the **route path** (the URI path relative to the base URL) on the left. It connects it to the **route handler** (the controller and the method, here `Home::index`) on the right, with any parameters that go to the controller.

Write the controller and the method with a double colon between them, like `Users::list`.

If the method needs parameters, list them after the method name, separated by slashes:

```cfm
<cfset routes.get("product/(:num)/(:num)", "Catalog::productLookup/$1/$2")>
```

### Examples

Here are a few basic routing examples.

A URL with the word **journals** in the first segment goes to the class `\App\Controllers\Blogs` and the [default method](#default-method), which is usually `index()`:

```cfm
<cfset routes.get("journals", "Blogs")>
```

A URL with the segments **blog/joe** goes to the class `\App\Controllers\Blogs` and the method `users()`. The ID is set to `34`:

```cfm
<cfset routes.get("blog/joe", "Blogs::users/34")>
```

A URL with **product** as the first segment and anything in the second goes to the class `\App\Controllers\Catalog` and the method `productLookup()`:

```cfm
<cfset routes.get("product/(:any)", "Catalog::productLookup")>
```

A URL with **product** as the first segment and a number in the second goes to the class `\App\Controllers\Catalog` and the method `productLookupByID()`. The match is passed to the method as an argument:

```cfm
<cfset routes.get("product/(:num)", "Catalog::productLookupByID/$1")>
```

### HTTP verb Routes

You can use these HTTP verbs: `get`, `post`, `put`, `delete`, `patch`, `head` and `options`:

```cfm
<cfset routes.get("products", "Product::feature")>
<cfset routes.post("products", "Product::feature")>
<cfset routes.put("products/1", "Product::feature")>
<cfset routes.delete("products/1", "Product::feature")>
<cfset routes.options("products", "Product::feature")>
<cfset routes.head("products", "Product::feature")>
<cfset routes.patch("products/1", "Product::feature")>
```

Each of these methods returns the route collection.

Matching several verbs with one call of `match()` is described in [Mapping Multiple Routes](#mapping-multiple-routes).

### Specifying Route Handlers

#### Controller's Namespace

When you write a controller and method as a string, and the controller does not start with a backslash, the [default namespace](#default-namespace) is put in front of it:

```cfm
<cfset routes.get("/", "Home::index")>
<!--- this is the same as --->
<cfset routes.get("/", "App\Controllers\Home::index")>
```

If you put a backslash at the start, the name is used as it is:

```cfm
<cfset routes.get("/", "\Admin\Home::index")>
```

You can also set the namespace with the `namespace` option:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["namespace"] = "Admin\Controllers">
<cfset routes.get("/", "Home::index", routeOptions)>
```

See [Assigning Namespace](#assigning-namespace) for details.

Controllers in a sub-folder of `app/Controllers/` are named with a backslash, for example `Api\Setup::index`.

!!! note "Difference from CodeIgniter"
    A handler with a slash in the controller name, such as `Admin/Home::index`, throws an error. The namespace delimiter is a backslash.

#### Array Callable Syntax

!!! warning "Not ported yet"
    A handler can only be a string. The array form `[Controller::class, "method"]` does not exist in CFML.

##### Array Callable Syntax and Placeholders

!!! warning "Not ported yet"
    A handler can only be a string. Use `$1`, `$2` and so on in the string for parameters.

#### Using Closures

!!! warning "Not ported yet"
    A handler cannot be a function. Write the code in a controller method.

### Specifying Route Paths

#### Placeholders

A typical route looks like this:

```cfm
<cfset routes.get("product/(:num)/(:alpha)", "Catalog::productLookup")>
```

The first argument is the URI to match. The second argument is where it goes. In the example, if the word "product" is in the first segment and a number is in the second, the class `Catalog` and the method `productLookup` are used.

Placeholders are strings that stand for a regular expression. While the route is made, they are replaced by the expression. They are mostly used to make routes easier to read.

These placeholders are available:

| Placeholder | Description |
|-------------|-------------|
| `(:any)` | Matches all characters from that point to the end of the URI. This can include several URI segments. |
| `(:segment)` | Matches any character except a slash (`/`), so it matches one segment only. |
| `(:num)` | Matches any positive integer. |
| `(:alpha)` | Matches any string of letters. |
| `(:alphanum)` | Matches any string of letters or digits, or both. |
| `(:hash)` | The same as `(:segment)`. Use it to see which routes use hashed IDs. |

`{locale}` matches one segment (any characters except a slash) and is also the way to detect the locale from the URI. When a route with `{locale}` matches, the matched segment is stored. The framework then sets it as the locale of the request (`setLocale_()`) before the controller runs. If `useSupportedLocalesOnly()` was switched on in the route collection, a locale that is not in `supportedLocales` of **app/Config/App.cfc** makes the router throw a `PageNotFoundException`. By default the flag is off. The helper `route_to()` and the reverse routing fill `{locale}` with the locale of the request, or with an extra last argument.

```cfm
<cfset routes.get("{locale}/books", "Books::index")>
```

!!! note "Difference from CodeIgniter"
    The locale is found by the number of the group, not by a named group, because Java 1.4.2 has no named groups. A `(` inside a character class such as `[(]`, before `{locale}` in the same route, is counted as a group and gives a wrong locale.

!!! warning "Not ported yet"
    `group()` is not ported, so a `{locale}` prefix for a group of routes cannot be written. Put `{locale}` in each route.

##### The Behavior of (:any)

A single `(:any)` matches several segments of the URL. For example, this route:

```cfm
<cfset routes.get("product/(:any)", "Catalog::productLookup/$1")>
```

matches **product/123**, **product/123/456**, **product/123/456/789** and so on.

If `$1` holds a slash, it is still split into several parameters when passed to `Catalog::productLookup()`. The method must be ready for the largest number of parameters. Give the extra parameters a default, so a shorter URL still works:

```cfm
<cffunction name="productLookup" access="public" returntype="string" output="no">
	<cfargument name="id1" type="any" required="no" default="">
	<cfargument name="id2" type="any" required="no" default="">
	<cfargument name="id3" type="any" required="no" default="">

	<!--- ... --->
	<cfreturn "">
</cffunction>
```

!!! important
    Do not put a placeholder after `(:any)`. The number of parameters that go to the method can change.

See [Multiple URI Segments as One Parameter](#multiple-uri-segments-as-one-parameter) for the option that passes several segments as one parameter.

If you do not want to match several segments, use `(:segment)`:

```cfm
<cfset routes.get("product/(:segment)", "Catalog::productLookup/$1")>
```

This only matches **product/123** and gives a 404 error for the other examples.

#### Custom Placeholders

!!! warning "Not ported yet"
    The method `addPlaceholder()` does not exist. Write the regular expression in the route instead.

#### Regular Expressions

You can use regular expressions in your routes, including back-references:

```cfm
<cfset routes.get("products/([a-z]+)/([0-9]+)", "Products::show/$1/$2")>
```

!!! important
    With back-references, use the dollar syntax (`$1`), not the double backslash syntax.

In the example, a URI like **products/shirts/123** calls the method `show()` of the controller `Products`, with the first and second segment as arguments.

A regular expression can also catch a segment that holds a slash. For example, a user opens a page that needs a login and you want to send them back to it afterwards:

```cfm
<cfset routes.get("login/(.+)", "Auth::login/$1")>
```

If `$1` holds a slash, it is still split into several parameters when passed to `Auth::login()`.

!!! note "Difference from CodeIgniter"
    The regular expressions are the ones of Java (`java.util.regex`), not PCRE. The route is matched against the whole URI. A dynamic controller, such as `$1::index`, throws an error for security reasons.

You can mix placeholders and regular expressions.

### View Routes

!!! warning "Not ported yet"
    The `view()` method of the route collection does not exist. Use a controller method that returns a view.

### Redirecting Routes

!!! warning "Not ported yet"
    The `addRedirect()` method does not exist.

### Environment Restrictions

!!! warning "Not ported yet"
    The `environment()` method of the route collection does not exist. Put the routes in `app/Config/Routes.local.cfm` for routes on your own machine only.

### Routes with any HTTP verbs

!!! warning
    It is better to use the routes of the HTTP verbs. CSRF protection does not protect GET requests. If the URI of an `add()` route can be reached with GET, CSRF protection does not work.

To make a route for any HTTP verb, use `add()`:

```cfm
<cfset routes.add("products", "Product::feature")>
```

The routes of the current verb are checked before the routes made with `add()`.

### Mapping Multiple Routes

!!! warning "Not ported yet"
    The `map()` and `match()` methods of the route collection do not exist. Call a verb method once for each route.

### Command-Line Only Routes

!!! warning "Not ported yet"
    ColdIgniter has no command line, so there are no `cli()` routes.

## Global Options

All the methods that create a route accept a struct of options as the last argument. Make it with `StructNew()`. The options change how the route works.

ColdIgniter reads the options `filter` and `namespace`. The other options of CodeIgniter are listed below as not ported.

### Applying Filters

You can change what a route does by giving it filters that run before or after the controller. This is useful for authentication or logging of an API.

The value of `filter` is a string or an array of strings. Each one is the alias of a filter defined in `app/Config/Filters.cfc`, or the class name of a filter. See [Controller Filters](filters.md) for how to define aliases.

!!! note
    A filter on a route is applied whenever the route matches. A controller reached by [Auto Routing (Improved)](auto_routing_improved.md) has no route, so set its filters in `app/Config/Filters.cfc`.

#### Alias Filter

Use an alias defined in `app/Config/Filters.cfc` as the value of `filter`:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = "auth">
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

You can pass arguments to the `before()` and `after()` methods of the filter. Write them after a colon, separated by commas:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = "role:admin,manager">
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

#### Classname Filter

You can use the class name of a filter as the value of `filter`:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = "App\Filters\AuthFilter">
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

#### Multiple Filters

You can use an array as the value of `filter`:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = ArrayNew(1)>
<cfset ArrayAppend(routeOptions["filter"], "auth")>
<cfset ArrayAppend(routeOptions["filter"], "throttle")>
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

##### Filter Arguments

You can also pass arguments to a filter in the array:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["filter"] = ArrayNew(1)>
<cfset ArrayAppend(routeOptions["filter"], "auth:dual,noreturn")>
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

In this example the list `dual`, `noreturn` is passed to the `before()` and `after()` methods of the filter, in the argument `arguments_`.

### Assigning Namespace

The [default namespace](#default-namespace) is put in front of the controllers. You can use a different namespace for one route with the `namespace` option. The value is the namespace you want:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["namespace"] = "Admin\Controllers">
<cfset routes.get("admin", "Admin::index", routeOptions)>
```

The new namespace is only used for that call.

### Limit to Hostname

!!! warning "Not ported yet"
    The `hostname` option is not read.

#### Restrict by Multiple Hostnames

!!! warning "Not ported yet"
    The `hostname` option is not read.

### Limit to Subdomains

!!! warning "Not ported yet"
    The `subdomain` option is not read.

### Offsetting the Matched Parameters

!!! warning "Not ported yet"
    The `offset` option is not read.

## Reverse Routing

Reverse routing lets you name the controller and method, and any parameters, that a link should go to, and have the router look up the route for it. This lets route definitions change without changes in your application code. It is typically used in views to make links.

For example, if you have a route to a photo gallery that you want to link to, use the `url_to()` function to get the route that should be used. The first argument is the fully qualified controller and method, separated by a double colon (`::`), as when you write the route itself. Any parameters that should be passed to the route come next:

```cfm
<!--- the route is defined as: --->
<cfset routes.get("users/(:num)/gallery/(:num)", "Galleries::showUserGallery/$1/$2")>

<!--- generate the URI to link to user ID 15, gallery 12: --->
<cfoutput><a href="#url_to("Galleries::showUserGallery", 15, 12)#">View Gallery</a></cfoutput>
<!--- result: http://example.com/users/15/gallery/12 --->
```

`route_to()` gives the same route as a path relative to the base URL. See [route_to()](../general/common_functions.md#route_to).

## Named Routes

You can name routes to make your application less fragile. A name is applied to a route and can be used later, so even if the route definition changes, all the links in your application made with `url_to()` still work with no change. A route is named with the `as` option:

```cfm
<cfset routeOptions = StructNew()>
<cfset routeOptions["as"] = "user_gallery">
<cfset routes.get("users/(:num)/gallery/(:num)", "Galleries::showUserGallery/$1/$2", routeOptions)>

<!--- generate the URI to link to user ID 15, gallery 12: --->
<cfoutput><a href="#url_to("user_gallery", 15, 12)#">View Gallery</a></cfoutput>
<!--- result: http://example.com/users/15/gallery/12 --->
```

This also makes the views easier to read.

!!! note
    By default, all defined routes have names that match their paths, with placeholders replaced by their regular expressions. For example, if you define the route `routes.get("edit/(:num)", "PostController::edit/$1")`, you can make the URL with `route_to("edit/([0-9]+)", 12)`.

!!! warning
    A route that is defined first without a name (for example `routes.get("edit", "PostController::edit")`) takes the name `edit`. If you then define a named route with the same name as that path (for example `routes.get("edit/(:num)", "PostController::edit/$1", routeOptions)` with `as` set to `edit`), the second route is not registered, because its name conflicts with the name given to the first.

## Grouping Routes

!!! warning "Not ported yet"
    The `group()` method of the route collection does not exist.

### Setting Namespace

!!! warning "Not ported yet"
    This part needs `group()`. Use the `namespace` option on each route.

### Setting Filters

!!! warning "Not ported yet"
    This part needs `group()`. Use the `filter` option on each route.

### Setting Other Options

!!! warning "Not ported yet"
    This part needs `group()`.

### Nesting Groups

!!! warning "Not ported yet"
    This part needs `group()`.

## Route Priority

!!! warning "Not ported yet"
    There is no route priority. Routes are checked in the order they are defined. The routes of the current verb come before the routes made with `add()`.

### Changing Route Priority

!!! warning "Not ported yet"
    The `priority` option is not read.

## Routes Configuration Options

The route collection has some options that affect all routes. They are in `app/Config/Routing.cfc`.

### Default Namespace

When a route names a controller, the router puts the default namespace in front of it. The default value is `App\Controllers`.

If you set the value to an empty string, each route must give the full name of the controller.

If you put your controllers in a namespace, you can change this value to save typing:

```cfm
<cfset this.defaultNamespace = "Admin\Controllers">
```

### Default Method

This setting is used when the route handler names a controller and no method. The default value is `index`:

```cfm
<cfset this.defaultMethod = "index">
```

If you define this route:

```cfm
<cfset routes.get("/", "Home")>
```

the method `index()` of the controller `App\Controllers\Home` runs when the route matches.

### Translate URI Dashes

This option replaces dashes (`-`) with underscores in the method name taken from the route. The controller name is not changed. A dash is not valid in a method name:

```cfm
<cfset this.translateURIDashes = true>
```

With Auto Routing (Improved), see [Translate URI To CamelCase](auto_routing_improved.md#translate-uri-to-camelcase), which takes the place of this option.

### Use Defined Routes Only

The `autoRoute` option in `app/Config/Routing.cfc` is `false` by default, so auto routing is off.

When no defined route matches the URI, the system tries to match the URI against the controllers and methods if Auto Routing is on. You can turn this matching off, and keep to the routes you define, by setting the `autoRoute` property to `false`:

```cfm
<cfset this.autoRoute = false>
```

!!! warning
    CSRF protection does not protect GET requests. If a URI can be reached with GET, CSRF protection does not work.

### 404 Override

When no route matches the current URI, the system shows a generic 404 page. With the `override404` setting in `app/Config/Routing.cfc` you can name a controller and a method to run instead:

```cfm
<cfset this.override404 = "App\Controllers\Errors::show404">
```

You can also set it in `app/Config/Routes.cfm` with the method `set404Override()`:

```cfm
<cfset routes.set404Override("App\Controllers\Errors::show404")>
```

Give the full class name. The default namespace is not added to this value. The method gets the message of the error as its argument. The status code of the response is set to `404`. To change it, see [Response](../outgoing/response.md).

!!! warning "Not ported yet"
    The value cannot be a function.

### Route Processing by Priority

!!! warning "Not ported yet"
    The `prioritize` option is read from `app/Config/Routing.cfc`, but it has no effect.

### Multiple URI Segments as One Parameter

!!! warning "Not ported yet"
    The option `multipleSegmentsOneParam` is in `app/Config/Routing.cfc`, but it has no effect. A placeholder that matches several segments is always split into several parameters.

## Auto Routing (Improved)

Auto Routing (Improved) is a more secure automatic routing system.

See [Auto Routing (Improved)](auto_routing_improved.md) for details.

## Auto Routing (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

### Enable Auto Routing (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

### URI Segments (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

### Configuration Options (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

#### Default Controller (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

##### For Site Root URI (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

##### For Directory URI (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

#### Default Method (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

## Confirming Routes

CodeIgniter lists the routes with the command `spark routes`. ColdIgniter has no command line. To list the routes in code, use the route collection:

```cfm
<cfset routes = service("routes")>
<cfset list = routes.getRoutes("GET")>
```

`getRoutes()` returns a struct. The keys are the route paths (as regular expressions) and the values are the handlers.

### spark routes

!!! warning "Not ported yet"
    The command `spark routes` needs the command line tool `spark`, which ColdIgniter does not have.

#### Auto Routing (Improved)

!!! warning "Not ported yet"
    The output of `spark routes` for Auto Routing (Improved) is not ported, because ColdIgniter has no command line tool.

#### Auto Routing (Legacy)

!!! warning "Not ported yet"
    Auto Routing (Legacy) is not ported. ColdIgniter has only [Auto Routing (Improved)](auto_routing_improved.md).

#### Sort by Handler

!!! warning "Not ported yet"
    This is an option of `spark routes`, which ColdIgniter does not have.

#### Specify Host

!!! warning "Not ported yet"
    This is an option of `spark routes`, which ColdIgniter does not have.

## Getting Routing Information

It is often useful to know which controller and method handle the current request, and which filters are active for the route. This helps with logging, debugging or conditional logic.

### Retrieving the Current Controller/Method Names

The router has the names of the controller and the method that handle the current request:

```cfm
<cfset router = service("router")>
<cfset controllerName = router.controllerName()>
<cfset methodName = router.methodName()>
```

`controllerName()` returns the full name of the controller, with its namespace.

### Getting Active Filters for the Current Route

[Filters](filters.md) run actions before or after the request is handled. To get the filters of the current route, use `getFilters()` of the router:

```cfm
<cfset router = service("router")>
<cfset filters = router.getFilters()>
```

It returns an array of the filters that were set for the route with the `filter` option. It does not include the global filters or the ones set in `app/Config/Filters.cfc`.

### Getting Matched Route Options for the Current Route

The options of the matched route are returned by `getMatchedRouteOptions()`:

```cfm
<cfset router = service("router")>
<cfset options = router.getMatchedRouteOptions()>
```
