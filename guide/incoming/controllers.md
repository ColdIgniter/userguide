# Controllers

Controllers are the heart of your application. They decide how an HTTP request is handled.

## What is a Controller?

A controller is a component file that handles an HTTP request. [URI Routing](routing.md) connects a URI to a controller and a method. The method returns a view string or a `Response` object.

Controllers are in `app/Controllers/`. The starter application has one, `app/Controllers/Home.cfc`:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="index" access="public" returntype="string" output="no">
		<cfreturn view("welcome_message")>
	</cffunction>

</cfcomponent>
```

Every controller you create should extend `BaseController`. This component is in `app/Controllers/BaseController.cfc` and gives all your controllers the features below.

This line in `app/Config/Routes.cfm` sends the home page to the method `index` of the controller `Home`:

```cfm
<cfset routes.get("/", "Home::index")>
```

Routes that you only want on your own machine go in `app/Config/Routes.local.cfm`. This file is not part of the repository.

## Constructor

A controller has a special method, `initController()`. The framework calls it after the controller is created. It receives the request, the response and the logger.

If you override `initController()` in `BaseController`, keep the call to the parent method:

```cfm
<cffunction name="initController" access="public" returntype="void" output="no">
	<cfargument name="request_" type="any" required="yes">
	<cfargument name="response" type="any" required="yes">
	<cfargument name="logger" type="any" required="yes">

	<cfset parent.initController(arguments.request_, arguments.response, arguments.logger)>

	<!--- load your own things here --->
</cffunction>
```

!!! important
    You cannot return a response from `initController()`. A redirect returned from it does not work.

## Included Properties

The controller has these properties. `initController()` sets the first three.

### Request Object

The main [request](incomingrequest.md) of the application is always available as `this.request_`.

!!! note "Difference from CodeIgniter"
    The name ends with an underscore because `request` is a name that CFML already uses.

```cfm
<cfset name = this.request_.getPost("name")>
```

### Response Object

The main [response](../outgoing/response.md) of the application is always available as `this.response`.

### Logger Object

The [logger](../general/logging.md) is available as `this.logger`.

### Helpers

You can list helper files in the property `this.helpers`. When the controller is loaded, these helpers are loaded too, and you can use their functions anywhere in the controller:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cfset this.helpers = ArrayNew(1)>
	<cfset ArrayAppend(this.helpers, "url")>
	<cfset ArrayAppend(this.helpers, "form")>

</cfcomponent>
```

If you set `this.helpers` in `BaseController`, set it before the call to `parent.initController()`, because the helpers are loaded there.

## forceHTTPS

!!! warning "Not ported yet"
    The `forceHTTPS()` method does not exist yet.

## Validating Data

### validateData()

The controller has the method `validateData()` to check data. It takes four arguments: the data to check (a struct), the rules (a struct), custom error messages (optional) and a database group (optional).

The [Validation](../libraries/validation.md) page describes the formats of rules and messages, and lists the rules.

```cfm
<cffunction name="product" access="public" returntype="string" output="no">
	<cfargument name="id" type="any" required="no" default="">
	<cfset var data = StructNew()>
	<cfset var rule = StructNew()>
	<cfset var info = StructNew()>

	<cfset data["id"] = arguments.id>
	<cfset data["name"] = this.request_.getPost("name")>

	<cfset rule["id"] = "integer">
	<cfset rule["name"] = "required|max_length[255]">

	<cfif NOT validateData(data, rule)>
		<cfset info["errors"] = this.validator.getErrors()>
		<cfreturn view("store/product", info)>
	</cfif>

	<!--- the data is valid --->
	<cfreturn "">
</cffunction>
```

### validate()

!!! important
    This method exists only for compatibility. Use `validateData()` in new code.

The controller also has `validate()`. It takes the rules and, optionally, custom messages. It reads the data to check from `this.request_`.

!!! warning
    `validate()` reads its data with `getVar()`, which reads the values in the order of the setting `request_order` (default `GP`, the form and URL values), or the JSON body. An attacker could change what is checked. Use `validateData()` to check form data only.

Use `this.validator.getValidated()` to get the validated data after `validate()`.

Instead of the rules, you can pass the name of a group defined in `app/Config/Validation.cfc`. The group is a property of that component, for example `this.signup`. If the property does not exist, an error is thrown:

```cfm
<cfif NOT validate("signup")>
	<cfset info = StructNew()>
	<cfset info["errors"] = this.validator.getErrors()>
	<cfreturn view("signup", info)>
</cfif>
```

Validation can also be done in the model. Where you do it is up to you.

## Protecting Methods

Some methods should not be reachable from the web. Declare them with `access="private"`. A request for a private method gives a "page not found" error. A method whose name starts with an underscore (except `__invoke`) also gives one.

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="utility" access="private" returntype="void" output="no">
		<!--- some code --->
	</cffunction>

</cfcomponent>
```

Even if you define a route to `Helloworld::utility`, the URI shows a "page not found" error.

## Auto Routing (Improved)

!!! warning "Not ported yet"
    ColdIgniter has no automatic routing. Every URI needs a route.

## Auto Routing (Legacy)

!!! warning "Not ported yet"
    ColdIgniter has no automatic routing. Every URI needs a route.

### Let's try it: Hello World! (Legacy)

!!! warning "Not ported yet"
    This part needs automatic routing. Define a route in `app/Config/Routes.cfm` instead.

### Methods (Legacy)

!!! warning "Not ported yet"
    This part needs automatic routing.

### Passing URI Segments to Your Methods (Legacy)

!!! warning "Not ported yet"
    Without automatic routing, segments reach your method through route placeholders. See [URI Routing](routing.md).

### Default Controller (Legacy)

!!! warning "Not ported yet"
    This part needs automatic routing.

#### Defining a Default Controller (Legacy)

!!! warning "Not ported yet"
    This part needs automatic routing.

### Organizing Your Controllers into Sub-directories (Legacy)

!!! warning "Not ported yet"
    This part needs automatic routing.

## Remapping Method Calls

!!! warning "Not ported yet"
    The `_remap` method is not called. A route must name the method.

## Extending the Controller

To extend the controller, see [Extending the BaseController](../extending/basecontroller.md).

## That's it!

That is all there is to know about controllers.
