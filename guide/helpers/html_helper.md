# HTML Helper

The HTML Helper file contains functions that assist in working with HTML.

## Configuration

Void HTML elements (such as `<img>`) in the functions of this helper are HTML5-compatible by default. If you need to be compatible with XHTML, you must set the `html5` property in `app/Config/DocTypes.cfc` to `false`.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("html")>
```

## Available Functions

The following functions are available:

### img()

`img(src, indexPage, attributes)`

- `src` (string or struct, optional, default `""`): the image source URI, or a struct of attributes and values.
- `indexPage` (boolean, optional, default `false`): whether the `indexPage` of `Config\App` is added to the source path.
- `attributes` (string or struct, optional, default `""`): additional HTML attributes.
- Returns an HTML image element.

Lets you create HTML `<img>` elements. The first parameter contains the image source. Example:

```cfm
<cfoutput>#img("images/picture.jpg")#</cfoutput>
<!--- <img src="http://example.com/images/picture.jpg" alt=""> --->
```

There is an optional second parameter, a true/false value, that specifies if the *src* should have the `indexPage` of `Config\App` added to the address it creates. Presumably, this would be if you were using a media controller:

```cfm
<cfoutput>#img("images/picture.jpg", true)#</cfoutput>
<!--- <img src="http://example.com/index.cfm/images/picture.jpg" alt=""> --->
```

Additionally, a struct can be passed as the first parameter, for complete control over all attributes and values. If an *alt* attribute is not provided, ColdIgniter generates one with an empty string.

Example:

```cfm
<cfset imageProperties = StructNew()>
<cfset imageProperties["src"] = "images/picture.jpg">
<cfset imageProperties["alt"] = "Me, demonstrating how to eat 4 slices of pizza at one time">
<cfset imageProperties["class"] = "post_images">
<cfset imageProperties["width"] = "200">
<cfset imageProperties["height"] = "200">
<cfset imageProperties["title"] = "That was quite a night">
<cfset imageProperties["rel"] = "lightbox">

<cfoutput>#img(imageProperties)#</cfoutput>
<!--- <img src="http://example.com/images/picture.jpg" alt="Me, demonstrating how to eat 4 slices of pizza at one time" class="post_images" width="200" height="200" title="That was quite a night" rel="lightbox"> --->
```

!!! note "Difference from CodeIgniter"
    A struct has no order, so the attributes after `src` can come in another order than the one you set them in. The struct you pass is not changed. Write the keys in lower case and with brackets, as in the example.

### img_data()

`img_data(path, mime)`

- `path` (string): the path to the image file.
- `mime` (string, optional): the MIME type to use. Leave it out to guess.
- Returns a base64 encoded binary image string.

Generates a src-ready string from an image using the `data:` protocol. Example:

```cfm
<cfset src = img_data(ExpandPath("images/picture.gif"))> <!--- data:image/gif;base64,R0lGODl... --->

<cfoutput>#img(src)#</cfoutput>
```

There is an optional second parameter to specify the MIME type, otherwise the function uses your Mimes config to guess:

```cfm
<cfset src = img_data(ExpandPath("path/img_without_extension"), "image/png")> <!--- data:image/png;base64,HT5A822... --->
```

Note that `path` must exist and be a readable image format supported by the `data:` protocol. If the file does not exist or cannot be read, a `FileNotFoundException` is thrown. This function is not recommended for very large files, but it provides a convenient way of serving images from your app that are not web-accessible.

!!! note "Difference from CodeIgniter"
    CodeIgniter takes `null` for "guess the MIME type". Here the second parameter is left out. When the type cannot be guessed from the file extension, `image/jpg` is used.

### link_tag()

`link_tag(href, rel, type, title, media, indexPage, hreflang)`

- `href` (string or struct, optional, default `""`): the source of the link file.
- `rel` (string, optional, default `stylesheet`): the relation type.
- `type` (string, optional, default `text/css`): the type of the related document.
- `title` (string, optional, default `""`): the link title.
- `media` (string, optional, default `""`): the media type.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the link path.
- `hreflang` (string, optional, default `""`): the hreflang type.
- Returns an HTML link element.

Lets you create HTML `<link>` elements. This is useful for stylesheet links, as well as other links. The parameters are *href*, with optional *rel*, *type*, *title*, *media*, *indexPage* and *hreflang*.

*indexPage* is a boolean value that specifies if the *href* should have the page specified by the `indexPage` of `Config\App` added to the address it creates.

Example:

```cfm
<cfoutput>#link_tag("css/mystyles.css")#</cfoutput>
<!--- <link href="http://example.com/css/mystyles.css" rel="stylesheet" type="text/css"> --->
```

Further examples:

```cfm
<cfoutput>#link_tag("favicon.ico", "shortcut icon", "image/ico")#</cfoutput>
<!--- <link href="http://example.com/favicon.ico" rel="shortcut icon" type="image/ico"> --->

<cfoutput>#link_tag("feed", "alternate", "application/rss+xml", "My RSS Feed")#</cfoutput>
<!--- <link href="http://example.com/feed" rel="alternate" type="application/rss+xml" title="My RSS Feed"> --->
```

Alternately, a struct can be passed to the `link_tag()` function for complete control over all attributes and values:

```cfm
<cfset link = StructNew()>
<cfset link["href"] = "css/printer.css">
<cfset link["rel"] = "stylesheet">
<cfset link["type"] = "text/css">
<cfset link["media"] = "print">

<cfoutput>#link_tag(link)#</cfoutput>
<!--- <link href="http://example.com/css/printer.css" rel="stylesheet" type="text/css" media="print"> --->
```

The keys that are read from the struct are `href`, `rel`, `type`, `title`, `media`, `hreflang` and `indexPage`.

### script_tag()

`script_tag(src, indexPage)`

- `src` (string or struct, optional, default `""`): the source name or URL of a JavaScript file, or a struct specifying the attributes.
- `indexPage` (boolean, optional, default `false`): whether to treat `src` as a routed URI string.
- Returns an HTML script element.

Lets you create HTML `<script>` elements. The parameters are *src* and optional *indexPage*.

*indexPage* is a boolean value that specifies if the *src* should have the page specified by the `indexPage` of `Config\App` added to the address it creates.

Example:

```cfm
<cfoutput>#script_tag("js/mystyles.js")#</cfoutput>
<!--- <script src="http://example.com/js/mystyles.js"></script> --->
```

Alternately, a struct can be passed to the `script_tag()` function for complete control over all attributes and values:

```cfm
<cfset script = StructNew()>
<cfset script["src"] = "js/printer.js">
<cfset script["defer"] = "">

<cfoutput>#script_tag(script)#</cfoutput>
<!--- <script src="http://example.com/js/printer.js" defer></script> --->
```

!!! note "Difference from CodeIgniter"
    An attribute without a value, like `async` or `defer`, has an empty string for its value. CodeIgniter takes `null` there. A struct has no order, so the attributes can come in another order than the one you set them in. Write the key `src` in lower case and with brackets, as in the example.

### ul()

`ul(list, attributes)`

- `list` (array or struct): the list entries.
- `attributes` (string or struct, optional, default `""`): the HTML attributes.
- Returns an HTML unordered list element.

Permits you to generate an unordered HTML list from a simple or multi-dimensional array. Example:

```cfm
<cfset list = ListToArray("red,blue,green,yellow")>

<cfoutput>#ul(list, "class=""boldlist"" id=""mylist""")#</cfoutput>
```

The above code produces this:

```html
<ul class="boldlist" id="mylist">
  <li>red</li>
  <li>blue</li>
  <li>green</li>
  <li>yellow</li>
</ul>
```

Here is a more complex example, using a struct of arrays:

```cfm
<cfset list = StructNew()>
<cfset list["colors"] = ListToArray("red,blue,green")>
<cfset list["shapes"] = ListToArray("round,square")>

<cfoutput>#ul(list, "class=""boldlist"" id=""mylist""")#</cfoutput>
```

The above code produces this:

```html
<ul class="boldlist" id="mylist">
  <li>colors
    <ul>
      <li>red</li>
      <li>blue</li>
      <li>green</li>
    </ul>
  </li>
  <li>shapes
    <ul>
      <li>round</li>
      <li>square</li>
    </ul>
  </li>
</ul>
```

A list inside a list can go as deep as you need.

!!! note "Difference from CodeIgniter"
    A struct has no order, so the entries of a struct (`colors` and `shapes` above) can come in another order. An entry of a CFML array that is a list itself has no name: its position in the array, counted from 0, is printed in its place. For a named sub-list, put it in a struct.

### ol()

`ol(list, attributes)`

- `list` (array or struct): the list entries.
- `attributes` (string or struct, optional, default `""`): the HTML attributes.
- Returns an HTML ordered list element.

Identical to [`ul()`](#ul), only it produces an `<ol>` element for ordered lists instead of `<ul>`.

### video()

`video(src, unsupportedMessage, attributes, tracks, indexPage)`

- `src` (string or array): either a source string or an array of sources. See the [`source()`](#source) function.
- `unsupportedMessage` (string, optional, default `""`): the message to show if the video element is not supported by the browser.
- `attributes` (string, optional, default `""`): the HTML attributes.
- `tracks` (array, optional): use the track function inside an array. See the [`track()`](#track) function.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the video source path.
- Returns an HTML video element.

Permits you to generate an HTML video element from a source string or an array of sources. Example:

```cfm
<cfset tracks = ArrayNew(1)>
<cfset ArrayAppend(tracks, track("subtitles_no.vtt", "subtitles", "no", "Norwegian No"))>
<cfset ArrayAppend(tracks, track("subtitles_yes.vtt", "subtitles", "yes", "Norwegian Yes"))>

<cfoutput>#video("test.mp4", "Your browser does not support the video tag.", "controls")#</cfoutput>

<cfoutput>#video("http://www.example.com/test.mp4", "Your browser does not support the video tag.", "controls", tracks)#</cfoutput>

<cfset sources = ArrayNew(1)>
<cfset ArrayAppend(sources, source("movie.mp4", "video/mp4", "class=""test"""))>
<cfset ArrayAppend(sources, source("movie.ogg", "video/ogg"))>
<cfset ArrayAppend(sources, source("movie.mov", "video/quicktime"))>
<cfset ArrayAppend(sources, source("movie.ogv", "video/ogv; codecs=dirac, speex"))>

<cfoutput>#video(sources, "Your browser does not support the video tag.", "class=""test"" controls", tracks)#</cfoutput>
```

The above code produces this:

```html
<video src="http://example.com/test.mp4" controls>
  Your browser does not support the video tag.
</video>

<video src="http://www.example.com/test.mp4" controls>
  <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No">
  <track src="subtitles_yes.vtt" kind="subtitles" srclang="yes" label="Norwegian Yes">
  Your browser does not support the video tag.
</video>

<video class="test" controls>
  <source src="http://example.com/movie.mp4" type="video/mp4" class="test">
  <source src="http://example.com/movie.ogg" type="video/ogg">
  <source src="http://example.com/movie.mov" type="video/quicktime">
  <source src="http://example.com/movie.ogv" type="video/ogv; codecs=dirac, speex">
  <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No">
  <track src="subtitles_yes.vtt" kind="subtitles" srclang="yes" label="Norwegian Yes">
  Your browser does not support the video tag.
</video>
```

### audio()

`audio(src, unsupportedMessage, attributes, tracks, indexPage)`

- `src` (string or array): either a source string or an array of sources. See the [`source()`](#source) function.
- `unsupportedMessage` (string, optional, default `""`): the message to show if the audio element is not supported by the browser.
- `attributes` (string, optional, default `""`): the HTML attributes.
- `tracks` (array, optional): use the track function inside an array. See the [`track()`](#track) function.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the audio source path.
- Returns an HTML audio element.

Works like [`video()`](#video), only it produces an `<audio>` element instead of `<video>`. With a single source string, the line breaks of the output differ a little: there is no line break after the opening tag when you give no tracks and no message.

### source()

`source(src, type, attributes, indexPage)`

- `src` (string): the path of the media resource.
- `type` (string, optional, default `unknown`): the MIME-type of the resource with optional codecs parameters.
- `attributes` (string, optional, default `""`): the HTML attributes.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the source path.
- Returns an HTML source element.

Lets you create HTML `<source>` elements. The first parameter contains the path of the resource. Example:

```cfm
<cfoutput>#source("movie.mp4", "video/mp4", "class=""test""")#</cfoutput>
<!--- <source src="http://example.com/movie.mp4" type="video/mp4" class="test"> --->
```

### embed()

`embed(src, type, attributes, indexPage)`

- `src` (string): the path of the resource to embed.
- `type` (string, optional, default `unknown`): the MIME-type.
- `attributes` (string, optional, default `""`): the HTML attributes.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the source path.
- Returns an HTML embed element.

Lets you create HTML `<embed>` elements. The first parameter contains the embed source. Example:

```cfm
<cfoutput>#embed("movie.mov", "video/quicktime", "class=""test""")#</cfoutput>
<!--- <embed src="http://example.com/movie.mov" type="video/quicktime" class="test"> --->
```

### object()

`object(data, type, attributes, params, indexPage)`

- `data` (string): a resource URL.
- `type` (string, optional, default `unknown`): the content-type of the resource.
- `attributes` (string, optional, default `""`): the HTML attributes.
- `params` (array, optional): use the param function inside an array. See the [`param()`](#param) function.
- `indexPage` (boolean, optional, default `false`): whether `indexPage` is added to the resource URL.
- Returns an HTML object element.

Lets you create HTML `<object>` elements. The first parameter contains the object data. Example:

```cfm
<cfoutput>#object("movie.swf", "application/x-shockwave-flash", "class=""test""")#</cfoutput>

<cfset params = ArrayNew(1)>
<cfset ArrayAppend(params, param("foo", "bar", "ref", "class=""test"""))>
<cfset ArrayAppend(params, param("hello", "world", "ref", "class=""test"""))>

<cfoutput>#object("movie.swf", "application/x-shockwave-flash", "class=""test""", params)#</cfoutput>
```

The above code produces this:

```html
<object data="http://example.com/movie.swf" class="test"></object>

<object data="http://example.com/movie.swf" class="test">
  <param name="foo" type="ref" value="bar" class="test">
  <param name="hello" type="ref" value="world" class="test">
</object>
```

### param()

`param(name, value, type, attributes)`

- `name` (string): the name of the parameter.
- `value` (string): the value of the parameter.
- `type` (string, optional, default `ref`): the type of the parameter.
- `attributes` (string, optional, default `""`): the HTML attributes.
- Returns an HTML param element.

Lets you create HTML `<param>` elements. The first parameter contains the name of the param. Example:

```cfm
<cfoutput>#param("foo", "bar", "ref", "class=""test""")#</cfoutput>
<!--- <param name="foo" type="ref" value="bar" class="test"> --->
```

### track()

`track(src, kind, srcLanguage, label)`

- `src` (string): the path of the track (.vtt file).
- `kind` (string): the kind of timed track.
- `srcLanguage` (string): the language of the timed track.
- `label` (string): a user-readable title for the timed track.
- Returns an HTML track element.

Generates a track element to specify timed tracks. The tracks are formatted in WebVTT format. Example:

```cfm
<cfoutput>#track("subtitles_no.vtt", "subtitles", "no", "Norwegian No")#</cfoutput>
<!--- <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No"> --->
```

### doctype()

`doctype(type)`

- `type` (string, optional, default `html5`): the doctype name.
- Returns an HTML DocType declaration, or an empty string for a name that is not in the list.

Helps you generate document type declarations (DTDs). HTML 5 is used by default, but many doctypes are available.

Example:

```cfm
<cfoutput>#doctype()#</cfoutput>
<!--- <!DOCTYPE html> --->

<cfoutput>#doctype("html4-trans")#</cfoutput>
<!--- <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd"> --->
```

The following is a list of the pre-defined doctypes. These are pulled from `app/Config/DocTypes.cfc`.

| Document type | `type` parameter | Result |
| --- | --- | --- |
| XHTML 1.1 | xhtml11 | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">` |
| XHTML 1.0 Strict | xhtml1-strict | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">` |
| XHTML 1.0 Transitional | xhtml1-trans | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">` |
| XHTML 1.0 Frameset | xhtml1-frame | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">` |
| XHTML Basic 1.1 | xhtml-basic11 | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN" "http://www.w3.org/TR/xhtml-basic/xhtml-basic11.dtd">` |
| HTML 5 | html5 | `<!DOCTYPE html>` |
| HTML 4 Strict | html4-strict | `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">` |
| HTML 4 Transitional | html4-trans | `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">` |
| HTML 4 Frameset | html4-frame | `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">` |
| MathML 1.01 | mathml1 | `<!DOCTYPE math SYSTEM "http://www.w3.org/Math/DTD/mathml1/mathml.dtd">` |
| MathML 2.0 | mathml2 | `<!DOCTYPE math PUBLIC "-//W3C//DTD MathML 2.0//EN" "http://www.w3.org/Math/DTD/mathml2/mathml2.dtd">` |
| SVG 1.0 | svg10 | `<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">` |
| SVG 1.1 Full | svg11 | `<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">` |
| SVG 1.1 Basic | svg11-basic | `<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1 Basic//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11-basic.dtd">` |
| SVG 1.1 Tiny | svg11-tiny | `<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1 Tiny//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11-tiny.dtd">` |
| XHTML+MathML+SVG (XHTML host) | xhtml-math-svg-xh | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1 plus MathML 2.0 plus SVG 1.1//EN" "http://www.w3.org/2002/04/xhtml-math-svg/xhtml-math-svg.dtd">` |
| XHTML+MathML+SVG (SVG host) | xhtml-math-svg-sh | `<!DOCTYPE svg:svg PUBLIC "-//W3C//DTD XHTML 1.1 plus MathML 2.0 plus SVG 1.1//EN" "http://www.w3.org/2002/04/xhtml-math-svg/xhtml-math-svg.dtd">` |
| XHTML+RDFa 1.0 | xhtml-rdfa-1 | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.0//EN" "http://www.w3.org/MarkUp/DTD/xhtml-rdfa-1.dtd">` |
| XHTML+RDFa 1.1 | xhtml-rdfa-2 | `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.1//EN" "http://www.w3.org/MarkUp/DTD/xhtml-rdfa-2.dtd">` |
