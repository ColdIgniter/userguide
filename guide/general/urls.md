# ColdIgniter URLs

By default, URLs in ColdIgniter are designed to be search-engine and human-friendly. Rather than using the standard "query-string" approach to URLs that is synonymous with dynamic systems, ColdIgniter uses a **segment-based** approach:

```
https://example.com/news/article/my_article
```

Your URLs can be defined using the [URI Routing](../incoming/routing.md) feature with flexibility.

The [URI Library](../libraries/uri.md) and the [URL Helper](../helpers/url_helper.md) contain functions that make it easy to work with your URI data.

## URL Structure

### Base URL contains only the Hostname

When you have the Base URL **https://www.example.com/** and imagine the following URL:

```
https://www.example.com/blog/news/2022/10?page=2
```

We use the following terms:

| Term       | Example                      | Description                                                              |
| ---------- | ---------------------------- | ------------------------------------------------------------------------ |
| Base URL   | **https://www.example.com/** | Base URL is often denoted as **baseURL**.                                |
| URI path   | /blog/news/2022/10           |                                                                          |
| Route path | /blog/news/2022/10           | The URI path relative to the Base URL. It is also called **URI string**. |
| Query      | page=2                       |                                                                          |

### Base URL contains Sub folders

When you have the Base URL **https://www.example.com/ci-blog/** and imagine the following URL:

```
https://www.example.com/ci-blog/blog/news/2022/10?page=2
```

We use the following terms:

| Term       | Example                              | Description                                                              |
| ---------- | ------------------------------------ | ------------------------------------------------------------------------ |
| Base URL   | **https://www.example.com/ci-blog/** | Base URL is often denoted as **baseURL**.                                |
| URI path   | /ci-blog/blog/news/2022/10           |                                                                          |
| Route path | /blog/news/2022/10                   | The URI path relative to the Base URL. It is also called **URI string**. |
| Query      | page=2                               |                                                                          |

## URI Security

ColdIgniter is fairly restrictive regarding which characters it allows in your URI strings (Route path) in order to help minimize the possibility that malicious data can be passed to your application. URIs may only contain the following:

- Alpha-numeric text (latin characters only)
- Tilde: `~`
- Percent sign: `%`
- Period: `.`
- Colon: `:`
- Underscore: `_`
- Dash: `-`
- Space character

!!! note
    This check is performed by the `Router`. It tests each segment of the route path against the setting `permittedURIChars` of **app/Config/App.cfc**.

### Adding Permitted Characters

The permitted characters can be changed by the property `permittedURIChars` of `Config\App`. Its default is:

```cfm
<cfset this.permittedURIChars = "a-z 0-9~%.:_\-">
```

The value is the inside of a character class of a Java regular expression, tested without regard to case. If you want to allow other characters in URI paths, add them to the value in **app/Config/App.cfc**.

## Removing the index.cfm file

By default, the **index.cfm** file will be needed in your URLs:

```
example.com/index.cfm/news/article/my_article
```

The name of the file is the setting `indexPage` of **app/Config/App.cfc**. If your server supports rewriting URLs you can remove this file with URL rewriting, and set `indexPage` to an empty string.

!!! note "Difference from CodeIgniter"
    ColdIgniter runs under a web server that works with ColdFusion MX 6.1, so there are no Apache and nginx instructions here. The framework ships the file **public/.htaccess** with rewrite rules for Apache and for IIS with the URL Magic filter. How to set up the filter is described in [Running Your App](../installation/running.md).

The central rule of the file is the "negative" method, in which everything is sent to the front controller except real files and folders:

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([\s\S]*)$ index.cfm/$1 [L,NC,QSA]
```

In this example, any HTTP request other than those for existing directories and existing files is treated as a request for your **index.cfm** file.

!!! note
    These specific rules might not work for all server configurations. Make sure to also exclude from the rules any assets that you might need to be accessible from the outside world.

### Apache Web Server

The file **public/.htaccess** works with Apache when the *mod_rewrite* extension is enabled. See [Running Your App](../installation/running.md) for the rest.

### nginx

!!! warning "Not ported yet"
    ColdIgniter has no instructions for nginx. The framework ships rewrite rules only for Apache and for IIS with URL Magic.
