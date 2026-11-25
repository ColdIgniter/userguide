# URL Helper

The URL Helper file contains functions that assist in working with URLs.

## Loading this Helper

This helper is automatically loaded by the framework on every request.

## Available Functions

The following functions are available:

### site_url()

`site_url(relativePath, scheme, config_)`

- `relativePath` (string or array, optional, default `""`): a URI string or an array of URI segments.
- `scheme` (string, optional): the protocol, such as `http` or `https`. If an empty string `""` is set, a protocol-relative link is returned. Leave it out for the protocol of your site.
- `config_` (`Config\App` component, optional): an alternate configuration to use.
- Returns the site URL.

!!! note
    If you set `allowedHostnames` in `app/Config/App.cfc`, this returns the URL with the hostname set in it if the current URL matches.

Returns your site URL, as specified in your config file. The `index.cfm` file (or whatever you have set as the `indexPage` of your site in `app/Config/App.cfc`) is added to the URL, as are any URI segments you pass to the function.

You are encouraged to use this function any time you need to generate a local URL so that your pages become more portable in the event your URL changes.

Segments can be optionally passed to the function as a string or an array. Here is a string example:

```cfm
<cfoutput>#site_url("news/local/123")#</cfoutput>
```

The above example would return something like: **http://example.com/index.cfm/news/local/123**

Here is an example of segments passed as an array:

```cfm
<cfset segments = ListToArray("news,local,123")>
<cfoutput>#site_url(segments)#</cfoutput>
```

You may find the alternate configuration useful if generating URLs for a different site than yours, which contains different configuration preferences.

!!! note "Difference from CodeIgniter"
    CodeIgniter takes `null` for "no protocol given" and "no alternate configuration". Here the argument is left out. To pass the configuration without a protocol, name the arguments: `site_url(relativePath="news", config_=myConfig)`.

### base_url()

`base_url(relativePath, scheme)`

- `relativePath` (string or array, optional, default `""`): a URI string or an array of URI segments.
- `scheme` (string, optional): the protocol, such as `http` or `https`. If an empty string `""` is set, a protocol-relative link is returned. Leave it out for the protocol of your site.
- Returns the base URL.

!!! note
    If you set `allowedHostnames` in `app/Config/App.cfc`, this returns the URL with the hostname set in it if the current URL matches.

Returns your site base URL, as specified in your config file. Example:

```cfm
<cfoutput>#base_url()#</cfoutput>
```

This function returns the same thing as [`site_url()`](#site_url), without the `indexPage` being appended.

Also like [`site_url()`](#site_url), you can supply segments as a string or an array. Here is a string example:

```cfm
<!--- Returns like `http://example.com/blog/post/123` --->
<cfoutput>#base_url("blog/post/123")#</cfoutput>
```

The above example would return something like: **http://example.com/blog/post/123**

If you pass an empty string `""` as the second parameter, it returns the protocol-relative link:

```cfm
<!--- Returns like `//example.com/blog/post/123` --->
<cfoutput>#base_url("blog/post/123", "")#</cfoutput>
```

This is useful because unlike [`site_url()`](#site_url), you can supply a string to a file, such as an image or stylesheet. For example:

```cfm
<cfoutput>#base_url("images/icons/edit.png")#</cfoutput>
```

This would give you something like: **http://example.com/images/icons/edit.png**

### current_url()

`current_url(returnObject, request_)`

- `returnObject` (boolean, optional, default `false`): `true` if you would like a URI instance returned, instead of a string.
- `request_` (request component, optional, default `""`): an alternate request to use for path detection. Useful for testing.
- Returns the current URL, as a string or as a URI instance.

Returns the full URL of the page being currently viewed. When returning a string, the query and fragment parts of the URL are removed. When returning a URI, the query and fragment parts are preserved.

However for security reasons, it is created based on the `Config\App` settings, and not intended to match the browser URL.

If you set `allowedHostnames` in `app/Config/App.cfc`, this returns the URL with the hostname set in it if the current URL matches.

!!! note
    Calling `current_url()` is the same as doing this:

    ```cfm
    <cfset currentUrl = site_url(uri_string())>
    ```

### previous_url()

`previous_url(returnObject)`

- `returnObject` (boolean, optional, default `false`): `true` if you would like a URI instance returned instead of a string.
- Returns the URL the user was previously on, as a string or as a URI instance.

Returns the full URL (including segments) of the page the user was previously on.

!!! note "Difference from CodeIgniter"
    CodeIgniter takes the previous page from the session when there is one, and else a sanitized version of `HTTP_REFERER`. Both are not ported yet. ColdIgniter uses the `HTTP_REFERER` value as the browser sent it, and the site URL when there is none. Do not trust this value: the browser can send anything.

### uri_string()

`uri_string()`

- Returns a URI string.

Returns the path part of the current URL relative to baseURL.

For example, when your baseURL is **http://some-site.com/** and the current URL is:

```html
http://some-site.com/blog/comments/123
```

The function would return:

```html
blog/comments/123
```

When your baseURL is **http://some-site.com/subfolder/** and the current URL is:

```html
http://some-site.com/subfolder/blog/comments/123
```

The function would return:

```html
blog/comments/123
```

When you navigate to the baseURL, this function returns an empty string.

### index_page()

`index_page(altConfig)`

- `altConfig` (`Config\App` component, optional): an alternate configuration to use.
- Returns the `indexPage` value.

Returns your site **indexPage**, as specified in your config file. Example:

```cfm
<cfoutput>#index_page()#</cfoutput>
```

As with [`site_url()`](#site_url), you may specify an alternate configuration. You may find the alternate configuration useful if generating URLs for a different site than yours, which contains different configuration preferences.

### anchor()

`anchor(uri, title, attributes, altConfig)`

- `uri` (string or array, optional, default `""`): a URI string or an array of URI segments.
- `title` (string, optional, default `""`): the anchor title.
- `attributes` (string or struct, optional, default `""`): the HTML attributes.
- `altConfig` (`Config\App` component, optional): an alternate configuration to use.
- Returns an HTML hyperlink (anchor tag).

Creates a standard HTML anchor link based on your local site URL.

The first parameter can contain any segments you wish appended to the URL. As with the [`site_url()`](#site_url) function above, segments can be a string or an array.

!!! note
    If you are building links that are internal to your application do not include the base URL (`http://...`). This is added automatically from the information specified in your config file. Include only the URI segments you wish appended to the URL.

The second segment is the text you would like the link to say. If you leave it blank, the URL is used.

The third parameter can contain a list of attributes you would like added to the link. The attributes can be a simple string or a struct.

Here are some examples:

```cfm
<cfoutput>#anchor("news/local/123", "My News", "title=""News title""")#</cfoutput>
<!--- Prints: <a href="http://example.com/index.cfm/news/local/123" title="News title">My News</a> --->

<cfset attributes = StructNew()>
<cfset attributes["title"] = "The best news!">
<cfoutput>#anchor("news/local/123", "My News", attributes)#</cfoutput>
<!--- Prints: <a href="http://example.com/index.cfm/news/local/123" title="The best news!">My News</a> --->

<cfoutput>#anchor("", "Click here")#</cfoutput>
<!--- Prints: <a href="http://example.com/index.cfm">Click here</a> --->
```

As above, you may specify an alternate configuration. You may find the alternate configuration useful if generating links for a different site than yours, which contains different configuration preferences.

!!! note
    Attributes passed into the anchor function as a struct are automatically escaped to protect against XSS attacks.

### anchor_popup()

`anchor_popup(uri, title, attributes, altConfig)`

- `uri` (string, optional, default `""`): a URI string.
- `title` (string, optional, default `""`): the anchor title.
- `attributes` (string, struct or array, optional): the HTML attributes. Leave it out, or pass `false`, for a plain new window.
- `altConfig` (`Config\App` component, optional): an alternate configuration to use.
- Returns a pop-up hyperlink.

Nearly identical to the [`anchor()`](#anchor) function except that it opens the URL in a new window. You can specify JavaScript window attributes in the third parameter to control how the window is opened. If the third parameter is not set it simply opens a new window with your own browser settings.

Here is an example with attributes:

```cfm
<cfset atts = StructNew()>
<cfset atts["width"] = 800>
<cfset atts["height"] = 600>
<cfset atts["scrollbars"] = "yes">
<cfset atts["status"] = "yes">
<cfset atts["resizable"] = "yes">
<cfset atts["screenx"] = 0>
<cfset atts["screeny"] = 0>
<cfset atts["window_name"] = "_blank">

<cfoutput>#anchor_popup("news/local/123", "Click Me!", atts)#</cfoutput>
```

As above, you may specify an alternate configuration. You may find the alternate configuration useful if generating links for a different site than yours, which contains different configuration preferences.

!!! note
    The above attributes are the function defaults (together with `menubar`, which is `no`), so you only need to set the ones that are different from what you need. If you want the function to use all of its defaults simply pass an empty struct in the third parameter:

    ```cfm
    <cfoutput>#anchor_popup("news/local/123", "Click Me!", StructNew())#</cfoutput>
    ```

!!! note
    The **window_name** is not really an attribute, but an argument to the JavaScript [window.open()](https://www.w3schools.com/jsref/met_win_open.asp) method, which accepts either a window name or a window target.

!!! note
    Any other attribute than the ones listed above is parsed as an HTML attribute to the anchor tag.

!!! note
    Attributes passed into the anchor_popup function are automatically escaped to protect against XSS attacks.

### mailto()

`mailto(email, title, attributes)`

- `email` (string): the e-mail address.
- `title` (string, optional, default `""`): the anchor title.
- `attributes` (string or struct, optional, default `""`): the HTML attributes.
- Returns a "mail to" hyperlink.

Creates a standard HTML e-mail link. Usage example:

```cfm
<cfoutput>#mailto("me@my-site.com", "Click Here to Contact Me")#</cfoutput>
```

As with the [`anchor()`](#anchor) function above, you can set attributes using the third parameter:

```cfm
<cfset attributes = StructNew()>
<cfset attributes["title"] = "Mail me">
<cfoutput>#mailto("me@my-site.com", "Contact Me", attributes)#</cfoutput>
```

!!! note
    Attributes passed into the mailto function as a struct are automatically escaped to protect against XSS attacks.

### safe_mailto()

`safe_mailto(email, title, attributes)`

- `email` (string): the e-mail address.
- `title` (string, optional, default `""`): the anchor title.
- `attributes` (string or struct, optional, default `""`): the HTML attributes.
- Returns a spam-safe "mail to" hyperlink.

Identical to the [`mailto()`](#mailto) function except it writes an obfuscated version of the *mailto* tag using ordinal numbers written with JavaScript to help prevent the e-mail address from being harvested by spam bots.

### auto_link()

`auto_link(str, type, popup)`

- `str` (string): the input string.
- `type` (string, optional, default `both`): the link type (`email`, `url` or `both`).
- `popup` (boolean, optional, default `false`): whether to create popup links.
- Returns the linkified string.

Automatically turns URLs and e-mail addresses contained in a string into links. Example:

```cfm
<cfset string = auto_link(string)>
```

The second parameter determines whether URLs and e-mails are converted or just one or the other. The default behavior is both if the parameter is not specified. E-mail links are encoded as [`safe_mailto()`](#safe_mailto) as shown above.

Converts only URLs:

```cfm
<cfset string = auto_link(string, "url")>
```

Converts only e-mail addresses:

```cfm
<cfset string = auto_link(string, "email")>
```

The third parameter determines whether links are shown in a new window. The value can be true or false (boolean):

```cfm
<cfset string = auto_link(string, "both", true)>
```

!!! note
    The only URLs recognized are those that start with `www.` or with `://`.

!!! note "Difference from CodeIgniter"
    CodeIgniter checks each e-mail address it finds and links only a valid one. That check is not ported yet, so every text that looks like an e-mail address is linked. A URL or an e-mail address that is glued to the text before it (as in `1http://x.example`) is not linked.

### url_title()

`url_title(str, separator, lowercase)`

- `str` (string): the input string.
- `separator` (string, optional, default `-`): the word separator (usually `-` or `_`).
- `lowercase` (boolean, optional, default `false`): whether to transform the output string to lowercase.
- Returns a URL-formatted string.

Takes a string as input and creates a human-friendly URL string. This is useful if, for example, you have a blog in which you'd like to use the title of your entries in the URL. Example:

```cfm
<cfset title = "What's wrong with CSS?">
<cfset urlTitle = url_title(title)>
<!--- Produces: Whats-wrong-with-CSS --->
```

The second parameter determines the word delimiter. By default dashes are used. Preferred options are: `-` (dash) or `_` (underscore).

Example:

```cfm
<cfset title = "What's wrong with CSS?">
<cfset urlTitle = url_title(title, "_")>
<!--- Produces: Whats_wrong_with_CSS --->
```

The third parameter determines whether or not lowercase characters are forced. By default they are not. Options are boolean true/false.

Example:

```cfm
<cfset title = "What's wrong with CSS?">
<cfset urlTitle = url_title(title, "-", true)>
<!--- Produces: whats-wrong-with-css --->
```

### mb_url_title()

`mb_url_title(str, separator, lowercase)`

- `str` (string): the input string.
- `separator` (string, optional, default `-`): the word separator (usually `-` or `_`).
- `lowercase` (boolean, optional, default `false`): whether to transform the output string to lowercase.
- Returns a URL-formatted string.

This function works the same as [`url_title()`](#url_title).

!!! warning "Not ported yet"
    In CodeIgniter this function also converts all accented characters. That part is not ported yet.

### parse_subdomain()

!!! warning "Not ported yet"
    `parse_subdomain()` is not ported yet.

### prep_url()

`prep_url(str, secure)`

- `str` (string, optional, default `""`): the URL string.
- `secure` (boolean, optional, default `false`): `true` for `https://`.
- Returns a protocol-prefixed URL string.

This function adds `http://` or `https://` in the event that a protocol prefix is missing from a URL.

Pass the URL string to the function like this:

```cfm
<cfset address = prep_url("example.com")>
```

### url_to()

`url_to(controller, args)`

- `controller` (string): the route name or `Controller::method`.
- `args` (strings or numbers): one or more parameters to be passed to the route.
- Returns an absolute URL.

!!! note
    This function requires the controller/method to have a route defined in `app/Config/Routes.cfm`.

Builds an absolute URL to a controller method in your app. Example:

```cfm
<!--- The route is defined as: --->
<cfset routes.get("/", "Home::index")>
```

```cfm
<a href="<cfoutput>#url_to("Home::index")#</cfoutput>">Home</a>
```

You can also add arguments to the route. Here is an example:

```cfm
<!--- The route is defined as: --->
<cfset routes.get("pages/(:segment)", "Page::index/$1")>
```

```cfm
<a href="<cfoutput>#url_to("Page::index", "home")#</cfoutput>">Home</a>
<!--- Result: 'http://example.com/index.cfm/pages/home' --->
```

This is useful because you can still change your routes after putting links into your views.

If no route is found, a `RouterException` is thrown.

!!! warning "Not ported yet"
    In CodeIgniter, when you use `{locale}` in your route, you can give the locale value as the last parameter. That is not ported yet.

For full details, see [Reverse Routing](../incoming/routing.md#reverse-routing) and [Named Routes](../incoming/routing.md#named-routes).

### url_is()

`url_is(path)`

- `path` (string): the URL path relative to baseURL to check the current URI path against.
- Returns `true` or `false`.

Compares the path of the current URL against the given path to see if they match. Example:

```cfm
<cfif url_is("admin")>
	<!--- ... --->
</cfif>
```

This would match **http://example.com/admin**. It would match **http://example.com/subdir/admin** if your baseURL is `http://example.com/subdir/`.

You can use the `*` wildcard to match any other applicable characters in the URL:

```cfm
<cfif url_is("admin*")>
	<!--- ... --->
</cfif>
```

This would match any of the following:

- /admin
- /admin/
- /admin/users
- /admin/users/schools/classmates/...
