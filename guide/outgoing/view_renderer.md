# View Renderer

## Using the View Renderer

The `view()` function is a convenience function that grabs an instance of the
`renderer` service, sets the data, and renders the view. While this is often
exactly what you want, you may find times where you want to work with it more directly.
In that case you can access the View service directly:

```cfm
<cfset renderer = service("renderer")>
```

Alternately, you can ask the service for a new renderer that is not shared, for example with another view folder.
The arguments are the view path, the config and `false` for "not shared" (an empty view path and an empty config mean the defaults):

```cfm
<cfset renderer = service("renderer", viewPath, "", false)>
```

!!! important
    You should create services only within controllers. If you need
    access to the View class from a library, you should set that as a dependency
    in your library's constructor.

Then you can use any of the three standard methods that it provides:
`render()`, `setVar()` and `setData()`.

!!! warning "Not ported yet"
    `renderString()`, which renders a piece of a view from a string, is not ported (`View::renderString()` in CodeIgniter).

## What It Does

The `View` class processes the view templates stored in the application's view path
(**app/Views**). Each view parameter becomes a variable that you can read inside the template.
This means that your view parameter names need to be legal CFML variable names.

The View class uses a struct internally, to accumulate view parameters
until you call its `render()`. This means that your parameter (or variable) names
need to be unique, or a later variable setting will over-ride an earlier one.

This also impacts escaping parameter values for different contexts inside your
template. You will have to give each escaped value a unique parameter name.

No special meaning is attached to parameters whose value is a list or a struct. It is up
to you to process them appropriately in your template.

## Setting View Parameters

The `setVar()` method sets a view parameter.

```cfm
<cfset renderer = service("renderer")>
<cfset renderer.setVar("name", "Joe")>
```

The `setData()` method sets multiple view parameters at once.

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "Joe">
<cfset data["city"] = "Example Town">

<cfset renderer.setData(data)>
```

## Method Chaining

The `setVar()` and `setData()` methods are chainable, allowing you to combine a
number of different calls together in a chain:

```cfm
<cfreturn service("renderer").setVar("one", "foo").setVar("two", "bar").render("myview")>
```

## Escaping Data

!!! warning "Not ported yet"
    Escaping in `setVar()` and `setData()` through the last parameter (the `context`) is not ported (`esc(data, context)`). The parameter is accepted and ignored: the data is never escaped when it is set.

Escape the data inside the view with the `esc()` function. The first parameter is the value to escape. The second parameter is the
context to escape the data for (see below). Lists and structs are escaped value by value:

```cfm
<cfoutput>#esc(object.getStat())#</cfoutput>
```

### Escaping Contexts

By default, the `esc()` function assumes that the data you want to
escape is intended to be used within standard HTML. However, if the data is intended for use in Javascript
or in an href attribute, you would need different escaping rules to be effective. You can pass in the name of the
context as the second parameter. Valid contexts are `"html"`, `"js"`, `"url"` and `"attr"`. The context `"raw"` returns the data as it is:

```cfm
<cfoutput>
<a href="#esc(link, "url")#" data-foo="#esc(bar, "attr")#">Some Link</a>

<script>
    var siteName = '#esc(siteName, "js")#';
</script>
</cfoutput>
```

!!! note "Difference from CodeIgniter"
    There is no `css` context. An unknown context throws an `InvalidArgumentException`.

## View Renderer Options

Several options can be passed in a struct to the `render()` method:

- `debug` - can be set to false to disable the addition of debug code for
  the Debug Toolbar.

The third argument of `render()`, `saveData`, is true if the view data parameters should be retained for
subsequent calls. It is not a key of the options struct. Only `view()` takes a `saveData` key in its options, and it passes it on as this argument.

!!! warning "Not ported yet"
    The options `cache` and `cache_name` are not ported. They are ignored.

## Class Reference

### render

`render(view [, options [, saveData]])`

- `view` (string): file name of the view source
- `options` (struct): options, as key/value pairs
- `saveData` (boolean): if true, will save data for use with any other calls. If false, will clean the data after rendering the view. If left out, uses the config setting.
- Returns the rendered text for the chosen view (string)

Builds the output based upon a file name and any data that has already been set:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "George">
<cfset data["email"] = "george@example.com">

<cfset renderer.setData(data)>

<cfreturn renderer.render("myview")>
```

### renderString

!!! warning "Not ported yet"
    `renderString()` is not ported.

### setData

`setData([data [, context]])`

- `data` (struct): view data as key/value pairs
- `context` (string): accepted, but not used yet (see [Escaping Data](#escaping-data))
- Returns the renderer, for method chaining

Sets several pieces of view data at once:

```cfm
<cfset renderer.setData(data)>
```

Each call adds to the struct of data that the object is accumulating,
until the view is rendered.

### setVar

`setVar(name [, value [, context]])`

- `name` (string): name of the view data variable
- `value` (any): the value of this view data
- `context` (string): accepted, but not used yet
- Returns the renderer, for method chaining

Sets a single piece of view data:

```cfm
<cfset renderer.setVar("name", "Joe")>
```

If you use a view data variable that you have previously used
for this object, the new value will replace the existing one.
