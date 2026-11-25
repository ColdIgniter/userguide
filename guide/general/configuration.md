# Configuration

Every framework uses configuration files to define numerous parameters and initial settings. ColdIgniter configuration files define simple components where the required settings are public properties, set with `<cfset this.name = value>`.

Unlike many other frameworks, ColdIgniter configurable items are not contained in a single file. Instead, each class that needs configurable items has a configuration file with the same name as the class that uses it. You will find the application configuration files in the **app/Config** folder.

## What are Configuration Classes?

Configuration classes are used to define system default configuration values. Configuration values are typically *static*. Configuration classes are meant to hold the settings that configure how the application operates, rather than responding to each user's individual settings.

It is not recommended to change values set during the creation of a configuration object later during execution. In other words, treat configuration objects as read only.

Configuration values can be hard-coded in the component files or obtained from environment variables when the object is created.

## Working with Configuration Files

### Getting a Config Object

You can get the configuration object of a class in several ways.

#### new keyword

ColdFusion MX 6.1 has no `new` keyword. Create an instance with `new_()`:

```cfm
<cfset appConfig = new_("Config\App")>
```

!!! note "Difference from CodeIgniter"
    The configuration components of the framework are in the `Config` namespace, and a class name is a text such as `Config\App`. The autoloader maps that name to the file **app/Config/App.cfc**. See [Autoloading Files](../concepts/autoloader.md).

#### config()

By using the `config()` function:

```cfm
<cfset appConfig = config("Config\App")>
<cfset appConfig = config("App")>
```

`config()` returns the same object each time you ask for the same class (the objects are kept by [Factories](../concepts/factories.md)). Pass `false` as the second argument to get a new object. A name without a namespace is looked for in the `Config` namespace, so both lines above return the same object.

### Getting a Config Property

All configuration object properties are public, so you access the settings like any other property:

```cfm
<cfset appConfig = config("Config\App")>
<cfset baseURL = appConfig.baseURL>
```

## Creating Configuration Files

When you need a new configuration, first you create a new file in the folder **app/Config**.

The component must extend `ColdIgniter\Config\BaseConfig` (with the `parent` attribute) to ensure that it can receive environment-specific settings.

Define the component and fill it with public properties that represent your settings:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Config\BaseConfig">
	<cfset this.siteName = "My Site">
	<cfset this.siteEmail = "webmaster@example.com">
</cfcomponent>
```

Saved as **app/Config/CompanyProject.cfc**, it is used as `config("CompanyProject")`.

!!! note "Difference from CodeIgniter"
    CodeIgniter lets you put configuration files in the **Config** folder of any namespace. In ColdIgniter, the files that `config()` finds by a short name are in **app/Config**. Another namespace needs its full class name, and that class must be known to the autoloader.

## Environment Variables

One of today's best practices for application setup is to use Environment Variables. One reason for this is that Environment Variables are easy to change between deploys without changing any code. Configuration can change a lot across deploys, but code does not. For instance, multiple environments, such as the developer's local machine and the production server, usually need different configuration values for each particular setup.

Environment Variables should also be used for anything private such as passwords, API keys, or other sensitive data.

### Dotenv File

ColdIgniter makes it simple to set Environment Variables by using a "dotenv" file. The term comes from the file name, which starts with a dot before the text "env".

#### Creating Dotenv File

ColdIgniter expects the **.env** file to be at the root of your project alongside the **app** folder (or in the folder named by `envDirectory` in **app/Config/Paths.cfc**). There is a template file distributed with ColdIgniter at the project root named **env** (notice there is no dot at the start).

It has a collection of variables your project might use, all commented out. You can use this file as a starting place for your application by either renaming the template to **.env**, or by making a copy of it named **.env**.

!!! warning
    Make sure the **.env** file is NOT tracked by your version control system. For *git* that means adding it to **.gitignore**. Failure to do so could result in sensitive credentials being exposed to the public. Never put it in a folder that the web server serves.

#### Setting Variables

Settings are stored in **.env** files as a simple collection of name/value pairs separated by an equal sign.

```ini
S3_BUCKET = dotenv
SECRET_KEY = super_secret_key
CI_ENVIRONMENT = development
```

When your application runs, **.env** is loaded automatically at the start of each request. A value that is already set is not overwritten, unless it is empty or `0`. A value with spaces must be put in quotes, or the file gives an error. A line that starts with `#` is a comment, and so is the text after a space and `#` that follows an unquoted value.

#### Getting Variables

The loaded environment variables are read with the function `env()`:

```cfm
<cfset s3_bucket = env("S3_BUCKET")>
<cfset s3_bucket = env("S3_BUCKET", "default_value")>
```

The second argument is the value that is returned when the variable is not set. The words `true` and `false` become booleans, and the words `empty` and `null` become an empty string.

!!! note "Difference from CodeIgniter"
    There is no process environment that ColdIgniter could write to, and no `getenv()`, `$_SERVER` or `$_ENV`. The values of the **.env** file live in one structure for the length of the request, so a value is never visible outside of it. `env()` looks there first and then at the variables of the web server (the CGI variables such as `SERVER_NAME`). Names are not case-sensitive.

    A variable that is set in the environment of the server process is not read.

#### Nesting Variables

To save on typing, you can reuse variables that you have already specified in the file by wrapping the variable name within `${...}`:

```ini
BASE_DIR = "/var/webroot/project-root"
CACHE_DIR = "${BASE_DIR}/cache"
TMP_DIR = "${BASE_DIR}/tmp"
```

#### Namespaced Variables

There will be times when you will have several variables with the same name. The system needs a way of knowing what the correct setting should be. This problem is solved by "*namespacing*" the variables.

Namespaced variables use a dot notation to qualify variable names so they will be unique when incorporated into the environment. This is done by including a distinguishing prefix followed by a dot (.), and then the variable name itself.

```ini
# not namespaced variables
name = "George"
db = my_db

# namespaced variables
address.city = "Berlin"
address.country = "Germany"
frontend.db = sales
backend.db = admin
```

#### Namespace Separator

Some environments do not permit variable names with dots. In such a case, you can also use underscores (`_`) as a separator.

```ini
# namespaced variables with underscore
app_forceGlobalSecureRequests = true
app_CSPEnabled = true
```

## Configuration Classes and Environment Variables

When you create a configuration object, any *namespaced* environment variables are considered for merging into the configuration object's properties.

!!! important
    You cannot add a new property by setting environment variables, nor change a scalar value to an array. See [Environment Variables as Replacements for Data](#environment-variables-as-replacements-for-data).

!!! note
    This feature is implemented in the `ColdIgniter\Config\BaseConfig` component. So it will not work with the few files in the **app/Config** folder that do not extend it, such as **Autoload.cfc**, **Modules.cfc**, **Paths.cfc** and **Services.cfc**.

If the prefix of a namespaced variable exactly matches the namespace of the configuration class, then the trailing part of the setting (after the dot) is treated as a configuration property. If it matches an existing configuration property, the environment variable's value replaces the corresponding value from the configuration file. If there is no match, the configuration object is left unchanged.

```ini
Config\App.forceGlobalSecureRequests = true
Config\App.CSPEnabled = true
```

The same holds for a *short prefix*, which is a namespace using only the lowercase version of the configuration class name. If the short prefix matches the class name, the value from **.env** replaces the configuration file value.

```ini
app.forceGlobalSecureRequests = true
app.CSPEnabled = true
```

You can also write with underscores:

```ini
app_forceGlobalSecureRequests = true
app_CSPEnabled = true
```

!!! note "Difference from CodeIgniter"
    Prefixes and property names are not case-sensitive in ColdIgniter. The lowercase words `true` and `false` become booleans. When the property in the configuration file is a number, the value is turned into a number.

### Environment Variables as Replacements for Data

It is very important to always remember that environment variables contained in your **.env** are **only replacements for existing scalar values**.

Simply put, you can change only the property's scalar value that exists in the configuration component by setting it in your **.env**.

1. You cannot add a property that is not defined in the configuration component.
2. You cannot change a scalar value in a property to an array.
3. You cannot add an element to an existing array.

For example, you cannot just put `app.myNewConfig = foo` in your **.env** and expect `Config\App` to have that property and value at run time.

### Treating Environment Variables as Arrays

A namespaced environment variable can be further treated as an array (a structure or an array of the component). If the prefix matches the configuration class, then the remainder of the environment variable name is treated as a reference into the property if it also contains a dot.

```ini
# regular namespaced variable
Config\SimpleConfig.name = George

# array namespaced variables
Config\SimpleConfig.address.city = "Berlin"
Config\SimpleConfig.address.country = "Germany"
```

If this was referring to a `SimpleConfig` configuration object with a property `address` that is a structure with the keys `city` and `country`, the example above sets `name` to `George`, `address.city` to `Berlin` and `address.country` to `Germany`:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Config\BaseConfig">
	<cfset this.name = "">
	<cfset this.address = StructNew()>
	<cfset this.address["city"] = "">
	<cfset this.address["country"] = "">
</cfcomponent>
```

Any other elements of the `address` property are unchanged. An element of an array property is named by its number, starting at 0, such as `Config\SimpleConfig.list.0 = first`.

## Handling Different Environments

Configuring multiple environments is easily accomplished by using a separate **.env** file with values modified to meet that environment's needs.

The file should not contain every possible setting for every configuration class used by the application. In truth, it should include only those items that are specific to the environment or are sensitive details like passwords and API keys and other information that should not be exposed. But anything that changes between deployments is fair game.

In each environment, place the **.env** file in the project's root folder. For most setups, this will be the same level as the **app** folder.

Do not track **.env** files with your version control system. If you do, and the repository is made public, you will have put sensitive information where everybody can find it.

## Registrars

!!! warning "Not ported yet"
    Registrars (the `registrars` property of the base config class of CodeIgniter, and the registrar file of a namespace) are not ported. Implicit and explicit registrars do not change any configuration object.

### Implicit Registrars

!!! warning "Not ported yet"
    A **Config/Registrar.cfc** file in a namespace is not read.

### Explicit Registrars

!!! warning "Not ported yet"
    The `registrars` property of a configuration component is not read.

## Confirming Config Values

The actual configuration object property values are changed at runtime by [Environment Variables](#configuration-classes-and-environment-variables).

### config:check

!!! warning "Not ported yet"
    ColdIgniter has no command line, so the command `config:check` does not exist.
