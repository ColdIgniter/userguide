# Autoloading Files

Every application consists of a large number of classes in many different locations. The framework provides classes for core functionality. Your application will have a number of libraries, models, and other entities to make it work. Keeping track of where every single file is, and hard-coding that location into your files, is a massive headache and very error-prone. That's where the autoloader comes in.

## ColdIgniter Autoloader

ColdIgniter provides an autoloader that can be used with very little configuration. It locates individual namespaced classes by a directory structure that matches the class name: the class `App\Models\CustomerModel` is the file **app/Models/CustomerModel.cfc**.

The autoloader is always active. It is created and registered at the beginning of the framework's execution, in **system/Boot.cfc**.

!!! note "Difference from CodeIgniter"
    ColdFusion MX 6.1 has no namespaces, no `use` statements and no autoloader stack. In ColdIgniter a class name is a text such as `App\Models\CustomerModel`. You create an object from it with the global function `new_()`, and the autoloader finds the file, creates the component and runs its `__construct()` function. There is no Composer and no other autoloader to work beside it.

    ```cfm
    <cfset model = new_("App\Models\CustomerModel")>
    ```

    `new_(class, argument, argument, ...)` passes the arguments to `__construct()`, in order. A class with static members in the original (a call such as `Services::timer()`) is used through `static_("Config\Services").timer()`: `static_()` returns one object per class and request. `class_exists("App\Libraries\Foo")` tells whether the autoloader can find a class.

!!! important
    Be careful about the case of file names. Many developers work on case-insensitive file systems on Windows or macOS. However, most server environments use case-sensitive file systems. If the file name case is incorrect, the autoloader cannot find the file on the server.

A component that extends a class found by the autoloader names its parent in the `parent` attribute of `cfcomponent`, with the class name of the parent, for example `parent="ColdIgniter\Model"`. The `extends` attribute is used for a parent in the same folder, as controllers do with `BaseController`.

## Configuration

Initial configuration is done in **app/Config/Autoload.cfc**. This file contains the namespaces (`psr4`), the class map, the list of files and the list of helpers.

### Namespaces

The recommended method for organizing your classes is to create one or more namespaces for your application's files.

The `psr4` map in the configuration file allows you to map the namespace to the directory those classes can be found in:

```cfm
<cfset this.psr4 = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.psr4[request.APP_NAMESPACE] = request.APPPATH>
<cfset this.psr4["Acme\Blog"] = "/path/to/acme/blog">
```

The key of each row is the namespace itself. This does not need a trailing back slash. The value is the location of the directory the classes can be found in.

By default, the namespace `App` is located in the **app** directory, the namespace `Config` is located in the **app/Config** directory and the namespace `ColdIgniter` is the **system** directory.

If you create class files in these locations, with a file name that matches the class name, the autoloader will find them.

The `files` list holds paths of `.cfm` files that are included when the autoloader is registered, and the `helpers` list names helpers to load.

### Confirming Namespaces

!!! warning "Not ported yet"
    The `spark namespaces` command does not exist, because ColdIgniter has no command line.

### Application Namespace

By default, the application directory is namespaced to the `App` namespace. Controllers, libraries and models in the application directory are found under the `App` namespace.

#### Config Namespace

Config files are in the `Config` namespace, not in `App\Config` as you might expect. This allows the core system files to always be able to locate them, even when the application namespace has changed.

#### Changing App Namespace

You may change this namespace by editing the **app/Config/Constants.cfm** file and setting the new namespace value under the `APP_NAMESPACE` setting:

```cfm
<cfset defined("APP_NAMESPACE") OR define("APP_NAMESPACE", "App")>
```

You will need to modify any existing files that are referencing the current namespace. There is no **composer.json** to change.

### Classmap

!!! warning "Not ported yet"
    The class map of **app/Config/Autoload.cfc** is read, but classes listed in it are not loaded from it yet. Use a namespace in `psr4`.

## Composer Support

Composer does not exist on ColdFusion MX 6.1. Put a third-party library in a folder of its own, map a namespace to that folder in `psr4`, and create its classes with `new_()`.

### Priority of Autoloaders

There is one autoloader only, so there is no order of autoloaders.

## FileLocator Caching

!!! warning "Not ported yet"
    The FileLocator (`Autoloader/FileLocator.php` and `FileLocatorCached.php` in CodeIgniter) is not ported, so there is no `service("locator")` and no FileLocator cache. Helpers are searched in **app/Helpers** and **system/Helpers** only.
