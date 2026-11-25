# Helper Functions

## What are Helpers?

Helpers, as the name suggests, help you with tasks. Each helper file is simply a collection of functions in a particular category. There are [URL Helpers](../helpers/url_helper.md), that assist in creating links, there are [Form Helpers](../helpers/form_helper.md) that help you create form elements, [Text Helpers](../helpers/text_helper.md) perform various text formatting routines, [Cookie Helpers](../helpers/cookie_helper.md) set and read cookies, [Array Helpers](../helpers/array_helper.md) help you work with arrays, [HTML Helpers](../helpers/html_helper.md) build HTML tags, etc.

Unlike most other systems in ColdIgniter, Helpers are not written in an Object Oriented format. They are simple, procedural functions. Each helper function performs one specific task, with no dependence on other functions.

ColdIgniter does not load Helper Files by default, so the first step in using a Helper is to load it. Once loaded, it becomes globally available in your [controller](../incoming/controllers.md) and [views](../outgoing/views.md).

Helpers are typically stored in your **system/Helpers**, or **app/Helpers** directory. A helper is a `.cfm` file that holds `cffunction` tags.

## Loading Helpers

!!! note
    The [URL Helper](../helpers/url_helper.md) is always loaded so you do not need to load it yourself.

### Loading a Helper

Loading a helper file is quite simple using the following method:

```cfm
<cfset helper("name")>
```

The above code loads the **name_helper.cfm** file.

!!! important
    ColdIgniter helper file names are all lowercase. Therefore, `helper("Name")` will not work on case-sensitive file systems.

For example, to load the [Cookie Helper](../helpers/cookie_helper.md) file, which is named **cookie_helper.cfm**, you would do this:

```cfm
<cfset helper("cookie")>
```

!!! note
    The `helper()` function does not return a value, so do not try to assign it to a variable. Just use it as shown.

#### Auto-Discovery and Composer Packages

By default, ColdIgniter searches for the helper files in all namespaces that the autoloader knows (see [Autoloader](../concepts/autoloader.md)), through the file locator. You can also [specify a namespace](#loading-from-specified-namespace) for a helper that you want to load.

!!! note "Difference from CodeIgniter"
    There is no Composer on ColdFusion MX 6.1, so the namespaces are the ones in the `psr4` map of **app/Config/Autoload.cfc**.

#### Load Order

The `helper()` function looks for a file of the name in the **Helpers** folder of every namespace, and loads ALL matching helpers of the same name. This allows any module's helpers to be loaded, as well as any helpers you have created for this application, and you can add to the functions of a helper that ships with the framework.

The load order is as follows:

1. app/Helpers - Files loaded here are always loaded first.
2. The **Helpers** folder of each other namespace.
3. system/Helpers - The base file is loaded last.

A function name that is already defined is kept: the helper functions have no guard against a second definition, and the first one stays. As the file of your application is loaded first, its functions win.

### Loading Multiple Helpers

If you need to load more than one helper at a time, you can pass an array of file names in and all of them will be loaded:

```cfm
<cfset helper(ListToArray("cookie,text"))>
```

### Loading in a Controller

A helper can be loaded anywhere within your controller methods (or even within your View files, although that is not a good practice), as long as you load it before you use it.

You can load your helpers in the function `initController()` of your controller so that they become available automatically in any method, or you can load a helper in a specific method that needs it.

However, if you want to load them for the whole controller, you can use the `helpers` property of the controller instead. See [Controllers](../incoming/controllers.md).

### Loading from Specified Namespace

By default, ColdIgniter searches for the helper files in all namespaces and loads all the files it finds.

If you want to load only a helper in a specific namespace, prefix the name of the helper with the namespace that it can be located in. Within that namespaced directory, the loader expects it to live within a sub-directory named **Helpers**. For example, if the Blog code is in the namespace `Example\Blog`, with its files in **Modules/Blog/**, the helper file **Modules/Blog/Helpers/blog_helper.cfm** is loaded with:

```cfm
<cfset helper("Example\Blog\blog")>
```

### Auto-loading Helpers

If you find that you need a particular helper globally throughout your application, you can tell ColdIgniter to auto-load it during system initialization. This is done by opening the **app/Config/Autoload.cfc** file and adding the helper to the `helpers` property:

```cfm
<cfset ArrayAppend(this.helpers, "form")>
```

## Using a Helper

Once you have loaded the Helper File containing the function you intend to use, you will call it the way you would a standard function.

For example, to create a link using the `anchor()` function in one of your view files you would do this:

```cfm
<cfoutput>#anchor("blog/comments", "Click Here")#</cfoutput>
```

Where `Click Here` is the name of the link, and `blog/comments` is the URI to the controller/method you wish to link to.

## Creating Helpers

### Creating Custom Helpers

The helper filename is the **helper name** and **_helper.cfm**.

For example, to create info helper, you will create a file named **app/Helpers/info_helper.cfm**, and add a function to the file:

```cfm
<cffunction name="helpMe" returntype="string" output="no">
	<cfreturn "Help yourself.">
</cffunction>
```

You can add as many functions as you like to a single helper file.

### "Extending" Helpers

To "extend" Helpers, create a file in your **app/Helpers** folder with an identical name to the existing Helper.

If all you need to do is add some functionality to an existing helper, perhaps add a function or two, or change how a particular helper function operates, then it is overkill to replace the entire helper with your version. In this case, it is better to simply "extend" the Helper.

!!! note
    The term "extend" is used loosely since Helper functions are procedural and discrete and cannot be extended in the traditional programmatic sense. Under the hood, this gives you the ability to add to, or to replace the functions a Helper provides.

For example, to extend the native **Array Helper** you will create a file named **app/Helpers/array_helper.cfm**, and add or override functions:

```cfm
<cffunction name="array_flatten_with_dots" returntype="any" output="no">
	<cfargument name="array" type="any" required="yes">

	<!--- your own version of the function --->
	<cfreturn "">
</cffunction>
```

!!! important
    Do not specify the namespace `App\Helpers`.

See [Load Order](#load-order) for the order in which helper files are loaded.

## Now What?

In the Table of Contents, you will find a list of all the available [Helpers](../helpers/index.md). Browse each one to see what they do.
