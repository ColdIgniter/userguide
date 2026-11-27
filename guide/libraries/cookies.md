# Cookies

An **HTTP cookie** (web cookie, browser cookie) is a small piece of data that a server sends to the user's web browser. The browser may store it and send it back with later requests to the same server. Typically, it's used to tell if two requests came from the same browser - keeping a user logged-in, for example. It remembers stateful information for the stateless HTTP protocol.

Cookies are mainly used for three purposes:

- **Session management**: Logins, shopping carts, game scores, or anything else the server should remember
- **Personalization**: User preferences, themes, and other settings
- **Tracking**: Recording and analyzing user behavior

To help you efficiently send cookies to browsers, ColdIgniter provides the `ColdIgniter\Cookie\Cookie` class to abstract the cookie interaction.

!!! note "Difference from CodeIgniter"
    The options of a cookie are a struct, made with `StructNew()`. The keys are in lower case: `prefix`, `max-age`, `expires`, `path`, `domain`, `secure`, `httponly`, `samesite` and `raw`. The expiry time is a Unix timestamp (seconds since 1 January 1970), see [Cookie Personalization](#cookie-personalization).

## Creating Cookies

There are currently four ways to create a new `Cookie` value object.

```cfm
<!--- Using the constructor --->
<cfset options = StructNew()>
<cfset options["max-age"] = 7200>
<cfset options["prefix"] = "__Secure-">
<cfset options["path"] = "/">
<cfset options["domain"] = "">
<cfset options["secure"] = true>
<cfset options["httponly"] = true>
<cfset options["raw"] = false>
<cfset options["samesite"] = "Lax">

<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "remember_token", "f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6", options)>

<!--- Supplying a Set-Cookie header string --->
<cfset cookieClass = static_("ColdIgniter\Cookie\Cookie")>
<cfset myCookie = cookieClass.fromHeaderString("remember_token=f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6; Path=/; Secure; HttpOnly; SameSite=Lax", false)>

<!--- Using the fluent builder interface --->
<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "remember_token")>
<cfset myCookie = myCookie.withValue("f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6").withPrefix("__Secure-").withPath("/").withDomain("").withSecure(true).withHTTPOnly(true).withSameSite("Lax")>

<!--- Using the global function `cookie_`, which implicitly calls new_() --->
<cfset myCookie = cookie_("remember_token", "f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6")>
```

When constructing the `Cookie` object, only the `name` attribute is required. All other attributes are optional. If the optional attributes are not modified, their values will be filled up by the default values saved in the `Cookie` class.

!!! note "Difference from CodeIgniter"
    The global function is `cookie_()`, with a trailing underscore, because `cookie` is the name of a scope in CFML. The constructor and the static methods are called with `new_()` and `static_()`.

!!! warning "Not ported yet"
    An expiry time given as a date object (`DateTimeInterface`) or as a date text is not ported. `expires` and `withExpires()` take a Unix timestamp. Use `max-age` to set the lifetime in seconds from now.

### Overriding Defaults

To override the defaults currently stored in the class, you can pass a `Config\Cookie` instance or a struct of defaults to the static `setDefaults()` method.

```cfm
<cfset cookieClass = static_("ColdIgniter\Cookie\Cookie")>

<!--- pass in a Config\Cookie instance before constructing a Cookie class --->
<cfset cookieClass.setDefaults(config("Config\Cookie"))>
<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "login_token")>

<!--- pass in a struct of defaults --->
<cfset myDefaults = StructNew()>
<cfset myDefaults["expires"] = 0>
<cfset myDefaults["samesite"] = "Strict">
<cfset cookieClass.setDefaults(myDefaults)>
<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "login_token")>
```

Passing the `Config\Cookie` instance or a struct to `setDefaults()` will effectively overwrite your defaults and will persist until new defaults are passed.

#### Changing Defaults for a Limited Time

If you do not want this behavior but only want to change defaults for a limited time, you can take advantage of the return value of `setDefaults()`, which is the old defaults.

```cfm
<cfset cookieClass = static_("ColdIgniter\Cookie\Cookie")>

<cfset oldDefaults = cookieClass.setDefaults(config("Config\Cookie"))>
<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "my_token", "muffins")>

<!--- return the old defaults --->
<cfset cookieClass.setDefaults(oldDefaults)>
```

## Accessing Cookie's Attributes

Once instantiated, you can easily access a `Cookie`'s attribute by using one of its getter methods.

```cfm
<cfset options = StructNew()>
<cfset options["expires"] = 1739491200>
<cfset options["prefix"] = "__Secure-">
<cfset options["path"] = "/">
<cfset options["secure"] = true>
<cfset options["httponly"] = true>
<cfset options["raw"] = false>
<cfset options["samesite"] = "Lax">

<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "remember_token", "f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6", options)>

<cfset myCookie.getName()>             <!--- "remember_token" --->
<cfset myCookie.getPrefix()>           <!--- "__Secure-" --->
<cfset myCookie.getPrefixedName()>     <!--- "__Secure-remember_token" --->
<cfset myCookie.getExpiresTimestamp()> <!--- 1739491200 --->
<cfset myCookie.getExpiresString()>    <!--- "Fri, 14 Feb 2025 00:00:00 GMT" --->
<cfset myCookie.isExpired()>           <!--- true or false --->
<cfset myCookie.getMaxAge()>           <!--- the seconds from now to expires, 0 if past --->
<cfset myCookie.isRaw()>               <!--- false --->
<cfset myCookie.isSecure()>            <!--- true --->
<cfset myCookie.getPath()>             <!--- "/" --->
<cfset myCookie.getDomain()>           <!--- "" --->
<cfset myCookie.isHTTPOnly()>          <!--- true --->
<cfset myCookie.getSameSite()>         <!--- "Lax" --->

<!--- additional getter --->
<cfset myCookie.getId()>               <!--- "__Secure-remember_token;/;" --->

<!--- a struct of expires, path, domain, secure, httponly and samesite --->
<cfset myCookie.getOptions()>
```

## Immutable Cookies

A new `Cookie` instance is an immutable value object representation of an HTTP cookie. Being immutable, modifying any of the instance's attributes will not affect the original instance. The modification **always** returns a new instance. You need to retain this new instance in order to use it.

```cfm
<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "login_token", "admin")>
<cfset myCookie.getName()> <!--- "login_token" --->

<cfset myCookie.withName("remember_token")>
<cfset myCookie.getName()> <!--- "login_token" --->

<cfset new = myCookie.withName("remember_token")>
<cfset new.getName()> <!--- "remember_token" --->
```

## Validating a Cookie's Attributes

An HTTP cookie is regulated by several specifications that need to be followed in order to be accepted by browsers. Thus, when creating or modifying certain attributes of the `Cookie`, these are validated in order to check if these follow the specifications.

A `CookieException` is thrown if violations were reported. In CFML this is an error of the type `ColdIgniter.Cookie.Exceptions.CookieException`, see [Error Handling](../general/errors.md).

### Validating the Name Attribute

A cookie name can be any US-ASCII character, except for the following:

- control characters;
- spaces or tabs;
- separator characters, such as `( ) < > @ , ; : \ " / [ ] ? = { }`

If setting the `raw` option to `true` this validation will be strictly made. Without it, the characters are URL encoded when the cookie is sent. Cookie names cannot be an empty string.

### Validating the Value Attribute

If setting the `raw` option to `true`, the cookie value will also be validated. It must not contain control characters, spaces, tabs, or separator characters (`, ;`).

### Validating the Path Attribute

The cookie path must not contain control characters, spaces, tabs, or separator characters (`, ;`).

### Validating the Domain Attribute

The cookie domain must not contain control characters, spaces, tabs, or separator characters (`, ;`).

### Validating the Prefix Attribute

The cookie prefix must not contain control characters, spaces, tabs, or separator characters (`= , ; \t \r \n \v \f \0`).

When using the `__Secure-` prefix, cookies must be set with the `secure` flag set to `true`. If using the `__Host-` prefix, cookies must exhibit the following:

- `secure` flag set to `true`
- `domain` is empty
- `path` must be `/`

### Validating the SameSite Attribute

The SameSite attribute accepts three values:

- **Lax**: Cookies are not sent on normal cross-site subrequests (for example to load images or frames into a third party site), but are sent when a user is navigating to the origin site (*i.e.* when following a link).
- **Strict**: Cookies will only be sent in a first-party context and not be sent along with requests initiated by third party websites.
- **None**: Cookies will be sent in all contexts, *i.e.* in responses to both first-party and cross-origin requests.

ColdIgniter, however, allows you to set the SameSite attribute to an empty string. When an empty string is provided, the default SameSite setting saved in the `Cookie` class is used. You can change the default SameSite by using `setDefaults()` as discussed above.

Recent cookie specifications have changed such that modern browsers are being required to give a default SameSite if nothing was provided. This default is `Lax`. If you have set the SameSite to be an empty string and your default SameSite is also an empty string, your cookie will be given the `Lax` value.

If the SameSite is set to `None` you need to make sure that `secure` is also set to `true`.

When writing the SameSite attribute, the `Cookie` class accepts any of the values case-insensitively. You can also take advantage of the class's constants to make it not a hassle.

```cfm
<cfset cookieClass = static_("ColdIgniter\Cookie\Cookie")>

<cfset cookieClass.SAMESITE_LAX>    <!--- "Lax" --->
<cfset cookieClass.SAMESITE_STRICT> <!--- "Strict" --->
<cfset cookieClass.SAMESITE_NONE>   <!--- "None" --->
```

## Sending Cookies

Set the `Cookie` objects in the `CookieStore` of the Response object, and the framework will automatically send the cookies.

Use `setCookie()` of the [Response](../outgoing/response.md) to set:

```cfm
<cfset response = service("response")>

<cfset options = StructNew()>
<cfset options["max-age"] = 7200> <!--- Expires in 2 hours --->

<cfset myCookie = new_("ColdIgniter\Cookie\Cookie", "remember_token", "f699c7fd18a8e082d0228932f3acd40e1ef5ef92efcedda32842a211d62f0aa6", options)>

<cfset response.setCookie(myCookie)>
```

You can also use the [`set_cookie()`](../helpers/cookie_helper.md#set_cookie) helper function:

```cfm
<cfset helper("cookie")>

<cfset set_cookie(myCookie)>
```

## Using the Cookie Store

!!! note
    Normally, there is no need to use CookieStore directly.

The `CookieStore` class represents an immutable collection of `Cookie` objects.

### Getting the Store from Response

The `CookieStore` instance can be accessed from the current `Response` object.

```cfm
<cfset cookieStore = service("response").getCookieStore()>
```

### Creating CookieStore

ColdIgniter provides three other ways to create a new instance of the `CookieStore`.

```cfm
<!--- Passing an array of `Cookie` objects in the constructor --->
<cfset list = ArrayNew(1)>
<cfset ArrayAppend(list, cookie_("login_token"))>
<cfset ArrayAppend(list, cookie_("remember_token"))>
<cfset store = new_("ColdIgniter\Cookie\CookieStore", list)>

<!--- Passing an array of `Set-Cookie` header strings --->
<cfset headers = ArrayNew(1)>
<cfset ArrayAppend(headers, "remember_token=me; Path=/; SameSite=Lax")>
<cfset ArrayAppend(headers, "login_token=admin; Path=/; SameSite=Lax")>
<cfset store = static_("ColdIgniter\Cookie\CookieStore").fromCookieHeaders(headers)>

<!--- using the global `cookies` function --->
<cfset store = cookies(list, false)>

<!--- retrieving the `CookieStore` instance saved in our current `Response` object --->
<cfset store = cookies()>
```

!!! note
    When using the global `cookies()` function, the passed `Cookie` array will only be considered if the second argument, `getGlobal`, is set to `false`.

### Checking Cookies in Store

To check whether a `Cookie` object exists in the `CookieStore` instance, you can use several ways:

```cfm
<!--- check if cookie is in the current cookie collection --->
<cfset list = ArrayNew(1)>
<cfset ArrayAppend(list, cookie_("login_token"))>
<cfset ArrayAppend(list, cookie_("remember_token"))>
<cfset store = new_("ColdIgniter\Cookie\CookieStore", list)>
<cfset store.has("login_token")>

<!--- check if cookie is in the current Response's cookie collection --->
<cfset cookies().has("login_token")>
<cfset service("response").hasCookie("remember_token")>

<!--- using the cookie helper to check the current Response --->
<cfset helper("cookie")>
<cfset has_cookie("login_token")>
```

### Getting Cookies in Store

Retrieving a `Cookie` instance in a cookie collection is very easy:

```cfm
<!--- getting cookie in the current cookie collection --->
<cfset store.get("login_token")>

<!--- getting cookie in the current Response's cookie collection --->
<cfset cookies().get("login_token")>
<cfset service("response").getCookie("remember_token")>

<!--- using the cookie helper to get cookie from the request --->
<cfset helper("cookie")>
<cfset get_cookie("remember_token")>
```

When getting a `Cookie` instance directly from a `CookieStore`, an invalid name will throw a `CookieException`.

```cfm
<!--- throws CookieException --->
<cfset store.get("unknown_cookie")>
```

When getting a `Cookie` instance from the current `Response`'s cookie collection with `getCookie()`, an invalid name will just return an empty string, and the error is written to the log.

```cfm
<cfset service("response").getCookie("unknown_cookie")> <!--- "" --->
```

!!! note "Difference from CodeIgniter"
    CodeIgniter returns `null` here. CFML has no null, so ColdIgniter returns an empty string. Note that `cookies().get()` is the `CookieStore` method, which throws, and `service("response").getCookie()` is the one that returns an empty string.

If no arguments are supplied when getting cookies from the `Response`, all `Cookie` objects in store will be returned, as a struct keyed by the id of each cookie.

```cfm
<cfset cookies().display()> <!--- all Cookie objects --->

<!--- or even from the Response --->
<cfset service("response").getCookies()>
```

!!! note
    The helper function [`get_cookie()`](../helpers/cookie_helper.md#get_cookie) gets the cookie from the current `Request` object, not from `Response`. This function checks the cookies that the browser sent and fetches it right away.

### Adding/Removing Cookies in Store

As previously mentioned, `CookieStore` objects are immutable. You need to save the modified instance in order to work on it. The original instance is left unchanged.

```cfm
<!--- adding a new Cookie instance --->
<cfset new = store.put(cookie_("admin_token", "yes"))>

<!--- removing a Cookie instance --->
<cfset new = store.remove("login_token")>
```

!!! note
    Removing a cookie from the store **DOES NOT** delete it from the browser. If you intend to delete a cookie *from the browser*, you must put an empty value cookie with the same name to the store.

When interacting with the cookies in store in the current `Response` object, you can safely add or delete cookies without worrying the immutable nature of the cookie collection. The `Response` object will replace the instance with the modified instance.

```cfm
<cfset service("response").setCookie("admin_token", "yes")>
<cfset service("response").deleteCookie("login_token")>

<!--- using the cookie helper --->
<cfset helper("cookie")>
<cfset set_cookie("admin_token", "yes")>
<cfset delete_cookie("login_token")>
```

### Dispatching Cookies in Store

CodeIgniter removed the `dispatch()` method of the `CookieStore` in version 4.6.0, and it is not in ColdIgniter. The framework sends the cookies of the response for you.

## Cookie Personalization

Sane defaults are already in place inside the `Cookie` class to ensure the smooth creation of cookie objects. However, you may wish to define your own settings by changing the following settings in the `Config\Cookie` class in the **app/Config/Cookie.cfc** file.

| Setting | Options/Types | Default | Description |
|---|---|---|---|
| `prefix` | string | `""` | Prefix to prepend to the cookie name. |
| `expires` | number | `0` | The expires timestamp. |
| `path` | string | `/` | The path property of the cookie. |
| `domain` | string | `""` | The domain property of the cookie. |
| `secure` | `true`/`false` | `false` | If to be sent over secure HTTPS. |
| `httponly` | `true`/`false` | `true` | If not accessible to JavaScript. |
| `samesite` | `Lax`/`None`/`Strict` | `Lax` | The SameSite attribute. |
| `raw` | `true`/`false` | `false` | If the name and value are sent as they are, without URL encoding. |

!!! note "Difference from CodeIgniter"
    `expires` is a number only, a Unix timestamp. CodeIgniter also takes a `DateTimeInterface` or a string, which are not ported yet. `0` means no expiry, so the cookie lasts as long as the browser is open.

At runtime, you can manually supply a new default using the `setDefaults()` method.

## Class Reference

### Cookie

The class is `ColdIgniter\Cookie\Cookie`. Static members are used on `static_("ColdIgniter\Cookie\Cookie")`.

#### setDefaults()

`setDefaults(config)`

- `config` (`Config\Cookie` or struct, optional): the configuration object or struct.
- Returns the old defaults, as a struct.

Set the default attributes to a Cookie instance by injecting the values from the `Config\Cookie` config or a struct.

#### fromHeaderString()

`fromHeaderString(cookie, raw)`

- `cookie` (string): the `Set-Cookie` header string.
- `raw` (boolean, optional, default `false`): whether this cookie is not to be URL encoded.
- Returns a `Cookie` instance. May throw a `CookieException`.

Create a new Cookie instance from a `Set-Cookie` header.

#### Constructor

`new_("ColdIgniter\Cookie\Cookie", name, value, options)`

- `name` (string): the cookie name.
- `value` (string, optional, default `""`): the cookie value.
- `options` (struct, optional): the cookie options.
- Returns a `Cookie` instance. May throw a `CookieException`.

Construct a new Cookie instance. When both `expires` and `max-age` are given, `max-age` has precedence.

#### Getters

`getId()`, `getPrefix()`, `getName()`, `getPrefixedName()`, `getValue()`, `getExpiresTimestamp()`, `getExpiresString()`, `isExpired()`, `getMaxAge()`, `getDomain()`, `getPath()`, `isSecure()`, `isHTTPOnly()`, `getSameSite()`, `isRaw()` and `getOptions()` return the attribute named by the method. `getId()` is the ID used in indexing in the cookie collection: the prefixed name, the path and the domain, joined with `;`.

#### Withers

Each of these creates a new `Cookie` instance, and leaves the original unchanged:

- `withRaw(raw)`: new URL encoding option (default `true`).
- `withPrefix(prefix)`: new prefix (default `""`).
- `withName(name)`: new name.
- `withValue(value)`: new value.
- `withExpires(expires)`: new expiry, as a Unix timestamp.
- `withExpired()`: a cookie that will expire from the browser.
- `withDomain(domain)`: new domain.
- `withPath(path)`: new path.
- `withSecure(secure)`: new "Secure" attribute (default `true`).
- `withHTTPOnly(httponly)`: new "HttpOnly" attribute (default `true`).
- `withSameSite(samesite)`: new "SameSite" attribute.

#### toHeaderString()

Returns the string representation that can be passed as a header string.

#### toArray()

Returns the struct representation of the Cookie instance.

!!! note "Difference from CodeIgniter"
    A `Cookie` can be read like an array in CodeIgniter. Here the methods `offsetExists()`, `offsetGet()`, `offsetSet()` and `offsetUnset()` are called by name. The last two always throw, as a cookie is immutable. `withNeverExpiring()` was removed in CodeIgniter 4.6.0 and is not in ColdIgniter.

### CookieStore

The class is `ColdIgniter\Cookie\CookieStore`.

- `fromCookieHeaders(headers, raw)`: creates a CookieStore from an array of `Set-Cookie` headers. Called on `static_("ColdIgniter\Cookie\CookieStore")`.
- Constructor: `new_("ColdIgniter\Cookie\CookieStore", cookies)`, where `cookies` is an array of `Cookie` objects.
- `has(name, prefix, value)`: checks if a `Cookie` identified by name and prefix is present. If `value` is given, it must match. An empty `value` means any value.
- `get(name, prefix)`: retrieves a `Cookie` identified by a name and prefix. Throws a `CookieException` if it is not found.
- `put(cookie)`: stores a new cookie and returns a new `CookieStore`. The original is left unchanged.
- `remove(name, prefix)`: removes a cookie and returns a new `CookieStore`. The original is left unchanged.
- `display()`: returns all cookie instances in store, as a struct keyed by id.
- `clear()`: clears the cookie collection.
- `count()`: returns the number of cookies in the store.

!!! note "Difference from CodeIgniter"
    There is no `Countable` and no iterator. `count()` is a method, and `getIterator()` returns an array of the cookies that a `cfloop` can walk.
