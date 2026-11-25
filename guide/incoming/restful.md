# RESTful Resource Handling

Representational State Transfer (REST) is an architectural style for distributed applications, first described by Roy Fielding in his 2000 PhD dissertation, [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm). That may be a dry read, and you may find Martin Fowler's [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html) a gentler introduction.

REST has been interpreted, and misinterpreted, in more ways than most software architectures. The more of Roy Fielding's principles you use in an architecture, the more "RESTful" your application is considered.

ColdIgniter makes it easy to create RESTful APIs for your resources, with its resource routes and the `ResourceController`.

## Resource Routes

You can quickly create a handful of RESTful routes for one resource with the `resource()` method. It creates the routes that are needed for full CRUD of a resource: create a new resource, update an existing one, list all of the resource, show one, and delete one. The first argument is the resource name:

```cfm
<cfset routes.resource("photos")>

<!--- Equivalent to the following: --->
<cfset routes.get("photos/new", "Photos::new")>
<cfset routes.post("photos", "Photos::create")>
<cfset routes.get("photos", "Photos::index")>
<cfset routes.get("photos/(:any)", "Photos::show/$1")>
<cfset routes.get("photos/(:any)/edit", "Photos::edit/$1")>
<cfset routes.put("photos/(:any)", "Photos::update/$1")>
<cfset routes.patch("photos/(:any)", "Photos::update/$1")>
<cfset routes.delete("photos/(:any)", "Photos::delete/$1")>
```

!!! note
    The order above is for clarity. The real order in which the routes are created, in `RouteCollection`, makes sure the routes are resolved correctly.

!!! important
    Routes are matched in the order they are defined. If you have a resource `photos` above a `get` route for `photos/poll`, the route for the `show` action of the resource is matched first. To fix this, put the `get` line above the resource line so that it is matched first.

The second argument is a map of options that change the routes that are created. Make it with `StructNew()`. These routes are made for use as an API, where more HTTP methods are allowed. You can pass the `websafe` option to also create update and delete routes that work with HTML forms:

```cfm
<cfset options = StructNew()>
<cfset options["websafe"] = 1>
<cfset routes.resource("photos", options)>

<!--- The following routes are created as well: --->
<cfset routes.post("photos/(:any)/delete", "Photos::delete/$1")>
<cfset routes.post("photos/(:any)", "Photos::update/$1")>
```

### Change the Controller Used

You can choose the controller that is used by passing the `controller` option with the name of the controller:

```cfm
<cfset options = StructNew()>
<cfset options["controller"] = "Gallery">
<cfset routes.resource("photos", options)>

<!--- Would create routes like: --->
<cfset routes.get("photos", "App\Controllers\Gallery::index")>
```

If the name of the controller starts with a backslash, the default namespace is not added in front of it, and you give the full class name:

```cfm
<cfset options = StructNew()>
<cfset options["controller"] = "\App\Gallery">
<cfset routes.resource("photos", options)>

<!--- Would create routes like: --->
<cfset routes.get("photos", "\App\Gallery::index")>
```

See also [Controller's Namespace](routing.md).

### Change the Placeholder Used

By default the default placeholder of the route collection is used when a resource ID is needed, which is `(:any)` unless you change it with `setDefaultConstraint()`. You can change it for one resource by passing the `placeholder` option with the new string to use:

```cfm
<cfset options = StructNew()>
<cfset options["placeholder"] = "(:num)">
<cfset routes.resource("photos", options)>

<!--- Generates routes like: --->
<cfset routes.get("photos/(:num)", "Photos::show/$1")>
```

!!! note "Difference from CodeIgniter"
    The CodeIgniter guide says that `(:segment)` is the default. In both frameworks the code uses the default placeholder of the route collection, which is `(:any)`. This matches any text, including slashes, so use the `placeholder` option if you want the ID to be limited.

### Limit the Routes Made

You can limit the routes that are created with the `only` option. It is **an array** or **a comma separated list** of method names. Only the routes that match one of these methods are created. The rest are ignored:

```cfm
<cfset only = ArrayNew(1)>
<cfset ArrayAppend(only, "index")>
<cfset ArrayAppend(only, "show")>
<cfset options = StructNew()>
<cfset options["only"] = only>
<cfset routes.resource("photos", options)>
```

You can also remove routes you do not need with the `except` option. It is also **an array** or **a comma separated list** of method names. This option is applied after `only`:

```cfm
<cfset options = StructNew()>
<cfset options["except"] = "new,edit">
<cfset routes.resource("photos", options)>
```

The valid methods are `index`, `show`, `create`, `update`, `new`, `edit` and `delete`.

## ResourceController

The `ResourceController` is a good starting point for your RESTful API. It has methods that match the resource routes above.

Extend it, set the `modelName` and `format_` properties, and then write the methods you want to handle. Any method you do not write answers with the status code 501 (not implemented):

```cfm
<cfcomponent output="no" parent="ColdIgniter\RESTful\ResourceController">

	<cfset this.modelName = "App\Models\PhotoModel">
	<cfset this.format_ = "json">

	<cffunction name="index" access="public" returntype="any" output="no">
		<cfreturn respond(this.model.findAll())>
	</cffunction>

</cfcomponent>
```

!!! note "Difference from CodeIgniter"
    The property is called `format_`, with an underscore, because the controller has a method `format()`. If it is an empty string (null), the format is negotiated from the `Accept` header of the request, and the default is JSON. The model is created from `modelName` and stored as `this.model`.

The helper methods are `respond()`, `respondCreated()`, `respondUpdated()`, `respondDeleted()`, `respondNoContent()`, `fail()`, `failUnauthorized()`, `failForbidden()`, `failNotFound()`, `failValidationErrors()`, `failResourceExists()`, `failResourceGone()`, `failTooManyRequests()` and `failServerError()`. They are described in [API Response Trait](../outgoing/api_responses.md).

The methods are `index()`, `show(id)`, `new()`, `create()`, `edit(id)`, `update(id)` and `delete(id)`.

The routing for this would be:

```cfm
<cfset routes.resource("photos")>
```

## Presenter Routes

You can quickly create a presentation controller that goes with a resource controller, with the `presenter()` method. It creates routes for the controller methods that return views for your resource, or that process the forms sent from those views.

You do not need it, because the presentation can be handled by a normal controller. It is a convenience. Its use is like the resource routing:

```cfm
<cfset routes.presenter("photos")>

<!--- Equivalent to the following: --->
<cfset routes.get("photos/new", "Photos::new")>
<cfset routes.post("photos/create", "Photos::create")>
<cfset routes.post("photos", "Photos::create")>   <!--- alias --->
<cfset routes.get("photos", "Photos::index")>
<cfset routes.get("photos/show/(:any)", "Photos::show/$1")>
<cfset routes.get("photos/(:any)", "Photos::show/$1")>  <!--- alias --->
<cfset routes.get("photos/edit/(:any)", "Photos::edit/$1")>
<cfset routes.post("photos/update/(:any)", "Photos::update/$1")>
<cfset routes.get("photos/remove/(:any)", "Photos::remove/$1")>
<cfset routes.post("photos/delete/(:any)", "Photos::delete/$1")>
```

!!! note
    The order above is for clarity. The real order in which the routes are created, in `RouteCollection`, makes sure the routes are resolved correctly.

Do not make routes for `photos` for both a resource controller and a presenter controller. You must tell them apart, for example:

```cfm
<cfset routes.resource("api/photo")>
<cfset routes.presenter("admin/photos")>
```

The second argument is a map of options that change the routes that are created.

### Change the Controller Used

You can choose the controller that is used by passing the `controller` option with the name of the controller:

```cfm
<cfset options = StructNew()>
<cfset options["controller"] = "Gallery">
<cfset routes.presenter("photos", options)>

<!--- Would create routes like: --->
<cfset routes.get("photos", "App\Controllers\Gallery::index")>
```

A name that starts with a backslash is the full class name:

```cfm
<cfset options = StructNew()>
<cfset options["controller"] = "\App\Gallery">
<cfset routes.presenter("photos", options)>

<!--- Would create routes like: --->
<cfset routes.get("photos", "\App\Gallery::index")>
```

See also [Controller's Namespace](routing.md).

### Change the Placeholder Used

By default the default placeholder of the route collection is used when a resource ID is needed. You can change it by passing the `placeholder` option with the new string to use:

```cfm
<cfset options = StructNew()>
<cfset options["placeholder"] = "(:num)">
<cfset routes.presenter("photos", options)>

<!--- Generates routes like: --->
<cfset routes.get("photos/(:num)", "Photos::show/$1")>
```

### Limit the Routes Made

You can limit the routes that are created with the `only` option. It is **an array** or **a comma separated list** of method names. Only the routes that match one of these methods are created. The rest are ignored:

```cfm
<cfset only = ArrayNew(1)>
<cfset ArrayAppend(only, "index")>
<cfset ArrayAppend(only, "show")>
<cfset options = StructNew()>
<cfset options["only"] = only>
<cfset routes.presenter("photos", options)>
```

You can also remove routes you do not need with the `except` option. It is also **an array** or **a comma separated list** of method names. This option is applied after `only`:

```cfm
<cfset options = StructNew()>
<cfset options["except"] = "new,edit">
<cfset routes.presenter("photos", options)>
```

The valid methods are `index`, `show`, `new`, `create`, `edit`, `update`, `remove` and `delete`.

## ResourcePresenter

The `ResourcePresenter` is a good starting point for presenting views of your resource, and for processing the data of the forms in those views. It has methods that match the presenter routes above.

Extend it, set the `modelName` property, and then write the methods you want to handle. A method you do not write returns a "not implemented" text:

```cfm
<cfcomponent output="no" parent="ColdIgniter\RESTful\ResourcePresenter">

	<cfset this.modelName = "App\Models\PhotoModel">

	<cffunction name="index" access="public" returntype="any" output="no">
		<cfreturn view("photos/index")>
	</cffunction>

</cfcomponent>
```

The routing for this would be:

```cfm
<cfset routes.presenter("photos")>
```

## Presenter/Controller Comparison

This table compares the default routes made by `resource()` and `presenter()` with their methods.

| Operation | Method | Controller Route | Presenter Route | Controller Function | Presenter Function |
| --- | --- | --- | --- | --- | --- |
| **New** | GET | photos/new | photos/new | `new()` | `new()` |
| **Create** | POST | photos | photos | `create()` | `create()` |
| Create (alias) | POST | | photos/create | | `create()` |
| **List** | GET | photos | photos | `index()` | `index()` |
| **Show** | GET | photos/(:any) | photos/(:any) | `show(id)` | `show(id)` |
| Show (alias) | GET | | photos/show/(:any) | | `show(id)` |
| **Edit** | GET | photos/(:any)/edit | photos/edit/(:any) | `edit(id)` | `edit(id)` |
| **Update** | PUT/PATCH | photos/(:any) | | `update(id)` | |
| Update (websafe) | POST | photos/(:any) | photos/update/(:any) | `update(id)` | `update(id)` |
| **Remove** | GET | | photos/remove/(:any) | | `remove(id)` |
| **Delete** | DELETE | photos/(:any) | | `delete(id)` | |
| Delete (websafe) | POST | photos/(:any)/delete | photos/delete/(:any) | `delete(id)` | `delete(id)` |

The `websafe` rows for the controller exist only when you pass the `websafe` option.
