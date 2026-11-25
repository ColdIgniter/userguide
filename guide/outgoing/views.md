# Views

A view is simply a web page, or a page fragment, like a header, footer, sidebar, etc. In fact,
views can flexibly be embedded within other views (within other views, etc.) if you need
this type of hierarchy.

Views are never called directly, they must be loaded by a controller.

!!! warning "Not ported yet"
    View routes (a route that loads a view without a controller, `RouteCollection::view()` in CodeIgniter) are not ported. Load every view from a controller.

Remember that in an MVC framework, the controller acts as the traffic cop, so it is responsible
for fetching a particular view. If you have not read the [Controllers](../incoming/controllers.md)
page, you should do so before continuing.

Using the example controller you created in the controller page, let's add a view to it.

## Creating a View

Using your text editor, create a file called **blog_view.cfm** and put this in it:

```cfm
<html>
    <head>
        <title>My Blog</title>
    </head>
    <body>
        <h1>Welcome to my Blog!</h1>
    </body>
</html>
```

Then save the file in your **app/Views** directory.

## Displaying a View

To load and display a particular view file you will use the `view()` function in your
controller. A controller action returns the text of the page, so return the result of `view()`:

```cfm
<cfreturn view("blog_view")>
```

Where *name* is the name of your view file.

!!! important
    If the file extension is omitted, then the views are expected to end with the **.cfm** extension.
    If the file does not exist, a `ColdIgniter.View.Exceptions.ViewException` is thrown.

!!! note
    The `view()` function uses the [View Renderer](view_renderer.md) internally.

Now, create a file called **Blog.cfc** in the **app/Controllers** directory, and put this in it:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="index" access="public" returntype="string" output="no">
		<cfreturn view("blog_view")>
	</cffunction>

</cfcomponent>
```

Open the routing file located at **app/Config/Routes.cfm**, and add the following code:

```cfm
<cfset routes.get("blog", "Blog::index")>
```

If you visit your site, you should see your new view. The URL was similar to this:

```text
example.com/index.cfm/blog/
```

## Loading Multiple Views

A controller action returns one string, so to show several views in one page you join the
results of the `view()` calls with `&`.

For example, you may wish to have a header view, a menu view, a
content view, and a footer view. That might look something like this:

```cfm
<cffunction name="index" access="public" returntype="string" output="no">
	<cfset var data = StructNew()>
	<cfset var page = "">

	<cfset data["page_title"] = "Your title">

	<cfset page = view("header", data)>
	<cfset page = page & view("menu")>
	<cfset page = page & view("content")>
	<cfset page = page & view("footer")>

	<cfreturn page>
</cffunction>
```

In the example above, we are using "dynamically added data", which you will see below.

## Storing Views within Sub-directories

Your view files can also be stored within sub-directories if you prefer that type of organization.
When doing so you will need to include the directory name loading the view. Example:

```cfm
<cfreturn view("directory_name/file_name")>
```

## Namespaced Views

You can store views under a **Views** directory that is namespaced, and load that view as if it was namespaced.
ColdIgniter finds it through the namespaces of the [Autoloader](../concepts/autoloader.md), so you can package your
views together in a module-like fashion for easy re-use or distribution.

If you have an **example/blog** directory that is mapped to the namespace `Example\Blog` in the Autoloader,
you could retrieve view files as if they were namespaced also.

Following this example, you could load the **blog_view.cfm** file from **example/blog/Views** by
prepending the namespace to the view name (specifying the end directory `Views` as in
`Example\Blog\Views\` is not required):

```cfm
<cfreturn view("Example\Blog\Views\blog_view")>
```

The longest namespace that matches the start of the name is used, and the rest of the name is a path inside
the folder mapped to it. So the name above looks for **Views/blog_view.cfm** under the folder of `Example\Blog`.

!!! note
    You must use backslashes (`\`) when referencing namespaced views,
    as this is how ColdIgniter distinguishes a namespaced view from a regular
    file path. Forward slashes (`/`) are only used for subdirectories within
    the standard **app/Views** folder.

!!! note "Difference from CodeIgniter"
    The name is mapped to the folder as it is written, so the `Views` folder has to be part of the name in the
    example above. The shorter form without `Views` is not known to the lookup. Check this before you rely on it in a package.

## Overriding Namespaced Views

!!! warning "Not ported yet"
    Overriding a namespaced view from a folder in **app/Views** (`appOverridesFolder` in `Config\View`, CodeIgniter 4.7.0) is not ported. The setting is listed in **app/Config/View.cfc**, but the code ignores it.

### Configuration

!!! warning "Not ported yet"
    See above: the `appOverridesFolder` setting has no effect.

### Example

!!! warning "Not ported yet"
    See above: a view in **app/Views/overrides** is not used in place of a namespaced view.

## Caching Views

!!! warning "Not ported yet"
    The `cache` and `cache_name` options of `view()` are not ported. They are ignored, and the view is rendered on every call.

## Adding Dynamic Data to the View

Data is passed from the controller to the view by way of a struct in the second
parameter of the `view()` function.

Here's an example:

```cfm
<cfset data = StructNew()>
<cfset data["title"] = "My Real Title">
<cfset data["heading"] = "My Real Heading">

<cfreturn view("blog_view", data)>
```

Let's try it with your controller file. Open it and add this code:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="index" access="public" returntype="string" output="no">
		<cfset var data = StructNew()>

		<cfset data["title"] = "My Real Title">
		<cfset data["heading"] = "My Real Heading">

		<cfreturn view("blog_view", data)>
	</cffunction>

</cfcomponent>
```

Now open your view file and change the text to variables that correspond to the struct keys in your data:

```cfm
<cfoutput>
<html>
    <head>
        <title>#esc(title)#</title>
    </head>
    <body>
        <h1>#esc(heading)#</h1>
    </body>
</html>
</cfoutput>
```

Then load the page at the URL you've been using and you should see the variables replaced.

Each key of the struct becomes a variable of the same name inside the view. Use `esc()` for anything
that comes from a user or a database, see [Escaping Data](view_renderer.md).

!!! note "Difference from CodeIgniter"
    Write the keys with brackets, as in `data["title"]`. A key written with dots, as in `data.title`, is stored in upper case.

### The saveData Option

The data passed in is retained for subsequent calls to `view()`. If you
call the function multiple times in a single request, you will not have to pass
the desired data to each `view()`.

But this might not keep any data from "bleeding" into
other views, potentially causing issues. If you would prefer to clean the data after one call, you can pass the `saveData` option
in the struct of the third parameter.

```cfm
<cfset options = StructNew()>
<cfset options["saveData"] = false>

<cfreturn view("blog_view", data, options)>
```

Additionally, if you would like the default functionality of the `view()` function to be that it does clear the data
between calls, you can set `saveData` to `false` in **app/Config/View.cfc**. It is `true` in a new application.

## Creating Loops

The data you pass to your view files is not limited to simple variables. You can pass lists (arrays)
and structs, which can be looped to generate multiple rows. For example, if you pull data from your database it will
typically be in the form of a list of rows.

Here's a simple example. Add this to your controller:

```cfm
<cfcomponent output="no" extends="BaseController">

	<cffunction name="index" access="public" returntype="string" output="no">
		<cfset var data = StructNew()>
		<cfset var todo = ArrayNew(1)>

		<cfset ArrayAppend(todo, "Clean House")>
		<cfset ArrayAppend(todo, "Call Mom")>
		<cfset ArrayAppend(todo, "Run Errands")>

		<cfset data["todo_list"] = todo>
		<cfset data["title"] = "My Real Title">
		<cfset data["heading"] = "My Real Heading">

		<cfreturn view("blog_view", data)>
	</cffunction>

</cfcomponent>
```

Now open your view file and create a loop:

```cfm
<cfoutput>
<html>
    <head>
        <title>#esc(title)#</title>
    </head>
    <body>
        <h1>#esc(heading)#</h1>

        <h3>My Todo List</h3>

        <ul>
        <cfloop from="1" to="#ArrayLen(todo_list)#" index="i">
            <li>#esc(todo_list[i])#</li>
        </cfloop>
        </ul>
    </body>
</html>
</cfoutput>
```
