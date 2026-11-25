# Web Page Caching

ColdIgniter lets you cache your pages in order to achieve maximum performance.

Although ColdIgniter is quite fast, the amount of dynamic information you display in your pages will correlate directly to the server resources, memory, and processing cycles utilized, which affect your page load speeds. By caching your pages, since they are saved in their fully rendered state, you can achieve performance much closer to that of static web pages.

## How Does Caching Work?

Caching can be enabled on a per-page basis, and you can set the length of time that a page should remain cached before being refreshed.

!!! note
    Per-page means per URI. The HTTP method of the request is also taken into account. This means that the same URI will be cached separately if the HTTP method is different.

!!! warning
    Web Page Caching is shared between requests with the same HTTP method and URI. Its cache key does not vary by authenticated user, cookies, authorization headers, `Cache-Control`, or `Vary`. Do not use it for protected or personalized responses, as cached responses may be returned before authentication and authorization filters run.

When a page is loaded for the first time, the page will be cached using the currently configured cache engine. On subsequent page loads, the cache will be retrieved and sent to the requesting user's browser.

If it has expired, it will be deleted and refreshed before being sent to the browser.

!!! note "Difference from CodeIgniter"
    The setting `frontPages` of **app/Config/Cache.cfc** (default `true`) lets the framework keep a page that it served from the cache once in the memory of the application. Such a page is then sent before the framework starts, so no events, filters or controller run for it. The entry is dropped when the cached file changes, when the page expires, or when **app/Config/App.cfc**, **app/Config/Cache.cfc**, **app/Config/Filters.cfc** or the **.env** file changes. Set `frontPages` to `false` to turn this off.

## Configuring Caching

### Setting Cache Engine

Before using Web Page Caching, you must set the cache engine up by editing **app/Config/Cache.cfc**. See [Caching Driver](../libraries/caching.md) for details.

### Setting cacheQueryString

You can set whether or not to include the query string when generating the cache with `cacheQueryString` of **app/Config/Cache.cfc**.

Valid options are:

- `false`: (default) Disabled. The query string is not taken into account; the same cache is returned for requests with the same URI path but different query strings.
- `true`: Enabled, take all query parameters into account. Be aware that this may result in numerous cache generated for the same page over and over again.
- **array**: Enabled, but only take into account the specified list of query parameters. For example, an array made with `ListToArray("q,page")`.

### Setting cacheStatusCodes

You can control which HTTP response status codes are allowed to be cached with `cacheStatusCodes` of **app/Config/Cache.cfc**.

Valid options are:

- an empty array: (default) Cache all HTTP status codes.
- an array with `200`: (Recommended) Only cache successful responses. This prevents caching of error pages (404, 500, etc.) that should be temporary.
- an array of other status codes: Cache only specific status codes. For example, `ListToArray("200,404")` caches successful responses and not found pages.

```cfm
<cfset this.cacheStatusCodes = ListToArray("200")>
```

!!! warning
    Using an empty array may cache temporary error pages (404, 500, etc). For production applications, consider restricting this to `200` to avoid caching errors that should be temporary. For example, a cached 404 page would remain cached even after the resource is created, until the cache expires.

!!! note
    Regardless of this setting, redirect responses are never cached by the `PageCache` filter.

## Enabling Caching

To enable caching, put the following tag in any of your controller methods:

```cfm
<cfset cachePage(n)>
```

Where `n` is the number of **seconds** you wish the page to remain cached between refreshes.

The above tag can go anywhere within a method. It is not affected by the order that it appears, so place it wherever it seems most logical to you. Once the tag is in place, your pages will begin being cached.

!!! important
    If you change configuration options that might affect your output, you have to manually delete your cache.

## Deleting Caches

If you no longer wish to cache a page you can remove the caching tag and it will no longer be refreshed when it expires.

!!! note
    Removing the tag will not delete the cache immediately. It will have to expire normally.
