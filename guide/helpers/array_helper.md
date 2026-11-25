# Array Helper

The array helper provides several functions to simplify more complex usages of arrays. It is not intended to duplicate any of the functions that CFML or the framework already provides, unless it is to make their use much simpler.

In this page an "array" with string keys is a struct, and an array with number keys is a CFML array.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("array")>
```

## Available Functions

The following functions are available:

### dot_array_search()

`dot_array_search(index, array)`

- `index` (string): the dot-notation string that describes how to search the array.
- `array` (struct or array): the array to search.
- Returns the value found within the array, or an empty string.

This function allows you to use dot-notation to search through an array for a specific key, and allows the use of the `*` wildcard. Given the following array:

```cfm
<cfset data = StructNew()>
<cfset data["foo"] = StructNew()>
<cfset data["foo"]["buzz"] = StructNew()>
<cfset data["foo"]["buzz"]["fizz"] = 11>
<cfset data["foo"]["bar"] = StructNew()>
<cfset data["foo"]["bar"]["baz"] = 23>
```

We can locate the value of `fizz` by using the search string `foo.buzz.fizz`. Likewise, the value of `baz` can be found with `foo.bar.baz`:

```cfm
<!--- Returns: 11 --->
<cfset fizz = dot_array_search("foo.buzz.fizz", data)>

<!--- Returns: 23 --->
<cfset baz = dot_array_search("foo.bar.baz", data)>
```

You can use the asterisk (`*`) as a wildcard to replace any of the segments. When found, it searches through all of the child nodes until it finds the key. This is handy if you do not know the values, or if your values have a numeric index:

```cfm
<!--- Returns: 23 --->
<cfset baz = dot_array_search("foo.*.baz", data)>
```

If the array key contains a dot (`.`), then the key can be escaped with a backslash (`\`):

```cfm
<cfset data = StructNew()>
<cfset data["foo"] = StructNew()>
<cfset data["foo"]["bar.baz"] = 23>
<cfset data["foo.bar"] = StructNew()>
<cfset data["foo.bar"]["baz"] = 43>

<!--- Returns: 23 --->
<cfset barBaz = dot_array_search("foo.bar\.baz", data)>

<!--- Returns: 43 --->
<cfset fooBar = dot_array_search("foo\.bar.baz", data)>
```

!!! note "Difference from CodeIgniter"
    CodeIgniter returns `null` when nothing is found. CFML has no null, so ColdIgniter returns an empty string. A value that is an empty string and a key that does not exist look the same.

### array_deep_search()

!!! warning "Not ported yet"
    `array_deep_search()` is not ported yet.

### array_sort_by_multiple_keys()

!!! warning "Not ported yet"
    `array_sort_by_multiple_keys()` is not ported yet.

### array_flatten_with_dots()

`array_flatten_with_dots(array, id)`

- `array` (struct or array): the multidimensional array to flatten.
- `id` (string, optional, default `""`): an ID to put before the outer keys. Used internally for flattening keys.
- Returns the flattened array, as a map.

This function flattens a multidimensional array to a single key-value array by using dots as separators for the keys.

```cfm
<cfset arrayToFlatten = StructNew()>
<cfset arrayToFlatten["personal"] = StructNew()>
<cfset arrayToFlatten["personal"]["first_name"] = "john">
<cfset arrayToFlatten["personal"]["last_name"] = "smith">
<cfset arrayToFlatten["personal"]["age"] = "26">
<cfset arrayToFlatten["personal"]["address"] = "US">
<cfset arrayToFlatten["other_details"] = "marines officer">

<cfset flattened = array_flatten_with_dots(arrayToFlatten)>
```

On inspection, `flattened` has these keys and values:

| Key | Value |
| --- | --- |
| `personal.first_name` | `john` |
| `personal.last_name` | `smith` |
| `personal.age` | `26` |
| `personal.address` | `US` |
| `other_details` | `marines officer` |

You may use the `id` parameter on your own, but you are not required to do so. The function uses this parameter internally to track the flattened keys. If you supply an initial `id`, it is put before all keys.

```cfm
<!--- using the same data from above --->
<cfset flattened = array_flatten_with_dots(arrayToFlatten, "foo_")>
```

`flattened` now has the keys `foo_personal.first_name`, `foo_personal.last_name`, `foo_personal.age`, `foo_personal.address` and `foo_other_details`.

!!! note "Difference from CodeIgniter"
    The key of an entry of a CFML array is its position, counted from 0 as in CodeIgniter. A struct has no order, so the keys of the result come in no fixed order when the input is a struct.

### array_group_by()

!!! warning "Not ported yet"
    `array_group_by()` is not ported yet.
