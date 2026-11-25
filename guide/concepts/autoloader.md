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

The `psr4` map in the configuration file allows you to map the namespace to the directory those classes can be found in. The shipped file already creates the map and adds the `App` row (`this.psr4[request.APP_NAMESPACE] = request.APPPATH`). Add your own row below it:

```cfm
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

If you use third-party libraries that are not namespaced, you can load those classes using the classmap in **app/Config/Autoload.cfc**:

```cfm
<cfset this.classmap["Legacy_Report"] = "/path/to/lib/Legacy_Report.cfc">
```

The key of each row is the name of the class that you want to locate. The value is the path of its `.cfc` file. The file has to exist, or the class is looked for in the namespaces. The classmap is read before the namespaces.

## Composer Support

Composer does not exist on ColdFusion MX 6.1. Put a third-party library in a folder of its own, map a namespace to that folder in `psr4`, and create its classes with `new_()`.

### Priority of Autoloaders

There is one autoloader only, so there is no order of autoloaders.

## FileLocator Caching

The **FileLocator** (`service("locator")`) is responsible for finding files or getting a class name from a file. `helper()`, namespaced views, the Parser and the class search of [Factories](factories.md) use it.

To improve its performance, FileLocator Caching is available. The cached locator (`ColdIgniter\Autoloader\FileLocatorCached`) is used when the setting is on.

### How It Works

- At the end of the request, the data the locator found is saved in the cache, if the data changed.
- When the locator is made, the saved data is restored if it is there.

!!! note "Difference from CodeIgniter"
    CFML has no destructor, so ColdIgniter saves the data at the end of the request, after the `post_system` event. The data is kept by the [cache service](../libraries/caching.md) under the key `FileLocatorCache`, not in a file of its own, and it is kept for one day.

### How to Delete Cached Data

Once stored, the cached data does not change until it expires or is deleted. If you add or remove files, or change file paths or namespaces, the old data is returned and your app may not work properly. In that case delete the data with `service("locator").deleteCache()`, or delete the key `FileLocatorCache` from the cache.

### How to Enable FileLocator Caching

Set the following property to `true` in **app/Config/Optimize.cfc**:

```cfm
<cfset this.locatorCacheEnabled = true>
```

!!! note "Difference from CodeIgniter"
    There is no `spark optimize` command, so the property is set by hand. The other property of **app/Config/Optimize.cfc**, `configCacheEnabled`, is not read by anything yet (see [Config Caching](factories.md#config-caching)).
