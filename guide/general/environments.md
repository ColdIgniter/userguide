# Handling Multiple Environments

Developers often desire different system behavior depending on whether an application is running in a development or production environment. For example, verbose error output is something that would be useful while developing an application, but it may also pose a security issue when "live". In development environments, you might want additional tools loaded that you don't in production environments, etc.

## The Defined Environments

By default, ColdIgniter has three environments defined.

- `production` for production
- `development` for development
- `testing` for testing

!!! important
    The environment `testing` is reserved for testing. It has special conditions built into the framework at various places to assist with that. You can't use it for your development.

If you want another environment, e.g., for staging, you can add custom environments. See [Adding Environments](#adding-environments).

## Setting Environment

### The ENVIRONMENT Constant

To set your environment, ColdIgniter comes with the `ENVIRONMENT` constant (`request.ENVIRONMENT`). If you set `CI_ENVIRONMENT`, the value will be used, otherwise it defaults to `production`.

#### .env

The method to set the variable is your **.env** file:

```ini
CI_ENVIRONMENT = development
```

!!! note "Difference from CodeIgniter"
    The value is read only from the **.env** file. A server variable, a web server setting or a request header does not set the environment. A request header can reach the `CGI` scope by name, so reading it would let a client choose the environment. There is no `spark env` command. Edit the file instead.

In addition to affecting some basic framework behavior (see below), you may use this constant in your own development to differentiate between which environment you are running in.

## Adding Environments

To add custom environments, you just need to add boot files for them.

### Boot Files

ColdIgniter requires that a CFML file matching the environment's name is located under **app/Config/Boot**. These files can contain any customizations that you would like to make for your environment, whether it's updating the error display settings, loading additional developer tools, or anything else. These are automatically loaded by the system. The following files are already created in a fresh install:

* development.cfm
* production.cfm
* testing.cfm

If there is no file for the environment, the framework answers with the status 503 and the text "The application environment is not set correctly."

For example, if you want to add a `staging` environment, all you need to do is:

1. copy **app/Config/Boot/production.cfm** to **staging.cfm**.
2. customize settings in **staging.cfm** if you want.

## Confirming the Current Environment

To confirm the current environment, simply output the constant `request.ENVIRONMENT`:

```cfm
<cfoutput>#request.ENVIRONMENT#</cfoutput>
```

## Effects on Default Framework Behavior

There are some places in the ColdIgniter system where the `ENVIRONMENT` constant is used. This section describes how default framework behavior is affected.

### Error Reporting

Setting the `ENVIRONMENT` constant to a value of `development` will cause detailed error reports to be rendered to the browser when errors occur. Conversely, setting the constant to `production` will disable all error output. Disabling error reporting in production is a [good security practice](../concepts/security.md). See [Error Handling](errors.md).
