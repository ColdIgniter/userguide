# XML Helper

The XML Helper file contains functions that assist in working with XML data.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("xml")>
```

## Available Functions

The following functions are available:

### xml_convert()

`xml_convert(str, protectAll)`

- `str` (string): the text string to convert.
- `protectAll` (boolean, optional, default `false`): whether to protect all content that looks like a potential entity, such as `&foo;`, instead of just numbered entities.
- Returns the XML-converted string.

Takes a string as input and converts the following reserved XML characters to entities:

- Ampersands: `&`
- Less than and greater than characters: `<` `>`
- Single and double quotes: `'` `"`
- Dashes: `-`

This function ignores ampersands if they are part of existing numbered character entities, such as `&#123;`. Example:

```cfm
<cfset string = "<p>Here is a paragraph & an entity (&##123;).</p>">
<cfset string = xml_convert(string)>
<cfoutput>#string#</cfoutput>
```

outputs:

```html
&lt;p&gt;Here is a paragraph &amp; an entity (&#123;).&lt;/p&gt;
```

!!! note
    In a CFML string a number sign is written twice (`##`), as in the example above.
