# Factories

## Introduction

### What are Factories?

**Factories** are a way to load a class so that you get the same instance each time you ask for it. They are implemented by the component `ColdIgniter\Config\Factories` (**system/Config/Factories.cfc**).

Factories allow dynamic loading of components by their name. The shared instance gives a large performance boost and helps keep code clean of lengthy instantiation checks.

Factories are similar to CodeIgniter 3's `$this->load` in these points:

- Load a class
- Share the loaded class instance

Any class can be loaded by Factories, but the best examples are those classes that are used to work on or transmit common data. The framework itself uses Factories internally, for example to make sure the correct configuration is loaded when you call `config()`.

!!! note "Difference from CodeIgniter"
    The class has static functions in CodeIgniter. In ColdIgniter it is used through `static_("ColdIgniter\Config\Factories")`.

### Differences from Services

[Services](services.md) are for the classes of the framework itself and are defined, one function each, in **app/Config/Services.cfc**. Factories are for the classes of your application: config objects, models, libraries. A factory needs no definition: it finds the class by its name, creates it once and keeps it for the rest of the request.

Factories require a concrete class name to instantiate and do not have code to create instances. So Factories are not good for creating a complex instance that needs many dependencies, and you cannot change the class of the instance to be returned, unless you [define](#defining-classname-to-be-loaded) it.

On the other hand, Services have code to create instances, so a service can create a complex instance that needs other services or class instances. When you get a service, you give a service name, not a class name, so the returned instance can be changed without changing the client code.

## Loading Classes

### Loading a Class

Take **Models** as an example. In CodeIgniter you get the Factory for models with the magic static method `Factories::models()`. The static method name is called the *component*.

!!! note "Difference from CodeIgniter"
    A function cannot be called by a name that is made up at run time. In ColdIgniter, `__callStatic(component, arguments_)` does what the magic static method does. `component` is the name in lower case (`models`, `config`, `libraries`), and `arguments_` is an array. Its first item is the class name, its second item (optional) is a structure of [options](#factories-options), and the other items are given to the constructor of the class. For the common case of a shared instance with no options, `get(component, alias)` is a shorter call.

Both calls give an empty string when the class cannot be found.

#### Passing Classname without Namespace

If you pass a class name without a namespace, Factories first searches in the `App` namespace for the path that matches the component. The component `models` searches **app/Models**.

##### Passing Short Classname

In the following code, if you have `App\Models\UserModel`, the instance will be returned:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset userModel = factories.get("models", "UserModel")>
```

If you do not have `App\Models\UserModel`, it searches for `Models\UserModel` in all namespaces, the application first and `ColdIgniter` last.

The next time you ask for the same class anywhere in your code, Factories makes sure you get back the same instance:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset userModel = factories.get("models", "UserModel")>
```

!!! note "Difference from CodeIgniter"
    The search goes through the namespaces known to the autoloader, because the file locator of CodeIgniter is not ported yet. The result is the same.

##### Passing Short Classname with Sub-directories

If you want to load a class in sub directories, you use `/` as a separator. The following code loads **app/Libraries/Sub/SubLib.cfc** if it exists:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset subLib = factories.get("libraries", "Sub/SubLib")>
```

#### Passing Fully Qualified Classname

You can also request a fully qualified class name:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset userModel = factories.get("models", "Blog\Models\UserModel")>
```

It returns the instance of `Blog\Models\UserModel` if it exists.

## Convenience Functions

Two shortcut functions for Factories are provided. These functions are always available.

### config()

`config()` returns a shared instance of a Config class. The only required argument is the class name:

```cfm
<cfset appConfig = config("Config\App")>
```

The second argument is `getShared`. Pass `false` to get a new object:

```cfm
<cfset appConfig = config("Config\App", false)>
```

A name without a namespace, such as `App`, is looked for in the `Config` namespace first and gives the same object as `Config\App`.

### model()

`model()` returns a shared instance of a Model class. The only required argument is the class name:

```cfm
<cfset userModel = model("UserModel")>
```

It works like the call `__callStatic("models", ...)` above, so a short class name is looked for in **app/Models** first. The second argument is `getShared` (default `true`), and an optional third argument, a database connection, is given to the constructor of the model:

```cfm
<cfset userModel = model("UserModel", false)>
```

## Defining Classname to be Loaded

You can define the class name to be loaded before loading the class, with the `define()` method:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset factories.define("models", "Myth\Auth\Models\UserModel", "App\Models\UserModel")>
```

The first argument is a component. The second argument is a class alias (the class name you pass when you load the class), and the third argument is the true fully qualified class name to be loaded.

After that, if you load `Myth\Auth\Models\UserModel` with Factories, the `App\Models\UserModel` instance is returned:

```cfm
<cfset userModel = factories.get("models", "Myth\Auth\Models\UserModel")>
```

`define()` throws an `InvalidArgumentException` when the class does not exist, or when the alias is already defined for another class. Defining the same alias for the same class again does nothing.

## Factory Parameters

`__callStatic()` takes as its second array item a structure of option values (described below). These options override the default options of each component.

Any more items in the array are given on to the class constructor, which makes it easy to configure your class instance on the fly. For example, say your app uses a separate database for authentication and you want to be sure that any attempt to access user records goes through that connection:

```cfm
<cfset factories = static_("ColdIgniter\Config\Factories")>
<cfset arguments_ = ArrayNew(1)>
<cfset ArrayAppend(arguments_, "UserModel")>
<cfset ArrayAppend(arguments_, StructNew())>
<cfset ArrayAppend(arguments_, authDb)>
<cfset userModel = factories.__callStatic("models", arguments_)>
```

`authDb` is a database connection you made before. Now any time the `UserModel` is loaded from Factories it returns the instance that uses the alternate database connection.

## Factories Options

The default behavior might not work for every component. For example, your component name and its path may not match, or you may need to limit instances to a certain type of class. Each component takes a set of options to direct discovery and instantiation.

| Key        | Type            | Description                                                                                                                  | Default                                                         |
| ---------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| component  | string or empty | The name of the component (if different than the name used in the call). This can be used to alias one component to another. | empty (the component name)                                      |
| path       | string or empty | The relative path within the namespace or folder to look for classes.                                                        | empty (the component name, with the first letter in upper case) |
| instanceOf | string or empty | A required class name to match on the returned instance.                                                                     | empty (no filtering)                                            |
| getShared  | boolean         | Whether to return a shared instance of the class or load a fresh one.                                                        | `true`                                                          |
| preferApp  | boolean         | Whether a class with the same basename in the App namespace overrides other explicit class requests.                         | `true`                                                          |

!!! note
    `preferApp` works only when you request [a class name without a namespace](#passing-classname-without-namespace).

!!! note "Difference from CodeIgniter"
    A structure cannot hold null, so an option that is null in CodeIgniter is an empty string here. Keys of a structure are not case sensitive.

## Factories Behavior

Options can be applied in one of three ways (listed in ascending priority):

- A configuration class `Config\Factory` with a property that matches the name of a component.
- The method `setOptions()` of Factories.
- Passing options directly at call time, as the second item of the array of `__callStatic()`.

### Configurations

To set default component options, create the file **app/Config/Factory.cfc** that extends `ColdIgniter\Config\Factory` and supplies the options as a structure property that matches the name of the component.

#### Example: Filters Factories

For example, if you want to create **Filters** by Factories, the component name is `filters`. If you want to be sure that each filter is an instance of a class that extends the `FilterInterface` of ColdIgniter, your **app/Config/Factory.cfc** file might look like this:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Config\Factory">

	<cfset this.filters = StructNew()>
	<cfset this.filters["instanceOf"] = "ColdIgniter\Filters\FilterInterface">

</cfcomponent>
```

Now you can create a filter with `factories.get("filters", "SomeFilter")`, and the returned instance is sure to be a ColdIgniter filter. This prevents a conflict with a third-party module that happens to have an unrelated `Filters` path in its namespace.

#### Example: Library Factories

If you want to load your library classes in the **app/Libraries** directory with the component `library`, the path `Libraries` is different from the default path `Library`.

In this case, your **app/Config/Factory.cfc** file looks like this:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Config\Factory">

	<cfset this.library = StructNew()>
	<cfset this.library["path"] = "Libraries">

</cfcomponent>
```

Now you can load your libraries with the component `library`:

```cfm
<cfset someLib = factories.get("library", "SomeLib")>
```

### setOptions Method

The `Factories` component has a method to set options at run time: give the component and a structure of options, and they are merged with the default values and stored for the next call. It returns the stored options:

```cfm
<cfset options = StructNew()>
<cfset options["instanceOf"] = "App\Models\BaseModel">
<cfset factories.setOptions("models", options)>
```

`setOptions()` also resets the component: the shared instances and the defined aliases of that component are dropped, so they are found again with the new options.

### Parameter Options

`__callStatic()` takes as its second array item a structure of option values. These options override the stored options of the component and can be used at call time to get exactly what you need. The keys are the option names.

For example, by default Factories assumes that you want a shared instance of a component. With the second array item you control whether that one call returns a new or a shared instance:

```cfm
<cfset arguments_ = ArrayNew(1)>
<cfset options = StructNew()>
<cfset options["getShared"] = false>
<cfset ArrayAppend(arguments_, "UserModel")>
<cfset ArrayAppend(arguments_, options)>
<cfset userModel = factories.__callStatic("models", arguments_)>
```

### getOptions Method

`getOptions(component)` returns the stored options of a component. If none are stored yet, they are taken from `Config\Factory` (or from the defaults) and stored.

### reset Method

`reset(component)` drops the options, aliases and shared instances of one component. `reset()` with no argument drops them for every component.

### injectMock Method

`injectMock(component, alias, instance)` stores an object as the shared instance for an alias, so the next request for that alias returns it. It is meant for testing, to replace a class by a mock:

```cfm
<cfset factories.injectMock("models", "UserModel", mockModel)>
```

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
