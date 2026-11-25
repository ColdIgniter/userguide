# View Parser

The View Parser can perform simple text substitution for
pseudo-variables contained within your view files.
It can parse simple variables or variable tag pairs.

Pseudo-variable names or control constructs are enclosed in braces, like this:

```html
<html>
<head>
    <title>{blog_title}</title>
</head>
<body>
    <h3>{blog_heading}</h3>

    {blog_entries}
        <h5>{title}</h5>
        <p>{body}</p>
    {/blog_entries}

</body>
</html>
```

These variables are not actual CFML variables, but rather plain text
representations that allow you to eliminate CFML from your templates
(view files). The template is read as plain text: CFML tags in it are not run.

!!! note
    ColdIgniter does **not** require you to use this class since
    using plain CFML in your view pages (for instance using the
    [View renderer](view_renderer.md))
    lets them run a little faster.
    However, some developers prefer to use some form of template engine if
    they work with designers who they feel would find some
    confusion working with CFML.

## Using the View Parser Class

The simplest method to load the parser class is through its service:

```cfm
<cfset parser = service("parser")>
```

Alternately, you can ask the service for a new parser that is not shared. The arguments are the view path, the config and `false`
for "not shared" (an empty view path and an empty config mean the defaults):

```cfm
<cfset parser = service("parser", viewPath, "", false)>
```

Then you can use any of the three standard rendering methods that it provides:
`render()`, `setVar()` and `setData()`. The Parser also has a public `parse(template, data)` method that
parses a string you give it.

!!! warning "Not ported yet"
    Setting the delimiters directly through `setDelimiters()` is not ported (**system/View/Parser.php**). The braces `{` and `}` are fixed.

!!! important
    The `Parser` never runs CFML tags in a template, and literal `<?` and `?>` are turned into entities.
    Conditions in `{ if }` and `{ elseif }` tags are read by a small evaluator and are never run as code, so they are safe for a template that
    less-trusted users edit.

!!! note "Difference from CodeIgniter"
    In CodeIgniter the conditions are run as PHP code, and the guide tells you to restrict them. Here the evaluator is always restricted, so
    there is no `restrictConditionals` option.

### What It Does

The `Parser` class processes templates stored in the application's view path.
Literal CFML tags are not supported.

Each view parameter (which we refer to as a pseudo-variable) triggers a substitution,
based on the type of value you provided for it. Pseudo-variables are not
made into CFML variables; instead their value is accessed through the pseudo-variable
syntax, where its name is referenced inside braces.

The Parser class uses a struct internally, to accumulate pseudo-variable
settings until you call its `render()`. This means that your pseudo-variable names
need to be unique, or a later parameter setting will over-ride an earlier one.

### Parser Templates

You can use the `render()` method to parse (or render) simple templates,
like this:

```cfm
<cfset parser = service("parser")>

<cfset data = StructNew()>
<cfset data["blog_title"] = "My Blog Title">
<cfset data["blog_heading"] = "My Blog Heading">

<cfreturn parser.setData(data).render("blog_template")>
```

View parameters are passed to `setData()` as a struct
of data to be replaced in the template.

In the above example, the template would contain two variables: `{blog_title}` and `{blog_heading}`

The first parameter to `render()` contains the name of the template, where
`blog_template` is the name of your view template file.

!!! important
    If the file extension is omitted, then the views are expected to end with the **.cfm** extension.

### Parser Configuration Options

The `render()` method takes a struct of options as its second argument, and `saveData` as its third.

-   `saveData` - true if the view data parameters should be retained for subsequent calls;
    the default is the `saveData` setting of **app/Config/View.cfc**

!!! warning "Not ported yet"
    The options `cache`, `cache_name`, `cascadeData` and `restrictConditionals` are not ported (**system/View/Parser.php**). The `renderString()` method they also apply to is not ported either.

## Substitution Variations

There are two types of substitution supported: simple and looping.
Substitutions are performed in the same sequence that pseudo-variables were added.

The **simple substitution** performed by the parser is a one-to-one
replacement of pseudo-variables where the corresponding data parameter
has a simple value (a string or a number), as in this example:

```cfm
<cfset data = StructNew()>
<cfset data["blog_title"] = "My Blog Title">
<cfset data["blog_heading"] = "My Blog Heading">

<cfreturn service("parser").setData(data).render("blog_template")>
```

Substitution values are treated as data, not as template code. Any Parser
syntax within a value is rendered literally and is not processed by later
substitutions.

The `Parser` takes substitution a lot further with "variable pairs",
used for looping, and with some advanced
constructs for conditional substitution.

When the parser executes, it will generally

- handle any conditional substitutions
- handle any looping substitutions
- handle the remaining single substitutions

A value in braces is escaped for HTML. To show a value as it is, write it between exclamation marks, as in `{! blog_body !}`, see [Escaping Data](#escaping-data).
Spaces just inside the braces are ignored, so `{ blog_title }` and `{blog_title}` are the same.

### Loop Substitutions

A loop substitution happens when the value for a pseudo-variable is
a list (an array) of structs, like a list of row settings.

The above example code allows simple variables to be replaced. What if
you would like an entire block of variables to be repeated, with each
iteration containing new values? Consider the template example we showed
at the top of the page:

```html
<html>
<head>
    <title>{blog_title}</title>
</head>
<body>
    <h3>{blog_heading}</h3>

    {blog_entries}
        <h5>{title}</h5>
        <p>{body}</p>
    {/blog_entries}

</body>
</html>
```

In the above code you'll notice a pair of variables: `{blog_entries}`
data... `{/blog_entries}`. In a case like this, the entire chunk of data
between these pairs would be repeated multiple times, corresponding to
the number of rows in the "blog_entries" element of the data.

Parsing variable pairs is done using the identical code shown above to
parse single variables, except, you will add a list of structs
corresponding to your variable pair data. Consider this example:

```cfm
<cfset data = StructNew()>
<cfset data["blog_title"] = "My Blog Title">
<cfset data["blog_heading"] = "My Blog Heading">
<cfset data["blog_entries"] = ArrayNew(1)>

<cfset entry = StructNew()>
<cfset entry["title"] = "Title 1">
<cfset entry["body"] = "Body 1">
<cfset ArrayAppend(data["blog_entries"], entry)>

<cfset entry = StructNew()>
<cfset entry["title"] = "Title 2">
<cfset entry["body"] = "Body 2">
<cfset ArrayAppend(data["blog_entries"], entry)>

<cfreturn service("parser").setData(data).render("blog_template")>
```

The value for the pseudo-variable `blog_entries` is a list of structs.
The outer level does not have keys associated with each of the nested "rows".

A struct whose values are structs also works: each value is one row. Any other value in a pair is skipped.

Inside the pair, only the data of the row is known. The values of the outer template are not passed in (see [Cascading Data](#cascading-data)).

If your "pair" data is coming from a database result, make a list of structs from it first, as your model returns it.

!!! warning "Not ported yet"
    Looping over a list of objects (the `asArray()` method of an object, and the Entity classes, in **system/View/Parser.php**) is not ported. Pass structs.

### Nested Substitutions

!!! warning "Not ported yet"
    A nested substitution, where the value is a struct of simple values (a record) that is read inside a pair of the same name, is not ported (**system/View/Parser.php**). A pair works with a list of structs, or a struct of structs, only.

### Comments

You can place comments in your templates that will be ignored and removed during parsing by wrapping the
comments in a `{#  #}` symbols.

```text
{# This comment is removed during parsing. #}
{blog_entries}
    <div>
        <h2>{title}</h2>
        <p>{body}</p>
    </div>
{/blog_entries}
```

### Cascading Data

!!! warning "Not ported yet"
    The `cascadeData` option is not ported (**system/View/Parser.php**). The row of a pair never sees the other values of the template.

### Preventing Parsing

You can specify portions of the page to not be parsed with the `{noparse}` `{/noparse}` tag pair. Anything in this
section will stay exactly as it is, with no variable substitution, looping, etc, happening to the markup between the brackets.

```text
{noparse}
    <h1>Untouched Code</h1>
{/noparse}
```

### Conditional Logic

The Parser class supports some basic conditionals to handle `if`, `else`, and `elseif` syntax. All `if`
blocks must be closed with an `endif` tag. Blocks can be nested:

```text
{if $role == 'admin'}
    <h1>Welcome, Admin!</h1>
{endif}
```

All variables used within if statements must have been previously set with the same name, and are written with a leading `$`.
A condition is made of one or two operands. An operand is a variable (`$name`), a string in quotes, a number, `true`, `false` or `[]` (an empty list).
The comparison operators are `==`, `!=`, `===` and `!==`. An operand on its own is true unless it is an empty string, `0`, `false`, or an empty list or struct.

```text
{if $role == 'admin'}
    <h1>Welcome, Admin</h1>
{elseif $role == 'moderator'}
    <h1>Welcome, Moderator</h1>
{else}
    <h1>Welcome, User</h1>
{endif}
```

Values passed through `setData()` can control a conditional in a trusted or an untrusted template:

```cfm
<cfset data = StructNew()>
<cfset data["role"] = "admin">

<cfreturn service("parser").setData(data).render("welcome")>
```

The value of `$role` is checked by the condition; it is never run as code.

!!! note "Difference from CodeIgniter"
    Only the four comparison operators above exist. There are no `<`, `>`, `&&`, `||`, arithmetic or function calls. A condition inside a pair is checked with the data of the whole template, not of the row.

#### Restricting Conditionals

!!! warning "Not ported yet"
    The `restrictConditionals` option and the `restrictParserConditionals` setting (CodeIgniter 4.7.5, **system/View/Parser.php**) have no effect. They are not needed, because conditionals are never run as code.

#### Changing the Conditional Delimiters

!!! warning "Not ported yet"
    `setConditionalDelimiters()` is not ported (**system/View/Parser.php**). Conditionals always use `{ if }`. To keep JavaScript with braces from being read as a tag, put it between `{noparse}` and `{/noparse}`.

### Escaping Data

Variable substitutions are escaped for HTML by default to help prevent XSS attacks on your pages.
To disable automatic escaping for a substitution, add exclamation marks to its delimiters:

```text
{! unescaped_var !}
```

!!! warning "Not ported yet"
    Choosing another context with the `esc` filter, as in `{ user_link | esc(attr) }`, is not ported, because filters are not ported (see [Filters](#filters)). Escape the value in the controller with `esc(value, "attr")` and use `{! name !}`.

### Filters

!!! warning "Not ported yet"
    Filters (`{ value|filter }`, the `filters` setting of **app/Config/View.cfc**, and the provided filters, in **system/View/Parser.php** and **system/View/Filters.php**) are not ported. The setting is listed in the config, but the code ignores it.

#### Provided Filters

!!! warning "Not ported yet"
    See [Filters](#filters): none of the provided filters (abs, capitalize, date, date_modify, default, esc, excerpt, highlight, highlight_code, limit_chars, limit_words, local_currency, local_number, lower, nl2br, number_format, prose, round, strip_tags, title, upper) exists.

#### Custom Filters

!!! warning "Not ported yet"
    See [Filters](#filters): custom filters cannot be added.

### Parser Plugins

!!! warning "Not ported yet"
    Plugins (`{+ name +}` tags, the `plugins` setting of **app/Config/View.cfc**, and the provided plugins, in **system/View/Parser.php** and **system/View/Plugins.php**) are not ported. The setting is listed in the config, but the code ignores it. A `{+ ... +}` tag stays in the output as written.

#### Provided Plugins

!!! warning "Not ported yet"
    See [Parser Plugins](#parser-plugins): none of the provided plugins (current_url, previous_url, siteURL, mailto, safe_mailto, lang, validation_errors, route, csp_script_nonce, csp_style_nonce) exists. The CSP nonce placeholders still work, see [Content Security Policy](csp.md).

#### Registering a Plugin

!!! warning "Not ported yet"
    See [Parser Plugins](#parser-plugins): plugins cannot be registered.

## Usage Notes

If you include substitution parameters that are not referenced in your
template, they are ignored.

If you do not include a substitution parameter that is referenced in your
template, the original pseudo-variable is shown in the result. This is also true for a value that is a list or a struct
used as a single value.

If you provide a simple value when a list is expected, i.e., for a variable pair, the pair is skipped and the tags stay in the result as written.

### View Fragments

You do not have to use variable pairs to get the effect of iteration in
your views. It is possible to use a view fragment for what would be inside
a variable pair, and to control the iteration in your controller instead
of in the view.

An example with the iteration controlled in the view:

```cfm
<cfset template = "<ul>{menuitems}<li><a href=""{! link !}"">{ title }</a></li>{/menuitems}</ul>">

<cfset data = StructNew()>
<cfset data["menuitems"] = ArrayNew(1)>

<cfset item = StructNew()>
<cfset item["title"] = "First Link">
<cfset item["link"] = "/first">
<cfset ArrayAppend(data["menuitems"], item)>

<cfset item = StructNew()>
<cfset item["title"] = "Second Link">
<cfset item["link"] = "/second">
<cfset ArrayAppend(data["menuitems"], item)>

<cfreturn service("parser").parse(template, data)>
```

Result:

```html
<ul><li><a href="/first">First Link</a></li><li><a href="/second">Second Link</a></li></ul>
```

An example with the iteration controlled in the controller, using a view fragment. `items` is a list of structs with the keys `title` and `link`:

```cfm
<cfset parser = service("parser")>
<cfset fragment = "<li><a href=""{! link !}"">{ title }</a></li>">
<cfset menu = "">
<cfset data = StructNew()>

<cfloop from="1" to="#ArrayLen(items)#" index="i">
	<cfset menu = menu & parser.parse(fragment, items[i])>
</cfloop>

<cfset data["menuitems"] = menu>

<cfreturn parser.parse("<ul>{! menuitems !}</ul>", data)>
```

Result:

```html
<ul><li><a href="/first">First Link</a></li><li><a href="/second">Second Link</a></li></ul>
```

## Class Reference

### render

`render(view [, options [, saveData]])`

- `view` (string): file name of the view source
- `options` (struct): options, as key/value pairs
- `saveData` (boolean): if true, will save data for use with any other calls, if false, will clean the data after rendering the view
- Returns the rendered text for the chosen view (string)

Builds the output based upon a file name and any data that has already been set:

```cfm
<cfreturn service("parser").setData(data).render("myview")>
```

Any conditional substitutions are performed first, then pairs, then the remaining single substitutions.

### renderString

!!! warning "Not ported yet"
    `renderString()` is not ported. Use `parse(template, data)` for a template that is a string.

### parse

`parse(template, data)`

- `template` (string): the template text
- `data` (struct): the values to put in
- Returns the parsed text (string)

Parses a string with the data you give, without using the data set before:

```cfm
<cfreturn service("parser").parse("<p>{ name }</p>", data)>
```

### setData

`setData([data [, context]])`

- `data` (struct): view data as key/value pairs
- `context` (string): accepted, but not used yet (see [View Renderer](view_renderer.md))
- Returns the parser, for method chaining

Sets several pieces of view data at once:

```cfm
<cfset parser.setData(data)>
```

### setVar

`setVar(name [, value [, context]])`

- `name` (string): name of the view data variable
- `value` (any): the value of this view data
- `context` (string): accepted, but not used yet
- Returns the parser, for method chaining

Sets a single piece of view data:

```cfm
<cfset parser.setVar("name", "Joe")>
```

### setDelimiters

!!! warning "Not ported yet"
    `setDelimiters()` is not ported.

### setConditionalDelimiters

!!! warning "Not ported yet"
    `setConditionalDelimiters()` is not ported.
