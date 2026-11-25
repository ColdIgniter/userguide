# Services

## Introduction

### What are Services?

The **Services** in ColdIgniter provide the functionality to create and share new class instances. They are implemented as the component `Config\Services` (the file **app/Config/Services.cfc**), which extends the framework's `ColdIgniter\Config\Services` (**system/Config/Services.cfc**).

All of the core classes within ColdIgniter are provided as "services". This simply means that, instead of hard-coding a class name to load, the classes to call are defined within a very simple configuration file. This file acts as a type of factory to create new instances of the required class.

### Why use Services?

A quick example will probably make things clearer, so imagine that you need to pull in an instance of the Timer class. The simplest method would simply be to create a new instance of that class:

```cfm
<cfset timer = new_("ColdIgniter\Debug\Timer")>
```

And this works great. Until you decide that you want to use a different timer class in its place. Maybe this one has some advanced reporting the default timer does not provide. In order to do this, you now have to locate all of the locations in your application that you have used the timer class. Since you might have left them in place to keep a performance log of your application constantly running, this might be a time-consuming and error-prone way to handle this. That's where services come in handy.

Instead of creating the instance ourself, we let a central component create an instance of the class for us. This component is kept very simple. It only contains a function for each class that we want to use as a service. The function typically returns a **shared instance** of that class, passing any dependencies it might have into it. Then, we would replace our timer creation code with code that calls this global function or the Services component:

```cfm
<cfset timer = service("timer")>

<!--- the same, written in full --->
<cfset timer = static_("Config\Services").timer()>
```

When you need to change the implementation used, you can modify the services configuration file, and the change happens automatically throughout your application without you having to do anything. Now you just need to take advantage of any new functionality and you're good to go. Very simple and error-resistant.

!!! note
    It is recommended to only create services within controllers. Other files, like models and libraries, should have the dependencies either passed into the constructor or through a setter function.

## How to Get a Service

As many ColdIgniter classes are provided as services, you can get them like the following:

```cfm
<cfset timer = service("timer")>
```

The `timer` is an instance of the Timer class, and if you call `service("timer")` again, you will get exactly the same instance.

The Services typically return a **shared instance** of the class. A second call with other parameters returns the same instance, and the new parameters are ignored.

The services that exist are: `autoloader`, `cache`, `coldigniter`, `csp`, `encrypter`, `exceptions`, `filters`, `format`, `honeypot`, `incomingrequest`, `language`, `locator`, `logger`, `migrations`, `negotiator`, `pager`, `parser`, `redirectresponse`, `renderer`, `request`, `response`, `responsecache`, `router`, `routes`, `security`, `session`, `siteurifactory`, `superglobals`, `throttler`, `timer`, `toolbar`, `uri` and `validation`.

!!! warning "Not ported yet"
    Every service of CodeIgniter that is not in that list is not ported yet. The file `Config/Services.php` of CodeIgniter holds the full list (for example `curlrequest`, `email`, `image`, `typography` and `clirequest`).

!!! note "Difference from CodeIgniter"
    A service name that does not exist gives an empty string. Service names are not case sensitive.

### Getting a New Instance

If you want to get a new instance of the Timer class, you need to pass `false` to the argument `getShared`:

```cfm
<cfset timer = static_("Config\Services").timer(false)>
```

### Convenience Functions

#### service()

`service()` returns the shared instance of the requested service. The only required parameter is the service name. This is the same as the function name within the Services component, and always returns a SHARED instance of the class, so calling the function multiple times returns the same instance:

```cfm
<cfset logger = service("logger")>
```

If the creation function requires additional parameters, they can be passed after the service name:

```cfm
<cfset renderer = service("renderer", "/path/to/views")>
```

#### single_service()

The second function, `single_service()`, works just like `service()` but returns a new instance of the class:

```cfm
<cfset logger = single_service("logger")>
```

Like `service()`, it takes the service name and then the parameters of the creation function. It gives an empty string when no service has that name.

!!! note "Difference from CodeIgniter"
    CFML has no reflection, and a call can skip arguments by name. The parameters you give are put under the names of the parameters of the creation function (read from the component metadata), the `getShared` flag is set to `false`, and the defaults of the function fill the rest.

## Defining Services

To make services work well, you have to be able to rely on each class having a constant API to use. When you want to extend or replace core classes, you only need to keep the functions the framework calls.

!!! note "Difference from CodeIgniter"
    ColdFusion MX 6.1 has no interfaces. A replacement class extends the class it replaces, or provides the same functions.

For example, to replace the `RouteCollection` class, create a component with the new behavior:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Router\RouteCollection">
    <!--- your changes here --->
</cfcomponent>
```

Finally, add the `routes()` function to **app/Config/Services.cfc** to create a new instance of it instead of `ColdIgniter\Router\RouteCollection`:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Config\Services" hint="Services Configuration file">

    <cffunction name="routes" access="public" returntype="any" output="no">
        <cfargument name="getShared" type="boolean" required="no" default="true">

        <cfif arguments.getShared>
            <cfreturn getSharedInstance("routes")>
        </cfif>

        <cfreturn new_("App\Libraries\MyRouteCollection")>
    </cffunction>

</cfcomponent>
```

The function in your file replaces the function of the same name in the framework's file. Take the arguments of the original function from **system/Config/Services.cfc**.

### Allowing Parameters

In some instances, you will want the option to pass a setting to the class during instantiation. Since the services file is a very simple component, it is easy to make this work.

A good example is the `renderer` service. By default, we want this class to be able to find the views at the view directory of **app/Config/Paths.cfc**. We want the developer to have the option of changing that path, though, if their needs require it. So the class accepts the view path as a constructor parameter. The service function looks like this (shortened):

```cfm
<cffunction name="renderer" access="public" returntype="any" output="no">
    <cfargument name="viewPath" type="string" required="no" default="">
    <cfargument name="config" type="any" required="no" default="">
    <cfargument name="getShared" type="boolean" required="no" default="true">

    <cfif arguments.getShared>
        <cfreturn getSharedInstance("renderer", arguments.viewPath, arguments.config)>
    </cfif>

    <cfif arguments.viewPath EQ "" OR arguments.viewPath EQ "0">
        <cfset arguments.viewPath = new_("Config\Paths").viewDirectory>
    </cfif>
    <cfif is_null(arguments.config)>
        <cfset arguments.config = variables.config("Config\View")>
    </cfif>

    <cfreturn new_("ColdIgniter\View\View", arguments.config, arguments.viewPath, "", request.CI_DEBUG, static_("Config\Services").get("logger"))>
</cffunction>
```

This sets the default path in the constructor, but allows for easily changing the path it uses:

```cfm
<cfset renderer = static_("Config\Services").renderer("/path/to/views", "", false)>
```

!!! note "Difference from CodeIgniter"
    An argument that is left out has an empty string as its value, and the framework treats an empty string as "no value" (`is_null()`).

### Shared Classes

There are occasions where you need to require that only a single instance of a service is created. This is easily handled with the `getSharedInstance()` function that is called from within the factory function. This handles checking if an instance has been created and saved within the component, and, if not, creates a new one. All of the factory functions provide a `getShared = true` value as the last parameter. You should stick to the function also:

```cfm
<cffunction name="example" access="public" returntype="any" output="no">
    <cfargument name="getShared" type="boolean" required="no" default="true">

    <cfif arguments.getShared>
        <cfreturn getSharedInstance("example")>
    </cfif>

    <cfreturn new_("App\Libraries\Example")>
</cffunction>
```

## Service Discovery

!!! warning "Not ported yet"
    ColdIgniter does not search the namespaces for **Config/Services.cfc** files of modules. Only **app/Config/Services.cfc** is used. Put the services of a module in that file.

### Resetting Services Cache

When the Services component is first called, early in the start of the framework, the service classes found by discovery are kept in a property, and the property is not updated. If modules are loaded later and they have services, the cache has to be updated.

This is done with `static_("Config\Services").resetServicesCache()`. It clears the cache and makes discovery run again when it is needed:

```cfm
<cfset static_("Config\Services").resetServicesCache()>
```

!!! note "Difference from CodeIgniter"
    Discovery in modules is not ported (see [Service Discovery](#service-discovery)), so the cache holds no module services. The function only clears the list and the `discovered` flag.

### Other Functions of Services

These functions of the Services component are not on the CodeIgniter page of this name. They exist in CodeIgniter too, and are useful in tests:

- `get(key)` returns a service that was already created, or creates it. `has(key)` tells whether a shared instance exists.
- `set(key, value)` stores a shared instance, and throws `ColdIgniter.Exceptions.InvalidArgumentException` if there is one already. `override(key, value)` stores it in any case.
- `injectMock(name, mock)` makes the object the shared instance of a service, and keeps it as a mock.
- `reset(initAutoloader)` throws away all shared instances and mocks. `resetSingle(name)` does it for one service.
- `serviceExists(name)` gives the name of the class that has the service, or an empty string.

Call them on `static_("Config\Services")`:

```cfm
<cfset static_("Config\Services").injectMock("timer", mockTimer)>
<cfset static_("Config\Services").resetSingle("timer")>
```
