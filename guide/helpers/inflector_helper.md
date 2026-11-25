# Inflector Helper

The Inflector Helper file contains functions that permit you to change **English** words to plural, singular, camel case, and so on.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("inflector")>
```

## Available Functions

The following functions are available:

### singular()

`singular(string)`

- `string` (string): the input string.
- Returns a singular word.

Changes a plural word to singular. Example:

```cfm
<cfoutput>#singular("dogs")#</cfoutput> <!--- Prints 'dog' --->
```

### plural()

`plural(string)`

- `string` (string): the input string.
- Returns a plural word.

Changes a singular word to plural. Example:

```cfm
<cfoutput>#plural("dog")#</cfoutput> <!--- Prints 'dogs' --->
```

### counted()

`counted(count, string)`

- `count` (numeric): the number of items.
- `string` (string): the input string.
- Returns a singular or plural phrase.

Changes a word and its count to a phrase. Example:

```cfm
<cfoutput>#counted(3, "dog")#</cfoutput> <!--- Prints '3 dogs' --->
```

### camelize()

`camelize(string)`

- `string` (string): the input string.
- Returns the camel case string.

Changes a string of words separated by spaces or underscores to camel case. Example:

```cfm
<cfoutput>#camelize("my_dog_spot")#</cfoutput> <!--- Prints 'myDogSpot' --->
```

### pascalize()

`pascalize(string)`

- `string` (string): the input string.
- Returns the Pascal case string.

Changes a string of words separated by spaces or underscores to Pascal case, which is camel case with the first letter capitalized. Example:

```cfm
<cfoutput>#pascalize("my_dog_spot")#</cfoutput> <!--- Prints 'MyDogSpot' --->
```

### underscore()

`underscore(string)`

- `string` (string): the input string.
- Returns the string with underscores instead of spaces.

Takes multiple words separated by spaces and underscores them. Example:

```cfm
<cfoutput>#underscore("my dog spot")#</cfoutput> <!--- Prints 'my_dog_spot' --->
```

### decamelize()

`decamelize(string)`

- `string` (string): the input string.
- Returns the string with underscores between words.

Takes multiple words in camelCase or PascalCase and converts them to snake_case. Example:

```cfm
<cfoutput>#decamelize("myDogSpot")#</cfoutput> <!--- Prints 'my_dog_spot' --->
```

### humanize()

`humanize(string, separator)`

- `string` (string): the input string.
- `separator` (string, optional, default `_`): the input separator.
- Returns the humanized string.

Takes multiple words separated by underscores and adds spaces between them. Each word is capitalized.

Example:

```cfm
<cfoutput>#humanize("my_dog_spot")#</cfoutput> <!--- Prints 'My Dog Spot' --->
```

To use dashes instead of underscores:

```cfm
<cfoutput>#humanize("my-dog-spot", "-")#</cfoutput> <!--- Prints 'My Dog Spot' --->
```

### is_pluralizable()

`is_pluralizable(word)`

- `word` (string): the input string.
- Returns `true` if the word is countable or `false` if not.

Checks if the given word has a plural version. Example:

```cfm
<cfset result = is_pluralizable("equipment")> <!--- Returns false --->
```

### dasherize()

`dasherize(string)`

- `string` (string): the input string.
- Returns the dasherized string.

Replaces underscores with dashes in the string. Example:

```cfm
<cfset result = dasherize("hello_world")> <!--- Returns 'hello-world' --->
```

### ordinal()

`ordinal(integer)`

- `integer` (numeric): the integer to determine the suffix for.
- Returns the ordinal suffix.

Returns the suffix that should be added to a number to denote the position, such as 1st, 2nd, 3rd, 4th. Example:

```cfm
<cfset result = ordinal(1)> <!--- Returns 'st' --->
```

### ordinalize()

`ordinalize(integer)`

- `integer` (numeric): the integer to ordinalize.
- Returns the ordinalized integer.

Turns a number into an ordinal string used to denote the position, such as 1st, 2nd, 3rd, 4th. Example:

```cfm
<cfset result = ordinalize(1)> <!--- Returns '1st' --->
```

!!! note "Difference from CodeIgniter"
    `decamelize()` also changes letters outside ASCII (such as `É`) to lower case. CodeIgniter changes only the letters A to Z there.
