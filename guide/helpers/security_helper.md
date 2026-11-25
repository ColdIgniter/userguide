# Security Helper

The Security Helper file contains security related functions.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("security")>
```

## Available Functions

The following functions are available:

### sanitize_filename()

`sanitize_filename(filename, relativePath)`

- `filename` (string): the file name.
- `relativePath` (boolean, optional, default `false`): whether a relative path is acceptable.
- Returns the sanitized file name.

Provides protection against directory traversal.

Backslashes are always removed. If `relativePath` is `true`, forward slashes are kept to allow relative paths such as `path/to/file.txt`, while parent-directory sequences (`../`) are removed.

!!! note
    When you pass relative paths to this helper, use forward slashes (`/`) as directory separators, on Windows too.

For more information, see the Security library. Its page is not written yet.

### strip_image_tags()

`strip_image_tags(str)`

- `str` (string): the input string.
- Returns the input string with no image tags.

This is a security function that strips image tags from a string. It leaves the image URL as plain text.

Example:

```cfm
<cfset string = strip_image_tags(string)>
```

!!! note "Difference from CodeIgniter"
    The search works on one image tag at a time, and a tag may run over a line break. A tag that has a `<` character inside it is left as it is.

### encode_php_tags()

!!! warning "Not ported yet"
    `encode_php_tags()` is not ported yet.
