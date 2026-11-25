# Error Handling

ColdIgniter builds error reporting into your system through errors that are thrown with `cfthrow` and caught with `cftry` and `cfcatch`, both those of ColdFusion MX 6.1 and a few that are provided by the framework.

Depending on your environment's setup, the default action when an error is thrown is to display a detailed error report unless the application is running under the `production` environment. In the `production` environment, a more generic message is displayed to keep the best user experience for your users.

!!! note "Difference from CodeIgniter"
    In ColdFusion MX 6.1 an exception is not an object of a class. It is an error with a `type` (such as `ColdIgniter.Exceptions.PageNotFoundException`), a `message`, an `errorcode` and a `tagContext` (the template and line). The `cfcatch` variable holds them. A framework "exception class" is a component whose functions throw the error, called through `static_()`.

## Using Exceptions

This section is a quick overview for newer programmers, or for developers who are not experienced with using exceptions.

### What are Exceptions

Exceptions are simply events that happen when the exception is "thrown". This halts the current flow of the template, and execution is then sent to the error handler which displays the appropriate error page:

```cfm
<cfthrow type="Application" message="Some message goes here">
```

### Catching Exceptions

If you are calling a function that might throw an exception, you can catch that exception using a `cftry` block:

```cfm
<cftry>
    <cfset userModel = new_("App\Models\UserModel")>
    <cfset userModel.find(id)>

    <cfcatch type="any">
        <cfreturn "Caught: " & cfcatch.message>
    </cfcatch>
</cftry>
```

If the model throws an exception, it is caught and the code within the `cfcatch` block is executed. In this example, the function returns the error message that the model defined.

#### Catching Specific Exceptions

In the example above, we catch any type of exception. If we only want to watch for specific types of exceptions, like a `ColdIgniter.Database.Exceptions.DataException`, we can specify that in the `type` of the `cfcatch`. Any other exceptions that are thrown and are not of the caught type will be passed on to the error handler:

```cfm
<cftry>
    <cfset userModel = new_("App\Models\UserModel")>
    <cfset userModel.find(id)>

    <cfcatch type="ColdIgniter.Database.Exceptions.DataException">
        <cfreturn "Invalid data: " & cfcatch.message>
    </cfcatch>
</cftry>
```

This can be handy for handling the error yourself, or for performing cleanup before the template ends. If you want the error handler to function as normal, you can throw the error again within the `cfcatch` block:

```cfm
<cftry>
    <cfset userModel = new_("App\Models\UserModel")>
    <cfset userModel.find(id)>

    <cfcatch type="ColdIgniter.Database.Exceptions.DataException">
        <!--- cleanup here, then pass the error on to the error handler --->
        <cfrethrow>
    </cfcatch>
</cftry>
```

## Configuration

### Error Reporting

When the setting `display_errors` is on, ColdIgniter will display a detailed error report with all errors. The setting is made by the boot file of the environment (**app/Config/Boot/development.cfm** and **app/Config/Boot/testing.cfm** turn it on, **app/Config/Boot/production.cfm** turns it off).

So by default, ColdIgniter will display a detailed error report in the `development` and `testing` environments, and will not display any errors in the `production` environment.

You can change your environment by setting the `CI_ENVIRONMENT` variable. See [Handling Multiple Environments](environments.md).

!!! note "Difference from CodeIgniter"
    `display_errors` is not a setting of ColdFusion MX 6.1. It is a value that the boot files keep for the length of the request, with `ini_set()`.

!!! important
    Disabling error reporting DOES NOT stop logs from being written if there are errors.

!!! warning
    If the detailed error report is displayed, it can show data of the request and of the server. Never run a site that the public can reach in the `development` environment.

### Logging Exceptions

By default, all exceptions other than "404 - Page Not Found" exceptions are logged. This can be turned on and off by setting the `log` value of **app/Config/Exceptions.cfc**:

```cfm
<cfset this.log = true>
```

To ignore logging on other status codes, you can set the status code to ignore in the same file:

```cfm
<cfset this.ignoreCodes = ArrayNew(1)>
<cfset ArrayAppend(this.ignoreCodes, 404)>
<cfset ArrayAppend(this.ignoreCodes, 403)>
```

!!! note
    It is possible that logging still will not happen for exceptions if your current [Log settings](logging.md#configuration) are not set up to log `critical` errors, which all exceptions are logged as.

### Logging Deprecation Warnings

!!! warning "Not ported yet"
    ColdFusion MX 6.1 has no deprecation errors and no error levels, so the logging of deprecation warnings (the settings `logDeprecations` and `deprecationLogLevel` of the Exceptions config of CodeIgniter) has no effect, and the environment variable `CODEIGNITER_SCREAM_DEPRECATIONS` is not read.

#### Configuration

!!! warning "Not ported yet"
    See above: there is nothing to configure.

## Framework Exceptions

### Exception Design

An exception is not an object in CFML, so the exception classes of CodeIgniter are components that only keep the same names. A framework error is thrown with `cfthrow` and its own `type`, which starts with `ColdIgniter.` (for example `ColdIgniter.Exceptions.ConfigException`).

!!! note "Difference from CodeIgniter"
    In CodeIgniter the framework exceptions extend the base classes of the language and implement `ExceptionInterface`, so one `catch` of the interface finds all of them. In ColdIgniter the type of each error is its own, and `ExceptionInterface` (**system/Exceptions/ExceptionInterface.cfc**) is an empty base component that nothing extends. Catch the exact type: `<cfcatch type="ColdIgniter.Exceptions.ConfigException">`.

#### LogicException

`LogicException` represents an error in the program logic. Throw it with the type `ColdIgniter.Exceptions.LogicException`:

```cfm
<cfthrow type="ColdIgniter.Exceptions.LogicException" message="The value cannot be used here.">
```

#### RuntimeException

`RuntimeException` is for an error that can only be found at run time. Throw it with the type `ColdIgniter.Exceptions.RuntimeException`:

```cfm
<cfthrow type="ColdIgniter.Exceptions.RuntimeException" message="The file could not be read.">
```

The following framework exceptions are also available:

#### PageNotFoundException

This is used to signal a 404, Page Not Found error:

```cfm
<cfset static_("ColdIgniter\Exceptions\PageNotFoundException").forPageNotFound()>
```

You can pass a message into the function that will be displayed in place of the default message on the 404 page:

```cfm
<cfset static_("ColdIgniter\Exceptions\PageNotFoundException").forPageNotFound("The page you asked for is gone.")>
```

For the default 404 view file location, see [HTTP Status Code and Error Views](#http-status-code-and-error-views).

#### ConfigException

This exception is used when the values from the configuration class are invalid, or when the config class is not the right type, etc. It is thrown with the type `ColdIgniter.Exceptions.ConfigException`, and its `getExitCode()` returns 3.

!!! warning "Not ported yet"
    Of the functions of `ConfigException` in CodeIgniter only `forDisabledMigrations()` exists. To throw the error for another case, use `cfthrow` with the type `ColdIgniter.Exceptions.ConfigException`.

#### DatabaseException

This exception is thrown for database errors, such as when the database connection cannot be created, or when it is temporarily lost:

```cfm
<cfthrow type="ColdIgniter.Database.Exceptions.DatabaseException" message="Unable to connect to the database.">
```

Its `getExitCode()` returns 8.

#### RedirectException

This exception is used to redirect from anywhere in your code. In CodeIgniter you write `throw new RedirectException($response)`. In ColdIgniter the function `raise()` does the same: give it a redirect response:

```cfm
<cfset static_("ColdIgniter\HTTP\Exceptions\RedirectException").raise(redirect().to("login"))>
```

ColdIgniter catches the error, takes the response from it and sends it, so the request ends there. Code that catches the type itself (`ColdIgniter.HTTP.Exceptions.RedirectException`) gets the object with `lastThrown()` and calls `getResponse()`.

!!! note "Difference from CodeIgniter"
    The type is its own and is not found by a catch of a base type, as explained in Exception Design above.

## Specify HTTP Status Code in Your Exception

In CodeIgniter an exception that implements `HTTPExceptionInterface` has its code used as the HTTP status code. In ColdIgniter the error handler takes the HTTP status code from the `errorcode` of the thrown error for these types: `ColdIgniter.Exceptions.PageNotFoundException`, `ColdIgniter.HTTP.Exceptions.BadRequestException`, `ColdIgniter.HTTP.Exceptions.RedirectException`, `ColdIgniter.Security.Exceptions.SecurityException` and `ColdIgniter.Honeypot.Exceptions.HoneypotException`. Any other error gives the status code 500.

!!! note "Difference from CodeIgniter"
    `HTTPExceptionInterface` (**system/Exceptions/HTTPExceptionInterface.cfc**) is an empty base component that nothing extends, so you cannot add your own type to the list by implementing it. The types that count are the ones named above. For another status code, set it in your own [exception handler](#defining-the-new-handler).

## HTTP Status Code and Error Views

The exception handler displays the error view corresponding to the HTTP status code, if one exists.

For example, a `PageNotFoundException` has the code `404`, so it will be the HTTP status code. Therefore if it is thrown, the system will show the **error_404.cfm** in the **app/Views/errors/html** folder.

If there is no view file corresponding to the HTTP status code, **production.cfm** or **error_exception.cfm** will be displayed.

!!! note
    If `display_errors` is on, **error_exception.cfm** is selected and a detailed error report is displayed.

You should customize all of the error views in the **app/Views/errors/html** folder for your site.

You can also create error views for specific HTTP status code. For example, if you want to create an error view for "400 Bad Request", add **error_400.cfm**.

!!! warning
    If an error view file with the corresponding HTTP status code exists, the exception handler will display that file regardless of the environment. The view file must be implemented in such a way that it does not display detailed error messages in production environment by yourself.

!!! note "Difference from CodeIgniter"
    There is no command line, so there is no **app/Views/errors/cli** folder. A request is always answered with the HTML view, also when the client asked for JSON or XML.

## Specify Exit Code in Your Exception

In CodeIgniter an exception that implements `HasExitCodeInterface` gives the exit code of the command. The exception handler of ColdIgniter reads the exit code of the types that have one (`ConfigException`, `FilterException`, `HoneypotException`, `CastException` and `DatabaseException`) through their `getExitCode()`, and gives it to the handler as the argument `exitCode`.

!!! note "Difference from CodeIgniter"
    A web request has no exit code, so nothing ends with it. The status code of the response is what the client sees. `HasExitCodeInterface` is a base component that only throws, and the list of types is kept in the exception handler.

## Custom Exception Handlers

If you need more control over how exceptions are displayed you can define your own handlers and specify when they apply.

### Defining the New Handler

The first step is to create a new component which extends `ColdIgniter\Debug\BaseExceptionHandler` (with the `parent` attribute). The new handler must define a single function: `handle()`. It gets the error, the request, the response, the HTTP status code and the exit code:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Debug\BaseExceptionHandler">

	<cffunction name="handle" access="public" returntype="void" output="no">
		<cfargument name="exception" type="any" required="yes">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="response" type="any" required="yes">
		<cfargument name="statusCode" type="numeric" required="yes">
		<cfargument name="exitCode" type="numeric" required="yes">

		<cfheader statuscode="#arguments.statusCode#">
		<cfset arguments.response.setStatusCode(arguments.statusCode)>
		<cfset request.ColdIgniter.output = "Something went wrong: " & esc(arguments.exception.message)>
	</cffunction>

</cfcomponent>
```

The built-in handler `ColdIgniter\Debug\ExceptionHandler` shows what a handler does: it sets the status code of the response and the headers, picks a view and puts the rendered page in `request.ColdIgniter.output`.

`ColdIgniter\Debug\ExceptionHandlerInterface` is a base component with the one function `handle()`, which only throws. A handler defines it. Nothing in the built-in handler answers in JSON or XML.

!!! note "Difference from CodeIgniter"
    There are no interfaces on ColdFusion MX 6.1, so a handler extends `BaseExceptionHandler` and does not implement the interface.

### Configuring the New Handler

Telling ColdIgniter to use your new exception handler component is done in the **app/Config/Exceptions.cfc** configuration file's `handler()` function. It gets the status code and the error. The status code is 500, or the `errorcode` of the types listed above, so test the `type` of the error to pick a handler for a case of your own:

```cfm
<cffunction name="handler" access="public" returntype="any" output="no">
	<cfargument name="statusCode" type="numeric" required="yes">
	<cfargument name="exception" type="any" required="yes">

	<cfif arguments.exception.type EQ "ColdIgniter.Database.Exceptions.DatabaseException">
		<cfreturn new_("App\Libraries\DatabaseErrorHandler", this)>
	</cfif>

	<cfreturn new_("ColdIgniter\Debug\ExceptionHandler", this)>
</cffunction>
```

You can use any logic your application needs to determine whether it should handle the exception, but the most common is checking the type of the error (`arguments.exception.type`). If your component should handle it then return a new instance of it, with the config object as the argument.
