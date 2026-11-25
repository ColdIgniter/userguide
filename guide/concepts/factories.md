# Factories

## Introduction

### What are Factories?

**Factories** are a way to load a class so that you get the same instance each time you ask for it. They are implemented by the component `ColdIgniter\Config\Factories` (**system/Config/Factories.cfc**).

Factories allow dynamic loading of components by their name. The shared instance gives a large performance boost and helps keep code clean of lengthy instantiation checks.

!!! note "Difference from CodeIgniter"
    The class has static functions in CodeIgniter. In ColdIgniter it is used through `static_("ColdIgniter\Config\Factories")`.

### Differences from Services

[Services](services.md) are for the classes of the framework itself and are defined, one function each, in **app/Config/Services.cfc**. Factories are for the classes of your application: config objects, models, libraries. A factory needs no definition: it finds the class by its name, creates it once and keeps it for the rest of the request.

## Loading Classes

### Loading a Class

A factory call has the type of the class (the component, such as `config` or `models`) and the class name:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset userModel = factories.get("models", "App\Models\UserModel")>
```

A second call with the same class name returns the same instance. A class that cannot be found gives an empty string.

To get a new instance each time, call `__callStatic()` with the name of the type, an array with the class name, and a structure with the option `getShared` set to `false`:

```cfm
<cfset arguments_ = ArrayNew(1)>
<cfset options = StructNew()>
<cfset options["getShared"] = false>
<cfset ArrayAppend(arguments_, "App\Models\UserModel")>
<cfset ArrayAppend(arguments_, options)>
<cfset userModel = factories.__callStatic("models", arguments_)>
```

#### Passing Classname without Namespace

A class name without a namespace is looked for in the `Config` namespace only, that is, in **app/Config**.

!!! warning "Not ported yet"
    The search of a short class name in the namespace of the type, such as `App\Models`, is not ported (`locateClass()` of `Config/Factories.php`). Pass the fully qualified class name for anything that is not a config class.

#### Passing Fully Qualified Classname

You can pass the fully qualified class name, for example `App\Models\UserModel`. The class is found when the autoloader finds its file.

## Convenience Functions

Two convenience functions are provided in CodeIgniter. They are called the same way here, but only one of them exists.

### config()

`config()` is a short way to get a config object. It always uses the type `config`:

```cfm
<cfset appConfig = config("Config\App")>
<cfset appConfig = config("App")>
```

Both lines return the same object, because the short name `App` is looked for in the `Config` namespace. The second argument is `getShared`. Pass `false` to get a new object:

```cfm
<cfset appConfig = config("Config\App", false)>
```

### model()

!!! warning "Not ported yet"
    The global function `model()` does not exist. Get a model with the factory call shown above, or create one with `new_("App\Models\UserModel")`.

## Defining Classname to be Loaded

!!! warning "Not ported yet"
    `Factories::define()` (`Config/Factories.php` of CodeIgniter) is not ported, so a class name cannot be bound to another class. To replace a class, change the class name where it is requested.

## Factory Parameters

!!! warning "Not ported yet"
    A factory cannot pass parameters to the constructor of the class it creates. Create the object with `new_()` when it needs arguments.

## Factories Options

!!! warning "Not ported yet"
    The options of `Factories::setOptions()` (`component`, `path`, `instanceOf`, `getShared` and `preferApp`) cannot be changed. The options of the `config` type are fixed in the component. The only option read at a call is `getShared`.

## Factories Behavior

### Configurations

!!! warning "Not ported yet"
    The search of a class in every namespace, with the application namespace first (`preferApp`), is not ported. See [Passing Classname without Namespace](#passing-classname-without-namespace).

#### Example: Filters Factories

!!! warning "Not ported yet"
    A filters factory with its own options is not ported. See the page on [Filters](../incoming/filters.md) for how filters are found.

#### Example: Library Factories

!!! warning "Not ported yet"
    A libraries factory with its own options is not ported. Use `new_()` or the call of [Loading a Class](#loading-a-class) with the fully qualified class name.

### setOptions Method

!!! warning "Not ported yet"
    `setOptions()` is not ported.

### Parameter Options

!!! warning "Not ported yet"
    The options of a call, other than `getShared`, are not ported.

## Config Caching

!!! warning "Not ported yet"
    The cache of the config objects (`Cache/FactoriesCache.php` and `Cache/FactoriesCache/FileVarExportHandler.php` of CodeIgniter) is not ported.

### Prerequisite

!!! warning "Not ported yet"
    Not ported, see above.

### How It Works

!!! warning "Not ported yet"
    Not ported, see above.

### How to Update Config Values

!!! warning "Not ported yet"
    Not ported, see above.

### How to Enable Config Caching

!!! warning "Not ported yet"
    Not ported, see above.
