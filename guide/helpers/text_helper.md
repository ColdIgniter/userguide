# Text Helper

The Text Helper file contains functions that assist in working with text.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("text")>
```

## Available Functions

The following functions are available:

### random_string()

`random_string(type, len)`

- `type` (string, optional, default `alnum`): the randomization type.
- `len` (numeric, optional, default `8`): the output string length.
- Returns a random string.

Generates a random string based on the type and length you specify. Useful for creating passwords or generating random hashes.

The first parameter specifies the type of string, the second parameter specifies the length. The following choices are available:

- **alpha**: a string with lower and uppercase letters only.
- **alnum**: an alphanumeric string with lower and uppercase characters.
- **numeric**: a numeric string.
- **nozero**: a numeric string with no zeros.
- **crypto**: a random string of hexadecimal digits, made from random bytes.

!!! note
    When you use **crypto**, you must set an even number as the second parameter. If you set an odd number, an exception of the type `ColdIgniter.InvalidArgumentException` is thrown. The same exception is thrown for a type that is not in the list above.

Usage example:

```cfm
<cfoutput>#random_string("alnum", 16)#</cfoutput>
```

### increment_string()

`increment_string(str, separator, first)`

- `str` (string): the input string.
- `separator` (string, optional, default `_`): the separator to append a duplicate number with.
- `first` (numeric, optional, default `1`): the starting number.
- Returns an incremented string.

Increments a string by appending a number to it or increasing the number. Useful for creating "copies" of a file or duplicating database content which has unique titles or slugs.

Usage example:

```cfm
<cfoutput>
#increment_string("file", "_")# <!--- "file_1" --->
#increment_string("file", "-", 2)# <!--- "file-2" --->
#increment_string("file_4")# <!--- "file_5" --->
</cfoutput>
```

### alternator()

`alternator(args)`

- `args` (strings): a variable number of arguments.
- Returns the alternated string.

Allows two or more items to be alternated between, when cycling through a loop. Example:

```cfm
<cfloop from="1" to="10" index="i">
	<cfoutput>#alternator("string one", "string two")#</cfoutput>
</cfloop>
```

You can add as many parameters as you want, and with each iteration of your loop the next item is returned.

```cfm
<cfloop from="1" to="10" index="i">
	<cfoutput>#alternator("one", "two", "three", "four", "five")#</cfoutput>
</cfloop>
```

!!! note
    To use multiple separate calls to this function, simply call the function with no arguments to re-initialize. The position is kept for the length of the request.

### reduce_double_slashes()

`reduce_double_slashes(str)`

- `str` (string): the input string.
- Returns a string with normalized slashes.

Converts double slashes in a string to a single slash, except those found in URL protocol prefixes (such as `http://`).

Example:

```cfm
<cfset string = "http://example.com//index.cfm">
<cfoutput>#reduce_double_slashes(string)#</cfoutput> <!--- results in "http://example.com/index.cfm" --->
```

### strip_slashes()

`strip_slashes(str)`

- `str` (string, array or struct): the input string, or an array of strings.
- Returns the string or strings with stripped slashes.

Removes any slashes from an array of strings.

Example:

```cfm
<cfset str = StructNew()>
<cfset str["question"] = "Is your name O\'reilly?">
<cfset str["answer"] = "No, my name is O\'connor.">

<cfset str = strip_slashes(str)>
```

The above returns an array with these values:

| Key | Value |
| --- | --- |
| `question` | `Is your name O'reilly?` |
| `answer` | `No, my name is O'connor.` |

!!! note
    For historical reasons, this function also accepts and handles a string input.

### reduce_multiples()

`reduce_multiples(str, character, trim)`

- `str` (string): the text to search in.
- `character` (string, optional, default `,`): the character to reduce.
- `trim` (boolean, optional, default `false`): whether to also trim the specified character.
- Returns the reduced string.

Reduces multiple instances of a particular character occurring directly after each other. Example:

```cfm
<cfset string = "Fred, Bill,, Joe, Jimmy">
<cfset string = reduce_multiples(string)> <!--- results in "Fred, Bill, Joe, Jimmy" --->
```

If the third parameter is set to `true`, it removes occurrences of the character at the beginning and the end of the string. Example:

```cfm
<cfset string = ",Fred, Bill,, Joe, Jimmy,">
<cfset string = reduce_multiples(string, ",", true)> <!--- results in "Fred, Bill, Joe, Jimmy" --->
```

### quotes_to_entities()

`quotes_to_entities(str)`

- `str` (string): the input string.
- Returns the string with quotes converted to HTML entities.

Converts single and double quotes in a string to the corresponding HTML entities. Example:

```cfm
<cfset string = "Joe's ""dinner""">
<cfset string = quotes_to_entities(string)> <!--- results in "Joe&#39;s &quot;dinner&quot;" --->
```

### strip_quotes()

`strip_quotes(str)`

- `str` (string): the input string.
- Returns the string with quotes stripped.

Removes single and double quotes from a string. Example:

```cfm
<cfset string = "Joe's ""dinner""">
<cfset string = strip_quotes(string)> <!--- results in "Joes dinner" --->
```

### word_limiter()

`word_limiter(str, limit, endChar)`

- `str` (string): the input string.
- `limit` (numeric, optional, default `100`): the limit.
- `endChar` (string, optional, default `&#8230;`): the end character (usually an ellipsis).
- Returns the word-limited string.

Truncates a string to the number of *words* specified. Example:

```cfm
<cfset string = "Here is a nice text string consisting of eleven words.">
<cfset string = word_limiter(string, 4)>
<!--- Returns:  Here is a nice --->
```

The third parameter is an optional suffix added to the string. By default it adds an ellipsis.

### character_limiter()

`character_limiter(string, limit, endChar)`

- `string` (string): the input string.
- `limit` (numeric, optional, default `500`): the number of characters.
- `endChar` (string, optional, default `&#8230;`): the end character (usually an ellipsis).
- Returns the character-limited string.

Truncates a string to the number of *characters* specified. It maintains the integrity of words, so the character count may be slightly more or less than what you specify.

Example:

```cfm
<cfset string = "Here is a nice text string consisting of eleven words.">
<cfset string = character_limiter(string, 20)>
<!--- Returns:  Here is a nice text string --->
```

The third parameter is an optional suffix added to the string. If it is left out, this helper uses an ellipsis.

!!! note
    If you need to truncate to an exact number of characters, see the [`ellipsize()`](#ellipsize) function below.

### ascii_to_entities()

`ascii_to_entities(str)`

- `str` (string): the input string.
- Returns a string with ASCII values converted to entities.

Converts ASCII values to character entities, including high ASCII and MS Word characters that can cause problems when used in a web page, so that they can be shown consistently regardless of browser settings or stored reliably in a database. It identifies characters outside the normal range (like accented characters).

Example:

```cfm
<cfset string = ascii_to_entities(string)>
```

### entities_to_ascii()

`entities_to_ascii(str, all)`

- `str` (string): the input string.
- `all` (boolean, optional, default `true`): whether to convert unsafe entities as well.
- Returns a string with HTML entities converted to ASCII characters.

This function does the opposite of `ascii_to_entities()`. It turns character entities back into ASCII.

!!! note "Difference from CodeIgniter"
    An entity with a code point above 65535 is left as it is.

### convert_accented_characters()

`convert_accented_characters(str)`

- `str` (string): the input string.
- Returns a string with accented characters converted.

Transliterates high ASCII characters to low ASCII equivalents. Useful when non-English characters need to be used where only standard ASCII characters are safely used, for instance, in URLs.

Example:

```cfm
<cfset string = convert_accented_characters(string)>
```

!!! note
    This function uses a companion config file, `app/Config/ForeignCharacters.cfc`, to define the "to" and "from" array for transliteration.

### word_censor()

`word_censor(str, censored, replacement)`

- `str` (string): the input string.
- `censored` (array): the list of bad words to censor.
- `replacement` (string, optional, default `""`): what to replace bad words with.
- Returns the censored string.

Enables you to censor words within a text string. The first parameter contains the original string. The second contains an array of words which you disallow. The third (optional) parameter can contain a replacement value for the words. If not specified, they are replaced with number signs: `####`.

Example:

```cfm
<cfset disallowed = ListToArray("darn,shucks,golly,phooey")>
<cfset string = word_censor(string, disallowed, "Beep!")>
```

!!! note "Difference from CodeIgniter"
    A word is replaced by one number sign for each of its characters. CodeIgniter counts the bytes of the word, which gives more signs for a word with accented letters.

### highlight_code()

!!! warning "Not ported yet"
    `highlight_code()` is not ported yet. A call throws an exception of the type `ColdIgniter.RuntimeException`.

### highlight_phrase()

`highlight_phrase(str, phrase, tagOpen, tagClose)`

- `str` (string): the input string.
- `phrase` (string): the phrase to highlight.
- `tagOpen` (string, optional, default `<mark>`): the opening tag used for the highlight.
- `tagClose` (string, optional, default `</mark>`): the closing tag for the highlight.
- Returns the string with a phrase highlighted via HTML.

Highlights a phrase within a text string. The first parameter contains the original string, the second contains the phrase you wish to highlight. The third and fourth parameters contain the opening and closing HTML tags you would like the phrase wrapped in.

Example:

```cfm
<cfset string = "Here is a nice text string about nothing in particular.">
<cfoutput>#highlight_phrase(string, "nice text", "<span style=""color:##990000;"">", "</span>")#</cfoutput>
```

The above code prints:

```html
Here is a <span style="color:#990000;">nice text</span> string about nothing in particular.
```

!!! note
    Older browsers might not support the HTML5 `mark` tag, so it is recommended that you insert the following CSS code into your stylesheet if you need to support such browsers:

    ```html
    mark {
        background: #ff0;
        color: #000;
    }
    ```

### word_wrap()

`word_wrap(str, charlim)`

- `str` (string): the input string.
- `charlim` (numeric, optional, default `76`): the character limit.
- Returns the word-wrapped string.

Wraps text at the specified *character* count while maintaining complete words.

Example:

```cfm
<cfset string = "Here is a simple string of text that will help us demonstrate this function.">
<cfoutput>#word_wrap(string, 25)#</cfoutput>
```

This would produce:

```html
Here is a simple string
of text that will help us
demonstrate this
function.
```

Excessively long words are split, but URLs are not.

### ellipsize()

`ellipsize(str, maxLength, position, ellipsis)`

- `str` (string): the input string.
- `maxLength` (numeric): the string length limit.
- `position` (numeric, optional, default `1`): the position to split at, from 0 to 1.
- `ellipsis` (string, optional, default `&amp;hellip;`): what to use as the ellipsis character.
- Returns the ellipsized string.

This function strips tags from a string, splits it at a defined maximum length, and inserts an ellipsis.

The first parameter is the string to ellipsize, the second is the number of characters in the final string. The third parameter is where in the string the ellipsis should appear from 0 to 1, left to right. For example, a value of 1 places the ellipsis at the right of the string, .5 in the middle, and 0 at the left.

An optional fourth parameter is the kind of ellipsis.

Example:

```cfm
<cfset str = "this_string_is_entirely_too_long_and_might_break_my_design.jpg">
<cfoutput>#ellipsize(str, 32, .5, "&hellip;")#</cfoutput>
```

Produces:

```html
this_string_is_e&hellip;ak_my_design.jpg
```

!!! note "Difference from CodeIgniter"
    The default of `ellipsis` is the text `&amp;hellip;`, where CodeIgniter has `&hellip;`. Pass `&hellip;` as the fourth parameter, as in the example, to get the ellipsis character in a web page.

### excerpt()

`excerpt(text, phrase, radius, ellipsis)`

- `text` (string): the text to extract an excerpt from.
- `phrase` (string, optional): the phrase or word to extract the text around. Leave it out for no phrase.
- `radius` (numeric, optional, default `100`): the number of characters before and after `phrase`.
- `ellipsis` (string, optional, default `...`): what to use as the ellipsis character.
- Returns the excerpt.

This function extracts `radius` number of characters before and after the central `phrase`, with an ellipsis before and after.

The first parameter is the text to extract an excerpt from, the second is the central word or phrase to count before and after. The third parameter is the number of characters to count before and after the central phrase. If no phrase is passed, the excerpt includes the first `radius` characters with the ellipsis at the end.

Example:

```cfm
<cfset text = "Ut vel faucibus odio. Quisque quis congue libero. Etiam gravida eros lorem, eget porttitor augue dignissim tincidunt. In eget risus eget mauris faucibus molestie vitae ultricies odio. Vestibulum id ultricies diam. Curabitur non mauris lectus. Phasellus eu sodales sem. Integer dictum purus ac enim hendrerit gravida. Donec ac magna vel nunc tincidunt molestie sed vitae nisl. Cras sed auctor mauris, non dictum tortor. Nulla vel scelerisque arcu. Cras ac ipsum sit amet augue laoreet laoreet. Aenean a risus lacus. Sed ut tortor diam.">
<cfoutput>#excerpt(text, "Donec")#</cfoutput>
```

Produces:

```html
... non mauris lectus. Phasellus eu sodales sem. Integer dictum purus ac enim hendrerit gravida. Donec ac magna vel nunc tincidunt molestie sed vitae nisl. Cras sed auctor mauris, non dictum tortor. ...
```

!!! note "Difference from CodeIgniter"
    CodeIgniter takes `false` for "no phrase". Here the `phrase` argument is left out. The phrase is found without regard to upper and lower case.
