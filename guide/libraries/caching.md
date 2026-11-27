# Caching Driver

ColdIgniter features wrappers around some of the most popular forms of fast and dynamic caching. In ColdIgniter, two drivers are ported: the file-based driver and a dummy driver that stores nothing.

!!! warning "Not ported yet"
    The APCu, Memcached, Redis, Predis and WinCache drivers are not ported. See [Drivers](#drivers).

## Example Usage

The following example shows a common usage pattern within your controllers.

```cfm
<cfset foo = cache("foo")>

<cfif Len(foo) EQ 0>
	<cfoutput>Saving to the cache!<br></cfoutput>
	<cfset foo = "foobarbaz!">

	<!--- Save into the cache for 5 minutes --->
	<cfset cache().save("foo", foo, 300)>
</cfif>

<cfoutput>#foo#</cfoutput>
```

!!! note "Difference from CodeIgniter"
    CFML has no null. When an item is not found, `get()` returns an empty string, not `null`. This also means that an item that holds an empty string reads as not found.

You can grab an instance of the cache engine directly through the global function `service()`:

```cfm
<cfset cacheService = service("cache")>

<cfset foo = cacheService.get("foo")>
```

The global function `cache()` with no argument returns the same instance. With a key, `cache("foo")` returns the item, as in the example above.

## Configuring the Cache

All configuration for the cache engine is done in **app/Config/Cache.cfc**. In that file, the following items are available.

### handler

This is the name of the handler that should be used as the primary handler when starting up the engine. The names you can use are `file` and `dummy`. The default is `file`.

### backupHandler

In the case that the first choice `handler` is not available, this is the next cache handler to load. This is commonly the `file` handler since the file system is always available, but may not fit more complex, multi-server setups. The default is `dummy`. If the backup handler is not available either, the `dummy` handler is used.

### prefix

If you have more than one application using the same cache storage, you can add a custom prefix string here that is prepended to all key names. The default is an empty string.

### ttl

The default number of seconds to save items when none is specified. The default is `60`.

!!! warning
    This is not used by framework handlers, where 60 seconds is hard-coded, but may be useful to projects and modules. This will replace the hard-coded value in a future release.

### reservedCharacters

A string of characters that are not allowed in keys. A key that holds one of them makes the handler throw. The default is `{}()/\@:`.

### file_

This is a struct of settings specific to the `file` handler to determine how it should save the cache files. The name ends in an underscore, because `file` is a scope in CFML. It has these keys:

- `storePath`: the folder for the cache files. The default is `writable/cache/`. The folder is created when it does not exist.
- `memoryEntries`: how many cached items the handler keeps in the memory of the server, so that a hit does not read the file. The default is `100`. Set `0` to switch it off.

!!! note "Difference from CodeIgniter"
    `memoryEntries` is not in CodeIgniter. The files stay the durable copy: an item in memory is used only while the modified time of its file has not changed.

!!! warning "Not ported yet"
    The `mode` key of `file_`, which sets the permissions of the cache files, is not ported, because there is no `chmod`. The `memcached` and `redis` settings are not ported either.

### validHandlers

A struct of the handler names and the class that each one loads. Only the handlers listed here can be used. The ported ones are `dummy` and `file`.

### frontPages, cacheQueryString and cacheStatusCodes

These settings belong to [Web Page Caching](../general/caching.md).

## Command-Line Tools

!!! warning "Not ported yet"
    The `cache:clear` and `cache:info` commands are not ported, because ColdIgniter has no command line. To clear the cache from code, call `clean()` as shown below.

## Class Reference

### CacheInterface

CFML on ColdFusion MX 6.1 has no interfaces. `CacheInterface` is a plain component whose methods all throw, and `BaseHandler` extends it. The handlers extend `BaseHandler`.

!!! note "Difference from CodeIgniter"
    Two method names end in an underscore, because the plain names are built-in functions: `initialize_()` and `getMetaData_()`.

#### isSupported()

- Returns `true` if the handler is supported, `false` if not. For the `file` handler this is whether the cache folder is writable.

#### get()

`get(key)`

- `key` (string): the cache item name.
- Returns the item value, or an empty string if it is not found.

This method will attempt to fetch an item from the cache store. If the item does not exist, the method will return an empty string.

```cfm
<cfset foo = cacheService.get("my_item")>
```

#### remember()

`remember(key, ttl, callback)`

- `key` (string): the cache item name.
- `ttl` (numeric): the time to live in seconds.
- `callback` (array or string): the callback to invoke when the cache item is not found.
- Returns the value of the cache item.

Gets an item from the cache. If it is not found, this will invoke the callback and save the result. Either way, this will return the value.

CFML has no closures, so the callback is an array made of an object and the name of one of its methods, or a string `"Class::method"`. The method is called with no arguments:

```cfm
<cfset callback = ArrayNew(1)>
<cfset ArrayAppend(callback, model)>
<cfset ArrayAppend(callback, "getLatest")>

<cfset latest = cacheService.remember("latest", 300, callback)>
```

The method name must be a plain name of letters, digits and underscores, or an exception is thrown. As an empty string stands for a missing item, a cached empty string makes the callback run again.

#### save()

`save(key, value, ttl)`

- `key` (string): the cache item name.
- `value` (any): the data to save.
- `ttl` (numeric, optional, default `60`): the time to live, in seconds.
- Returns `true` on success, `false` on failure.

This method will save an item to the cache store. If saving fails, the method will return `false`.

```cfm
<cfset cacheService.save("cache_item_id", "data_to_cache")>
```

!!! note
    The `file` handler can store a string, a number, a struct, an array and a query. A component cannot be cached, also not inside a struct or an array: `save()` returns `false`.

!!! note "Difference from CodeIgniter"
    CodeIgniter writes `serialize()` text to the cache files. ColdIgniter writes a Java object stream, so the folder must be writable by the application only. A file written by an earlier version of the framework reads as not found.

#### delete()

`delete(key)`

- `key` (string): the name of the cached item.
- Returns `true` on success, `false` on failure.

This method will delete a specific item from the cache store. If item deletion fails, the method will return `false`.

```cfm
<cfset cacheService.delete("cache_item_id")>
```

#### deleteMatching()

`deleteMatching(pattern)`

- `pattern` (string): a glob-style pattern to match the keys of the cached items.
- Returns the number of deleted items.

This method will delete multiple items from the cache store at once by matching their keys against a glob-style pattern. It will return the total number of deleted items.

```cfm
<cfset cacheService.deleteMatching("prefix_*")> <!--- deletes all items of which keys start with "prefix_" --->
<cfset cacheService.deleteMatching("*_suffix")> <!--- deletes all items of which keys end with "_suffix" --->
```

!!! note
    With the `file` handler, the pattern is matched against the cache file names only and cannot match files in subdirectories. A pattern that holds a forward slash (`/`) therefore never matches. A wildcard does not match a leading period of a name.

For more information on glob-style syntax, see [Glob (programming)](https://en.wikipedia.org/wiki/Glob_(programming)#Syntax).

#### increment()

`increment(key, offset)`

- `key` (string): the cache ID.
- `offset` (numeric, optional, default `1`): the step to add.
- Returns the new value on success, `false` on failure.

Performs the incrementation of a raw stored value. If the item does not exist, it starts at `0`. If the stored value is not an integer, the method returns `false`.

```cfm
<!--- "iterator" has a value of 2 --->
<cfset cacheService.increment("iterator")> <!--- "iterator" is now 3 --->
<cfset cacheService.increment("iterator", 3)> <!--- "iterator" is now 6 --->
```

!!! note "Difference from CodeIgniter"
    With the `file` handler the increment is a read and a write of the file. It is not atomic.

#### decrement()

`decrement(key, offset)`

- `key` (string): the cache ID.
- `offset` (numeric, optional, default `1`): the step to reduce by.
- Returns the new value on success, `false` on failure.

Performs the decrementation of a raw stored value, in the same way as `increment()`.

```cfm
<!--- "iterator" has a value of 6 --->
<cfset cacheService.decrement("iterator")> <!--- "iterator" is now 5 --->
<cfset cacheService.decrement("iterator", 2)> <!--- "iterator" is now 3 --->
```

#### clean()

- Returns `true` on success, `false` on failure.

This method will "clean" the entire cache. If the deletion of the cache files fails, the method will return `false`. The files `index.html`, `index.htm`, `.htaccess` and `web.config` in the cache folder are kept.

```cfm
<cfset cacheService.clean()>
```

#### getCacheInfo()

- Returns information on the entire cache.

This method will return information on the entire cache. For the `file` handler, it is a struct with one entry for each cache file, holding the keys `name`, `server_path`, `size`, `date` and `relative_path`. The `dummy` handler returns an empty string.

```cfm
<cfset info = cacheService.getCacheInfo()>
```

!!! note
    The information returned and the structure of the data depend on which handler is being used.

#### getMetaData_()

`getMetaData_(key)`

- `key` (string): the cache item name.
- Returns the metadata for the cached item, or an empty string if it is not found.

This method will return detailed information on a specific item in the cache. For the `file` handler, it is a struct with the keys `expire` (the time the item expires, as a number of seconds since 1 January 1970, or an empty string if it never expires), `mtime` (the time the file was changed, in the same unit) and `data` (the value).

```cfm
<cfset meta = cacheService.getMetaData_("my_cached_item")>
```

!!! note
    The information returned and the structure of the data depend on which handler is being used.

#### validateKey()

`validateKey(key, prefix)`

- `key` (string): the potential cache key.
- `prefix` (string, optional, default `""`): a prefix.
- Returns the verified and prefixed key. If the key is longer than the maximum key length of the handler (255 characters for the `file` handler), it is hashed.

This method is used by handler methods to check that keys are valid. It throws an `InvalidArgumentException` for a value that is not a string, a key that holds a character of `reservedCharacters`, and an empty key.

!!! note "Difference from CodeIgniter"
    `validateKey()` is not a static method. A handler calls it on itself. Without a handler, call it on the base class:

```cfm
<cfset base = static_("ColdIgniter\Cache\Handlers\BaseHandler")>
<cfset prefixedKey = base.validateKey(key, prefix)>
```

## Drivers

### APCu Caching

!!! warning "Not ported yet"
    The APCu driver is not ported.

### File-based Caching

This requires a cache folder to be really writable by the application. It is `writable/cache/` unless you set `storePath` in `file_`.

Use this with care, and make sure to benchmark your application, as a point can come where disk I/O will negate positive gains by caching.

!!! note "Difference from CodeIgniter"
    The file cache belongs to each server, unless `writable/cache` is a folder that the servers share. Each file is checked for a change of its modified time, so a shared folder stays correct. Without a shared folder, a cached item or a delete only affects that one server.

    A key that the file system could not take as a file name (it holds one of `* ? " < > | / \ :`, or is `.` or `..`, or ends with a period or a space, or is a device name such as `CON`) is stored under the `md5` of the prefixed key.

### Memcached Caching

!!! warning "Not ported yet"
    The Memcached driver is not ported.

### WinCache Caching

!!! warning "Not ported yet"
    The WinCache driver is not ported.

### Redis Caching

!!! warning "Not ported yet"
    The Redis driver is not ported.

### Predis Caching

!!! warning "Not ported yet"
    The Predis driver is not ported.

### Dummy Cache

This is a caching backend that will always "miss". It stores no data, but lets you keep your caching code in place in environments that do not support your chosen cache.
