# Code Modules

ColdIgniter supports a form of code modularization to help you create reusable code. Modules are typically centered around a specific subject, and can be thought of as mini-applications within your larger application.

Any of the standard file types within the framework are supported, like controllers, models, views, config files, helpers, language files, etc. Modules may contain as few, or as many, of these as you like.

## Namespaces

The core element of the modules functionality comes from the [autoloader](../concepts/autoloader.md) that ColdIgniter uses. While any code can use the autoloader and namespaces, the primary way to take full advantage of modules is to namespace your code and add it to **app/Config/Autoload.cfc**, in the `psr4` property.

For example, let's say we want to keep a simple blog module that we can re-use between applications. We might create a folder with our company name, Acme, to store all of our modules within. We will put it right alongside our **app** directory in the main project root:

```text
acme/        (new modules directory)
app/
public/
system/
tests/
writable/
```

Open **app/Config/Autoload.cfc** and add the `Acme\Blog` namespace to the `psr4` property:

```cfm
<cfset this.psr4["Acme\Blog"] = request.ROOTPATH & "acme/Blog">
```

Now that this is set up, we can access any file within the **acme/Blog** folder through the `Acme\Blog` namespace. A common directory structure within a module will mimic the main application folder:

```text
acme/
    Blog/
        Config/
        Controllers/
        Database/
            Migrations/
            Seeds/
        Helpers/
        Language/
            en/
        Libraries/
        Models/
        Views/
```

Of course, there is nothing forcing you to use this exact structure, and you should organize it in the manner that best suits your module, leaving out directories you don't need, creating new directories for Entities, Interfaces, or Repositories, etc.

## Autoloading Non-class Files

Often your module will contain not only components but also other files, like procedural functions, bootstrapping files, module constants files, etc. which are not normally loaded the way components are loaded.

ColdIgniter can autoload these *non-class* files. All we need to do is provide the list of paths to those files and include them in the `files` property of your **app/Config/Autoload.cfc** file:

```cfm
<cfset ArrayAppend(this.files, request.ROOTPATH & "acme/Blog/Helpers/blog_functions.cfm")>
```

## Auto-Discovery

!!! warning "Not ported yet"
    Auto-discovery is not ported. The framework does not scan the namespaces for events, filters, registrars, route files or services. The file **app/Config/Modules.cfc** and its settings exist, but nothing reads them yet, and there is no Composer on ColdFusion MX 6.1. Name what a module adds in your own configuration, as shown in the sections below.

### Enable/Disable Discover

!!! warning "Not ported yet"
    See Auto-Discovery above.

### Specify Discovery Items

!!! warning "Not ported yet"
    See Auto-Discovery above.

### Discovery and Composer

!!! warning "Not ported yet"
    See Auto-Discovery above. There is no Composer.

#### Specify Composer Packages

!!! warning "Not ported yet"
    See Auto-Discovery above. There is no Composer.

#### Disable Composer Package Discovery

!!! warning "Not ported yet"
    See Auto-Discovery above. There is no Composer.

## Working with Files

This section will take a look at each of the file types (controllers, views, language files, etc) and how they can be used within the module.

### Routes

A route file of a module is not found by itself. Add its path to the `routeFiles` property of **app/Config/Routing.cfc**, next to the one of the application:

```cfm
<cfset ArrayAppend(this.routeFiles, request.ROOTPATH & "acme/Blog/Config/Routes.cfm")>
```

See [URI Routing](../incoming/routing.md).

### Filters

Filters of a module are not found by themselves. Add the alias of the filter of your module to **app/Config/Filters.cfc**. See [Controller Filters](../incoming/filters.md).

### Controllers

Controllers outside of the main **app/Controllers** directory cannot be automatically routed by URI detection, but must be specified within the Routes file itself. See [URI Routing](../incoming/routing.md).

### Config Files

No special change is needed when working with configuration files. These are still namespaced components and loaded with `new_()`:

```cfm
<cfset blogConfig = new_("Acme\Blog\Config\Blog")>
```

The `config()` function looks a short class name up in the `Config` namespace only, so pass the full class name for a config file of a module:

```cfm
<cfset blogConfig = config("Acme\Blog\Config\Blog")>
```

### Migrations

Migration files are found in the **Database/Migrations** folder of every folder of a namespace. By default the migration runner uses the namespace `App`. Call `setNamespace("Acme\Blog")` on it to use the module, or `setNamespace("")` to use all namespaces. Only that folder is searched, not the folders below it. See [Database Migrations](../dbmgmt/migration.md).

### Seeds

!!! warning "Not ported yet"
    There is no command line, so a seed is not run from one. See [Database Seeding](../dbmgmt/seeds.md).

### Helpers

A helper of a module is not found by `helper()`, which looks in **app/Helpers** and **system/Helpers** only. See [Helper Functions](helpers.md).

### Language Files

Language files are searched in the **Language/{locale}** folder of every namespace that the autoloader knows, as long as the file follows the same directory structure as the main application directory. See [Localization](../outgoing/localization.md).

### Libraries

Libraries are always instantiated by their fully-qualified class name, so no special access is provided:

```cfm
<cfset lib = new_("Acme\Blog\Libraries\BlogLibrary")>
```

### Models

If you instantiate models by their fully-qualified class names, no special access is provided:

```cfm
<cfset postModel = new_("Acme\Blog\Models\PostModel")>
```

### Views

Views can be loaded using the class namespace:

```cfm
<cfset output = view("Acme\Blog\Views\posts")>
```

See [Views](../outgoing/views.md).
