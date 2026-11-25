# Content Negotiation

## What is Content Negotiation?

Content negotiation is a way to decide what type of content to return to the client, based on what the client can handle and what the server can handle. It can decide whether the client wants HTML or JSON, whether an image should be returned as JPEG or PNG, which compression is supported, and more. It works by looking at four headers. Each of them can hold several values, each with its own priority. Matching these by hand is hard. ColdIgniter has the `Negotiate` class to do it for you.

At its heart, content negotiation is a part of the HTTP specification that lets a single resource serve more than one type of content, so that clients can ask for the type that works best for them.

A classic example is a browser that cannot show PNG files and asks only for GIF or JPEG images. When the server gets the request, it looks at the file types the client asks for and picks the best match from the formats it supports. Here it would most likely return a JPEG image.

The same negotiation can happen with four types of data:

- **Media/Document Type**: this could be the image format, or HTML versus XML or JSON.
- **Character Set**: the character set the returned document should use. Usually UTF-8.
- **Document Encoding**: usually the type of compression used on the result.
- **Document Language**: for sites that support several languages, this helps decide which one to return.

## Loading the Class

You can load an instance of the class through the `negotiator` service:

```cfm
<cfset negotiate = service("negotiator")>
```

This gets the current request and passes it to the Negotiate class.

You do not need to load the class on its own. You can reach it through the `IncomingRequest` of the current request. You cannot reach the class itself this way, but you can use all its methods through `negotiate()`:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "foo")>
<cfset ArrayAppend(supported, "bar")>
<cfset type = this.request_.negotiate("media", supported)>
```

When you use it this way, the first argument is the type of content you want to match: `media`, `charset`, `encoding` or `language`. The second argument is an array of the values you support.

## Negotiating

This section covers the four types of content that can be negotiated. Each example shows both ways to reach the negotiator.

### Media

The first aspect is the negotiation of media. It uses the `Accept` header, one of the most complex headers. A common example is a client that tells the server which format it wants the data in. This is common in APIs. A client might ask an API endpoint for JSON:

```
GET /foo HTTP/1.1
Accept: application/json
```

The server now needs a list of the content types it can provide. In this example the API might return raw HTML, JSON or XML. Give the list in order of preference:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "application/json")>
<cfset ArrayAppend(supported, "text/html")>
<cfset ArrayAppend(supported, "application/xml")>

<cfset format = this.request_.negotiate("media", supported)>
<!--- or --->
<cfset format = negotiate.media(supported)>
```

Here the client and the server agree on JSON, so `application/json` is returned. By default, if no match is found, the first element of the supported array is returned.

Sometimes you need a strict match. If you pass `true` as the last argument, an empty string is returned when there is no match:

```cfm
<cfset format = this.request_.negotiate("media", supported, true)>
<!--- or --->
<cfset format = negotiate.media(supported, true)>
```

### Language

Another common use is to decide which language to serve the content in. On a site with only one language it makes no difference, but a site that offers several translations will find it useful. The browser usually sends its preferred languages in the `Accept-Language` header:

```
GET /foo HTTP/1.1
Accept-Language: fr; q=1.0, en; q=0.5
```

Here the browser prefers French, with English as the second choice. If your site supports English and German, you would write:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "en")>
<cfset ArrayAppend(supported, "de")>

<cfset lang = this.request_.negotiate("language", supported)>
<!--- or --->
<cfset lang = negotiate.language(supported)>
```

In this example `en` is returned. If no match is found, the first element of the supported array is returned, so it should always be your preferred language.

#### Strict Locale Negotiation

By default the locale is compared in a lossy way: only the first part of the locale text, the language, is used. This is usually enough. Sometimes you want to tell regional versions such as `en-US` and `en-GB` apart, to serve different content.

For this, set `strictLocaleNegotiation` to true in `app/Config/Feature.cfc`. The default is false. Then the strict comparison is made first.

Now look at this example. The browser sends:

```
GET /foo HTTP/1.1
Accept-Language: fr; q=1.0, en-GB; q=0.5
```

The browser prefers French, with British English as the second choice. Your site supports German and American English:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "de")>
<cfset ArrayAppend(supported, "en-US")>

<cfset lang = this.request_.negotiate("language", supported)>
<!--- or --->
<cfset lang = negotiate.language(supported)>
```

Here `en-US` is returned. If no match is found, the first element of the supported array is returned. This is how the selection works. The browser prefers `fr`, but it is not in the supported array. `en-GB` is not in it either, so the next step is to look for variants. First it falls back to the most general locale, here `en`, which is not in the array. Then it looks for a regional locale `en-`. That matches `en-US` from the array, so `en-US` is returned.

The selection goes like this:

1. strict match (`en-GB`): the language and the country code
2. general locale match (`en`): the language only
3. regional locale match (`en-`): the language and any country code

### Encoding

The `Accept-Encoding` header holds the encodings the client prefers to receive. It is used to say which types of compression the client supports:

```
GET /foo HTTP/1.1
Accept-Encoding: compress, gzip
```

Your web server decides which types of compression you can use. Some servers support only **gzip**:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "gzip")>

<cfset type = this.request_.negotiate("encoding", supported)>
<!--- or --->
<cfset type = negotiate.encoding(supported)>
```

The value `identity` (no compression) is always added to your list.

### Character Set

The wanted character set is sent in the `Accept-Charset` header:

```
GET /foo HTTP/1.1
Accept-Charset: utf-16, utf-8
```

By default, if no match is found, **utf-8** is returned:

```cfm
<cfset supported = ArrayNew(1)>
<cfset ArrayAppend(supported, "utf-8")>

<cfset charset = this.request_.negotiate("charset", supported)>
<!--- or --->
<cfset charset = negotiate.charset(supported)>
```
