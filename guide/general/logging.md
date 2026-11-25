# Logging Information

## Log Levels

You can log information to the local log files by using the `log_message()` function. You must supply the "level" of the error in the first parameter, indicating what type of message it is (debug, error, etc). The second parameter is the message itself:

```cfm
<cfif NOT some_function()>
    <cfset log_message("error", "Some variable did not contain a value.")>
</cfif>
```

There are eight different log levels, matching to the [RFC 5424](https://tools.ietf.org/html/rfc5424) levels, and they are as follows:

| Level     | Description                                                                                                                                                                         |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| debug     | Detailed debug information.                                                                                                                                                         |
| info      | Interesting events in your application, like a user logging in, logging SQL queries, etc.                                                                                          |
| notice    | Normal, but significant events in your application.                                                                                                                                 |
| warning   | Exceptional occurrences that are not errors, like the use of deprecated APIs, poor use of an API, or other undesirable things that are not necessarily wrong.                       |
| error     | Runtime errors that do not require immediate action but should typically be logged and monitored.                                                                                   |
| critical  | Critical conditions, like an application component not available, or an unexpected exception.                                                                                       |
| alert     | Action must be taken immediately, like when an entire website is down, the database unavailable, etc.                                                                               |
| emergency | The system is unusable.                                                                                                                                                             |

The logging system does not provide ways to alert sysadmins or webmasters about these events, they solely log the information. For many of the more critical event levels, the logging happens automatically by the Error Handler, described in [Error Handling](errors.md).

## Configuration

You can modify which levels are actually logged, as well as assign different Loggers to handle different levels, within the **app/Config/Logger.cfc** configuration file.

The `threshold` value of the config file determines which levels are logged across your application. If any levels are requested to be logged by the application, but the threshold does not allow them to log currently, they will be ignored. The simplest method to use is to set this value to the minimum level that you want to have logged. For example, if you want to log warning messages, and not information messages, you would set the threshold to `5`. Any log requests with a level of 5 or less (which includes runtime errors, system errors, etc) would be logged and info, notices, and debug would be ignored:

```cfm
<cfset this.threshold = 5>
```

A complete list of levels and their corresponding threshold value is in the configuration file for your reference. The default is `4` in the `production` environment and `9` (all messages) in the others.

You can pick and choose the specific levels that you would like logged by assigning an array of log level numbers to the threshold value:

```cfm
<cfset this.threshold = ListToArray("1,2,3,8")>
```

### Using Multiple Log Handlers

The logging system can support multiple methods of handling logging running at the same time. Each handler can be set to handle specific levels and ignore the rest. Two handlers come with a default install:

- **File Handler** is the default handler and will create a single file for every day locally, named **log-YYYY-MM-DD.log** in the folder **writable/logs**. This is the recommended method of logging.
- **Errorlog Handler** This handler writes the logs with the `cflog` tag of ColdFusion MX 6.1. It is not active by default: it is commented out in **app/Config/Logger.cfc**.

!!! warning "Not ported yet"
    The ChromeLogger handler is not ported.

The handlers are configured in the main configuration file, in the `handlers` property, which is simply a map of handlers and their configuration. Each handler is specified with the key being the fully name-spaced class name. The value will be a map of varying properties, specific to each handler. Each handler's section will have one property in common: `handles`, which is an array of log level *names* that the handler will log information for. See **app/Config/Logger.cfc** for the entry of the File Handler.

!!! note "Difference from CodeIgniter"
    The File Handler refuses a file extension that the web server would run as a page (such as `cfm`, `cfml` or `cfc`), and any extension that is not made of letters and digits. In that case it writes nothing. The file permissions of a new log file are not ported.

## Modifying the Message with Context

You will often want to modify the details of your message based on the context of the event being logged. You might need to log a user id, an IP address, the current POST variables, etc. You can do this by use placeholders in your message. Each placeholder must be wrapped in curly braces. In the third parameter, you must provide a structure of placeholder names (without the braces) and their values. These will be inserted into the message string:

```cfm
<!--- Generates a message like: User 123 logged into the system from 192.0.2.10 --->
<cfset context = StructNew()>
<cfset context["user"] = 123>
<cfset context["ip_address"] = request_().getIPAddress()>
<cfset log_message("info", "User {user} logged into the system from {ip_address}", context)>
```

If you want to log an error that you caught, you can use the key of `exception`, and the value being the `cfcatch` structure itself. A string will be generated from it containing the error message, the file name and line number. You must still provide the exception placeholder in the message:

```cfm
<cftry>
    <cfthrow type="Application" message="Something went wrong.">

    <cfcatch type="any">
        <cfset context = StructNew()>
        <cfset context["exception"] = cfcatch>
        <cfset log_message("critical", "{exception}", context)>
    </cfcatch>
</cftry>
```

Several core placeholders exist that will be automatically expanded for you based on the current page request:

| Placeholder    | Inserted value                                 |
| -------------- | ---------------------------------------------- |
| {post_vars}    | The form variables                             |
| {get_vars}     | The URL variables                              |
| {env}          | Current environment name, i.e., development    |
| {file}         | The name of file calling the logger            |
| {line}         | The line in {file} where the logger was called |
| {env:foo}      | The value of 'foo' in the environment          |

!!! warning "Not ported yet"
    The placeholder `{session_vars}` is not ported. The placeholders `{file}` and `{line}` are found from the trace of an error that is thrown and caught on the spot, because ColdFusion MX 6.1 has no function that gives the caller.

## Using Third-Party Loggers

!!! warning "Not ported yet"
    PSR-3 and its interface do not exist in ColdIgniter. Any call that is done through the `log_message()` function uses the service `logger`, and calls its function `log_(level, message, context)`. To use a logger of your own, define a `logger` function in **app/Config/Services.cfc**, which extends the services of the framework, and make sure the autoloader can find your component. See [Services](../concepts/services.md).
