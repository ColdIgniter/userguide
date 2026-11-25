# Auto Routing (Improved)

This page covers routing from the controller and method names, without a route list.

## What is Auto Routing (Improved)?

By default, all routes must be [defined](routing.md#setting-routing-rules) in the configuration file.

With **Auto Routing (Improved)**, you name the controller and its method by convention and the request is routed automatically. There is no need to define routes by hand.

If you enable Auto Routing (Improved), when no defined route matches the URI, the system tries to match the URI against the controllers and methods.

!!! important
    For security reasons, if a controller is used in the defined routes, Auto Routing (Improved) does not route to that controller.

!!! note
    Auto Routing (Improved) is disabled by default. To use it, see [Enable Auto Routing (Improved)](#enable-auto-routing-improved).

!!! warning "Not ported yet"
    The older Auto Routing (Legacy) of CodeIgniter 4 is not ported. With `autoRoutesImproved` set to `false` in **app/Config/Feature.cfc**, auto routing finds no route and the URI gives a "page not found" error.

## Differences from Auto Routing (Legacy)

Auto Routing (Legacy) is a routing system from CodeIgniter 3, and ColdIgniter does not have it. If you do not know it, go to the next section.

If you know it well, these are some changes in **Auto Routing (Improved)**:

- A controller method needs an HTTP verb prefix, like `getIndex()` and `postCreate()`.
    - Since developers always know the HTTP method, a request with an unexpected HTTP method never runs the controller.
- The Default Controller (`Home` by default) and the Default Method (`index` by default) must be left out of the URI.
    - This keeps one URI for each controller method.
    - For example, by default you can open `/`, but `/home` and `/home/index` give "page not found".
- It checks the method parameter count.
    - If the URI has more parameters than the method, the result is "page not found".
- It does not support the `_remap()` method.
    - This keeps one URI for each controller method.
    - Instead it has the [Default Method Fallback](#default-method-fallback).
- It cannot reach controllers that are in defined routes.
    - Controllers reached by Auto Routing are kept apart from controllers reached by defined routes.

## Enable Auto Routing (Improved)

Set the `autoRoute` option to `true` in **app/Config/Routing.cfc**:

```cfm
<cfset this.autoRoute = true>
```

The property `autoRoutesImproved` in **app/Config/Feature.cfc** is already `true`:

```cfm
<cfset this.autoRoutesImproved = true>
```

!!! important
    When you use Auto Routing (Improved), you must remove this line in **app/Config/Routes.cfm**:

    ```cfm
    <cfset routes.get("/", "Home::index")>
    ```

    Defined routes take precedence over Auto Routing, and Auto Routing (Improved) denies access to a controller that a defined route uses.

## URI Segments

The segments in the URL, following the Model-View-Controller approach, usually represent:

```
http://example.com/{class}/{method}/{param1}
```

1. The first segment represents the controller **class** that should be invoked.
2. The second segment represents the class **method** that should be called.
3. The third and any further segments represent the **parameters** passed to the controller method.

Consider this URI:

```
http://example.com/hello-world/hello/1
```

In the example above, when you send an HTTP request with the **GET** method, Auto Routing (Improved) looks for a controller named `App\Controllers\HelloWorld` and runs its `getHello()` method, passing `"1"` as the first parameter.

!!! note
    A controller method that Auto Routing (Improved) runs needs an HTTP verb (`get`, `post`, `put` and so on) prefix, like `getIndex()` and `postCreate()`.

!!! note
    When the short name of a controller matches the first segment of a URI, it is loaded.

## Let's try it: Hello World!

Let's make a simple controller so you can see it in action.

### Create a Controller

Create a file called **HelloWorld.cfc** in your **app/Controllers** folder, and put this code in it:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="getIndex" access="public" returntype="string" output="no">
		<cfreturn "Hello World!">
	</cffunction>

</cfcomponent>
```

!!! important
    The file must be called **HelloWorld.cfc**. When you use Auto Routing (Improved), controller class names MUST be in CamelCase.

The `HelloWorld` controller extends `BaseController`. The BaseController is a convenient place for loading components and doing the work that all your controllers need. You can extend this component in any new controller.

!!! important
    A controller method that Auto Routing (Improved) runs needs an HTTP verb prefix, like `getIndex()` and `postCreate()`.

### Check the Routes

!!! warning "Not ported yet"
    The command `spark routes`, which lists the routes with the mark `GET(auto)`, is not ported, because ColdIgniter has no command line tool.

### Visit Your Site

Now visit your site using a URL similar to this:

```
http://example.com/hello-world
```

The system translates a URI with dashes (`-`) to CamelCase in the controller and method segments.

For example, the URI `sub-dir/hello-controller/some-method` runs the method `SubDir\HelloController::getSomeMethod()`.

If you did it right, you should see:

```
Hello World!
```

## Examples of Controller Names

The file **app/Controllers/HelloWorld.cfc** holds a valid controller name, because `App\Controllers\HelloWorld` is CamelCase.

The file **app/Controllers/helloWorld.cfc** is **not** valid, because the first letter (`h`) is not a capital.

The file **app/Controllers/helloworld.cfc** is also **not** valid, for the same reason.

## Controller Methods

### Method Visibility

A method that an HTTP request can run must be declared `access="public"`. A request for a private method gives "page not found".

!!! warning
    For security reasons, declare every new utility method as `access="private"`.

!!! note "Difference from CodeIgniter"
    CFML on ColdFusion MX 6.1 has no reflection. The router reads the methods, their access and their parameters from the component metadata. A method with no `access` attribute counts as public. Parameters are counted by the `cfargument` tags.

### Default Method

In the example above the method name is `getIndex()`. The method (HTTP verb plus `Index()`) is called the **Default Method**, and is run if the **second segment** of the URI is empty.

### Normal Methods

The second segment of the URI chooses which method of the controller is called.

Add a new method to your controller:

```cfm
<cffunction name="getComment" access="public" returntype="string" output="no">
	<cfreturn "This is a comment.">
</cffunction>
```

Now load this URL to run the `getComment()` method:

```
http://example.com/hello-world/comment/
```

You should see your new message.

## Passing URI Segments to Your Methods

If your URI has more than two segments, they are passed to your method as parameters.

For example, say you have a URI like this:

```
http://example.com/products/shoes/sandals/123
```

Your method is passed URI segments 3 and 4 (`"sandals"` and `"123"`), with the component below as `app/Controllers/Products.cfc`:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="getShoes" access="public" returntype="string" output="no">
		<cfargument name="type" type="string" required="yes">
		<cfargument name="id" type="string" required="yes">

		<cfreturn arguments.type & arguments.id>
	</cffunction>

</cfcomponent>
```

!!! note
    If the URI has more parameters than the method, Auto Routing (Improved) does not run the method, and the result is "page not found".

## Default Controller

The **Default Controller** is a special controller used when a URI ends with a folder name, or when no URI is given, as when only the root URL of your site is requested.

By default the Default Controller is `Home`.

!!! note
    Define only the default method (`getIndex()` for GET requests) in the default controller. If you define any other public method, it is not run.

For more, see [Configuration Options](#configuration-options).

## Default Method Fallback

If the controller method for the URI segment does not exist, and the default method is defined, the remaining URI segments are passed to the default method.

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="getIndex" access="public" returntype="string" output="no">
		<cfargument name="id" type="string" required="no" default="">
		<cfargument name="action" type="string" required="no" default="">

		<cfreturn "">
	</cffunction>

</cfcomponent>
```

Load this URL, with the component above as `app/Controllers/Product.cfc`:

```
http://example.com/product/15/edit
```

The method is passed URI segments 2 and 3 (`"15"` and `"edit"`).

!!! important
    If the URI has more parameters than the method, Auto Routing (Improved) does not run the method, and the result is "page not found".

### Fallback to Default Controller

If the controller for the URI segment does not exist, and the default controller (`Home` by default) exists in the folder, the remaining URI segments are passed to the default method of the default controller.

For example, when you have the default controller `Home` in the **app/Controllers/News** folder:

```cfm
<cfcomponent output="no" parent="App\Controllers\BaseController">

	<cffunction name="getIndex" access="public" returntype="string" output="no">
		<cfargument name="id" type="string" required="no" default="">

		<cfreturn "">
	</cffunction>

</cfcomponent>
```

Load this URL:

```
http://example.com/news/101
```

The `News\Home` controller and its default `getIndex()` method are found. The default method gets the second URI segment (`"101"`).

!!! note
    If there is an `App\Controllers\News` controller, it takes precedence. The URI segments are searched in order, and the first controller found is used.

!!! note
    If the URI has more parameters than the method, Auto Routing (Improved) does not run the method, and the result is "page not found".

## Organizing Your Controllers into Sub-directories

If you build a large application, you may want to organize your controllers in sub-folders.

Make sub-folders under **app/Controllers** and put your controller components in them.

!!! important
    Folder names MUST start with an uppercase letter and be in CamelCase.

When you use this feature, the first segment of your URI must name the folder. For example, say you have a controller here:

```
app/Controllers/Products/Shoes.cfc
```

To call that controller your URI looks like this:

```
http://example.com/products/shoes/show/123
```

!!! note
    You cannot have a folder with the same name in **app/Controllers** and in **public**. If there is a folder, the web server looks for it and does not route the request to ColdIgniter.

Each of your sub-folders may have a default controller, called when the URL has *only* the sub-folder. Put a controller there that matches the name of your default controller in **app/Config/Routing.cfc**.

## Examples of Controller/Methods and URIs

For a **GET** request with the default configuration, controllers, methods and URIs map like this:

| Controller/Method | URI | Description |
| --- | --- | --- |
| `Home::getIndex()` | / | The default controller and the default method. |
| `Blog::getIndex()` | /blog | The default method. |
| `UserProfile::getIndex()` | /user-profile | The default method. |
| `Blog::getTags()` | /blog/tags | |
| `Blog::getNews(id)` | /blog/news/123 | |
| `Blog\Home::getIndex()` | /blog | Sub-folder `Blog`, the default controller and the default method. If there is a `Blog` controller, it takes precedence. |
| `Blog\Tags::getIndex()` | /blog/tags | Sub-folder `Blog` and the default method. If there is a `Blog` controller, it takes precedence. |
| `Blog\News::getIndex(id)` | /blog/news/123 | Sub-folder `Blog` and the default method fallback. If there is a `Blog` controller, it takes precedence. |

## Applying Filters

Controller filters let you add work before and after the controller method. This is handy for authentication or API logging.

If you use Auto Routing, set the filters to apply in **app/Config/Filters.cfc**. See [Controller Filters](filters.md) for how to set them up.

## Configuration Options

These options are in the file **app/Config/Routing.cfc**.

### Default Controller

#### For Site Root URI

When a user visits the root of your site (for example **http://example.com**), the controller to use is set by the `defaultController` property, unless a route exists for it.

The default value is `Home`, which matches the controller at **app/Controllers/Home.cfc**:

```cfm
<cfset this.defaultController = "Home">
```

#### For Directory URI

The default controller is also used when no route matches and the URI points to a folder in the controllers folder. For example, if the user visits **http://example.com/admin** and a controller is found at **app/Controllers/Admin/Home.cfc**, it is used.

!!! important
    You cannot reach the default controller with the URI of the controller name. When the default controller is `Home`, you can open **http://example.com/**, but **http://example.com/home** is not found.

### Default Method

This works like the default controller setting, but sets the default method used when a controller matches the URI and no segment exists for the method. The default value is `index`.

In this example, if the user visits **example.com/products** and a `Products` controller exists, the method `Products::getListAll()` runs:

```cfm
<cfset this.defaultMethod = "listAll">
```

!!! important
    You cannot reach the controller with the URI of the default method name. In the example above you can open **example.com/products**, but **example.com/products/listall** is not found.

### Translate URI To CamelCase

This option translates a URI with dashes (`-`) to CamelCase in the controller and method segments. It is on by default, as in CodeIgniter 4.6.0 and later.

For example, the URI `sub-dir/hello-controller/some-method` runs the method `SubDir\HelloController::getSomeMethod()`.

!!! note
    When this option is on, the `translateURIDashes` option is ignored.

!!! note "Difference from CodeIgniter"
    A file system that ignores case would load `Blogcontroller` for the file `BlogController`. ColdIgniter reads the folder listing and compares the names of the folders and the class file with the case as written. This is done only when the option is on, and costs one folder listing for each segment of the class name.

#### Disable Translate URI To CamelCase

!!! note
    The option to turn this off exists only for backward compatibility. It is not recommended.

To turn it off, set `translateUriToCamelCase` to `false` in **app/Config/Routing.cfc**:

```cfm
<cfset this.translateUriToCamelCase = false>
```

## Module Routing

You can use auto routing even if you use [modules](../general/modules.md) and place the controllers in a different namespace.

To route to a module, set the `moduleRoutes` property in **app/Config/Routing.cfc**. It is a map. The key is the first URI segment for the module, and the value is the controller namespace:

```cfm
<cfset this.moduleRoutes.put("blog", "Acme\Blog\Controllers")>
```

With this setting, **http://localhost:8080/blog/foo/bar** is routed to `Acme\Blog\Controllers\Foo::getBar()`.

!!! note
    If you define `moduleRoutes`, the routing for the module takes precedence. In the example above, even if you have the `App\Controllers\Blog` controller, **http://localhost:8080/blog** is routed to the default controller `Acme\Blog\Controllers\Home`.
