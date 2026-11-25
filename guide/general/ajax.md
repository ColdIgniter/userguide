# AJAX Requests

The `isAJAX()` function of the incoming request uses the `X-Requested-With` header to define whether the request is XHR or normal. However, modern JavaScript APIs such as `fetch` no longer send this header by default, so `isAJAX()` becomes less reliable without additional configuration.

```cfm
<cfif request_().isAJAX()>
    <!--- answer with a fragment or with JSON --->
</cfif>
```

The function is true only when the header is present and its value is `XMLHttpRequest` (in any letter case).

To work around this problem, manually define the request header so the server can identify the request as XHR.

Here are common ways to send the `X-Requested-With` header in the Fetch API and other JavaScript libraries.

## Fetch API

```javascript
fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Requested-With": "XMLHttpRequest"
    }
});
```

## Axios

Axios does not include the `X-Requested-With` header by default. You can add it globally as follows:

```javascript
axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
```

If you prefer to avoid global defaults, create an Axios instance instead:

```javascript
const api = axios.create({
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    }
});
```

### Vue.js

Vue does not require a specific HTTP client. If your Vue app uses Axios, configure Axios once during application bootstrap or in a shared API module, and reuse that configuration throughout the app.

### React

React also does not provide a built-in HTTP client. If your React app uses Axios, reuse the shared Axios configuration above, or set the header for an individual request when needed:

```javascript
axios.get('your url', {
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    }
})
```

## jQuery

For libraries like jQuery, it is not necessary to send this header explicitly, because according to the [official documentation](https://api.jquery.com/jquery.ajax/) it is a standard header for all `$.ajax()` requests. But if you still want to force it, do it as follows:

```javascript
$.ajax({
    url: "your url",
    headers: {'X-Requested-With': 'XMLHttpRequest'}
});
```

## htmx

You can use the [ajax-header](https://github.com/bigskysoftware/htmx-extensions/blob/main/src/ajax-header/README.md) extension:

```html
<body hx-ext="ajax-header">
...
</body>
```

Or you can set the header manually with `hx-headers`:

```html
<button
    hx-post="/your-url"
    hx-headers='{"X-Requested-With": "XMLHttpRequest"}'>
    Send Request
</button>
```
