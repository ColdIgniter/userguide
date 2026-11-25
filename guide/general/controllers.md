# Controllers

A controller is a component in `app/Controllers/`. A route names the controller and the method that answer a request.

## A first controller

The starter application has one controller, `app/Controllers/Home.cfc`:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="index" access="public" returntype="string" output="no">
		<cfreturn view("welcome_message")>
	</cffunction>

</cfcomponent>
```

The method returns the text of the answer. Here it is the view `app/Views/welcome_message.cfm`.

## The route

Routes are in `app/Config/Routes.cfm`. This line sends the home page to the method `index` of the controller `Home`:

```cfm
<cfset routes.get("/", "Home::index")>
```

## Routes for your own machine

Routes that you only want on your own machine go in `app/Config/Routes.local.cfm`. This file is not part of the repository.
