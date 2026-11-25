# Form Helper

The Form Helper file contains functions that assist in working with forms.

## Configuration

The Form Helper reads `app/Config/DocTypes.cfc` to decide how to close void elements such as `<input>`. The `html5` property is `true` by default, which writes `<input>`. Set it to `false` if you need XHTML output, which writes `<input />`.

## Loading this Helper

This helper is loaded using the following code:

```cfm
<cfset helper("form")>
```

## Escaping Field Values

You may need to use HTML and characters such as quotes within your form elements. To do that safely, use the common function [`esc()`](../general/common_functions.md#esc).

Consider the following example:

```cfm
<cfset string = "Here is a string containing ""quoted"" text.">

<cfoutput><input type="text" name="myfield" value="#string#"></cfoutput>
```

Since the above string contains a set of quotes, it will cause the form to break. The `esc()` function converts HTML special characters so that the string can be used safely:

```cfm
<cfoutput><input type="text" name="myfield" value="#esc(string)#"></cfoutput>
```

!!! note
    If you use any of the form helper functions listed on this page, and you pass the values as a map, the form values are escaped automatically. There is no need to call `esc()` on them. Use it only if you write your own form elements, which you pass as strings.

## Available Functions

The following functions are available:

### form_open()

`form_open(action, attributes, hidden)`

- `action` (string, optional, default empty string): the form action or target URI string.
- `attributes` (struct or string, optional): HTML attributes.
- `hidden` (struct, optional): a map of hidden field names and values.
- Returns an HTML form opening tag.

Creates an opening form tag with a site URL **built from your** `app/Config/App.cfc` **file**. You can add form attributes and hidden input fields. If you give no `accept-charset` attribute, the function adds it with the value of the `charset` property in your `app/Config/App.cfc` file. If you give no method, it adds `method="post"`.

The main benefit of using this function, and not writing the HTML yourself, is that your site is more portable in the event your URLs ever change.

Here is a simple example:

```cfm
<cfoutput>#form_open("email/send")#</cfoutput>
```

The above example creates a form that points to your site URL plus the "email/send" URI segments, like this:

```html
<form action="http://example.com/index.cfm/email/send" method="post" accept-charset="utf-8">
```

If you leave `action` empty, the form points to the current URL.

You can also add `{locale}` like the following:

```cfm
<cfoutput>#form_open("{locale}/email/send")#</cfoutput>
```

The above example creates a form that points to your site URL plus the current request locale with the "email/send" URI segments, like this:

```html
<form action="http://example.com/index.cfm/en/email/send" method="post" accept-charset="utf-8">
```

**Adding Attributes**

Attributes can be added by passing a map as the second parameter, like this:

```cfm
<cfset attributes = StructNew()>
<cfset attributes["class"] = "email">
<cfset attributes["id"] = "myform">
<cfoutput>#form_open("email/send", attributes)#</cfoutput>
```

Alternatively, you can specify the second parameter as a string:

```cfm
<cfoutput>#form_open("email/send", "class=""email"" id=""myform""")#</cfoutput>
```

The above examples create a form similar to this:

```html
<form action="http://example.com/index.cfm/email/send" class="email" id="myform" method="post" accept-charset="utf-8">
```

If the CSRF filter is turned on, `form_open()` writes the CSRF field at the beginning of the form. You can give the ID of this field by passing `csrf_id` as an element of the `attributes` map:

```cfm
<cfset attributes = StructNew()>
<cfset attributes["csrf_id"] = "my-id">
<cfoutput>#form_open("/u/sign-up", attributes)#</cfoutput>
```

It returns:

```html
<form action="http://example.com/index.cfm/u/sign-up" method="post" accept-charset="utf-8">
<input type="hidden" id="my-id" name="csrf_test_name" value="964ede6e0ae8a680f7b8eab69136717d">
```

!!! note
    To use the automatic CSRF field, you need to turn the CSRF filter on in the `app/Config/Filters.cfc` file. See [Controller Filters](../incoming/filters.md). The page of the Security library is not written yet. The field is left out when the form uses the GET method and when the form points to another site.

**Adding Hidden Input Fields**

Hidden fields can be added by passing a map as the third parameter, like this:

```cfm
<cfset hidden = StructNew()>
<cfset hidden["username"] = "Joe">
<cfset hidden["member_id"] = "234">
<cfoutput>#form_open("email/send", "", hidden)#</cfoutput>
```

You can skip the second parameter by passing an empty string to it.

The above example creates a form similar to this:

```html
<form action="http://example.com/index.cfm/email/send" method="post" accept-charset="utf-8">
<input type="hidden" name="username" value="Joe">
<input type="hidden" name="member_id" value="234">
```

!!! note "Difference from CodeIgniter"
    A CFML struct has no order. Where you pass a struct to a function on this page, the attributes, hidden fields or options can come out in an order other than the one you set them in. The functions look for keys such as `name` and `value` in lower case, and dot notation stores a key in upper case. So write the keys with bracket notation and in lower case, such as `data["name"]`.

### form_open_multipart()

`form_open_multipart(action, attributes, hidden)`

- `action` (string, optional, default empty string): the form action or target URI string.
- `attributes` (struct or string, optional): HTML attributes.
- `hidden` (struct, optional): a map of hidden field names and values.
- Returns an HTML multipart form opening tag.

This function is identical to [`form_open()`](#form_open) above, except that it adds a *multipart* attribute, which is necessary if you want to use the form to upload files.

### form_hidden()

`form_hidden(name, value)`

- `name` (string or struct): the field name, or a map of field names and values.
- `value` (string, array or struct, optional, default empty string): the field value.
- Returns an HTML hidden input field tag.

Lets you create hidden input fields. You can pass a name and a value to create one field:

```cfm
<cfoutput>#form_hidden("username", "johndoe")#</cfoutput>
<!--- Would produce: <input type="hidden" name="username" value="johndoe"> --->
```

Or you can pass a map to create multiple fields:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "John Doe">
<cfset data["email"] = "john@example.com">
<cfset data["url"] = "http://example.com">

<cfoutput>#form_hidden(data)#</cfoutput>
```

Would produce:

```html
<input type="hidden" name="name" value="John Doe">
<input type="hidden" name="email" value="john@example.com">
<input type="hidden" name="url" value="http://example.com">
```

You can also pass a map to the value field:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "John Doe">
<cfset data["email"] = "john@example.com">
<cfset data["url"] = "http://example.com">

<cfoutput>#form_hidden("my_array", data)#</cfoutput>
```

Would produce:

```html
<input type="hidden" name="my_array[name]" value="John Doe">
<input type="hidden" name="my_array[email]" value="john@example.com">
<input type="hidden" name="my_array[url]" value="http://example.com">
```

If you pass a CFML array as the value, each field is named with empty brackets, such as `my_array[]`.

If you want to create hidden input fields with extra attributes, use [`form_input()`](#form_input):

```cfm
<cfset data = StructNew()>
<cfset data["type"] = "hidden">
<cfset data["name"] = "email">
<cfset data["id"] = "hiddenemail">
<cfset data["value"] = "john@example.com">
<cfset data["class"] = "hiddenemail">

<cfoutput>#form_input(data)#</cfoutput>
```

Would produce:

```html
<input type="hidden" name="email" value="john@example.com" id="hiddenemail" class="hiddenemail">
```

### form_input()

`form_input(data, value, extra, type)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- `type` (string, optional, default `text`): the type of input field, such as `text`, `email` or `number`.
- Returns an HTML text input field tag.

Lets you generate a standard text input field. You can minimally pass the field name and value in the first and second parameter:

```cfm
<cfoutput>#form_input("username", "johndoe")#</cfoutput>
<!--- Would produce: <input type="text" name="username" value="johndoe"> --->
```

Or you can pass a map that contains any data you want your form to contain:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "username">
<cfset data["id"] = "username">
<cfset data["value"] = "johndoe">
<cfset data["maxlength"] = "100">
<cfset data["size"] = "50">
<cfset data["style"] = "width:50%">

<cfoutput>#form_input(data)#</cfoutput>
```

Would produce:

```html
<input type="text" name="username" value="johndoe" id="username" maxlength="100" size="50" style="width:50%">
```

If you want boolean attributes, pass a boolean value (`true` or `false`). In this case the boolean value does not matter:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "username">
<cfset data["id"] = "username">
<cfset data["value"] = "">
<cfset data["required"] = JavaCast("boolean", true)>

<cfoutput>#form_input(data)#</cfoutput>
```

Would produce:

```html
<input type="text" name="username" value="" id="username" required >
```

!!! note "Difference from CodeIgniter"
    The value of a boolean attribute must have the boolean type. `JavaCast("boolean", true)` always gives one. A string such as `"true"` is written as an attribute with a value, like `required="true"`.

If you would like your form to contain some additional data, like JavaScript, you can pass it as a string in the third parameter:

```cfm
<cfset js = "onClick=""some_function ()""">
<cfoutput>#form_input("username", "johndoe", js)#</cfoutput>
<!--- Would produce: <input type="text" name="username" value="johndoe" onClick="some_function ()"> --->
```

Or you can pass it as a map:

```cfm
<cfset js = StructNew()>
<cfset js["onClick"] = "some_function ();">
<cfoutput>#form_input("username", "johndoe", js)#</cfoutput>
<!--- Would produce: <input type="text" name="username" value="johndoe" onClick="some_function ();"> --->
```

To support the expanded range of HTML5 input fields, you can pass an input type in as the fourth parameter:

```cfm
<cfset extra = StructNew()>
<cfset extra["placeholder"] = "Email Address...">
<cfoutput>#form_input("email", "joe@example.com", extra, "email")#</cfoutput>
<!--- Would produce: <input type="email" name="email" value="joe@example.com" placeholder="Email Address..."> --->
```

### form_password()

`form_password(data, value, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML password input field tag.

This function is identical in all respects to the [`form_input()`](#form_input) function above, except that it uses the "password" input type.

### form_upload()

`form_upload(data, value, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML file upload input field tag.

This function is identical in all respects to the [`form_input()`](#form_input) function above, except that it uses the "file" input type, so it can be used to upload files.

### form_textarea()

`form_textarea(data, value, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML textarea tag.

This function is identical in all respects to the [`form_input()`](#form_input) function above, except that it generates a "textarea" type. If you give no `cols` and `rows`, the tag gets `cols="40"` and `rows="10"`.

!!! note
    Instead of the *maxlength* and *size* attributes in the above example, you specify *rows* and *cols*.

### form_dropdown()

`form_dropdown(data, options, selected, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `options` (struct, optional): a map of the options to be listed.
- `selected` (string or array, optional): the value or the list of values to mark with the *selected* attribute.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML dropdown select field tag.

Lets you create a standard drop-down field. The first parameter contains the name of the field, the second parameter contains a map of options, and the third parameter contains the value you wish to be selected. You can also pass an array of multiple items through the third parameter, and the helper creates a multiple select for you.

Example:

```cfm
<cfset options = StructNew()>
<cfset options["small"] = "Small Shirt">
<cfset options["med"] = "Medium Shirt">
<cfset options["large"] = "Large Shirt">
<cfset options["xlarge"] = "Extra Large Shirt">

<cfoutput>#form_dropdown("shirts", options, "large")#</cfoutput>
```

Would produce:

```html
<select name="shirts">
<option value="small">Small Shirt</option>
<option value="med">Medium Shirt</option>
<option value="large" selected="selected">Large Shirt</option>
<option value="xlarge">Extra Large Shirt</option>
</select>
```

With a list of selected values:

```cfm
<cfset shirts_on_sale = ListToArray("small,large")>
<cfoutput>#form_dropdown("shirts", options, shirts_on_sale)#</cfoutput>
```

Would produce:

```html
<select name="shirts" multiple="multiple">
<option value="small" selected="selected">Small Shirt</option>
<option value="med">Medium Shirt</option>
<option value="large" selected="selected">Large Shirt</option>
<option value="xlarge">Extra Large Shirt</option>
</select>
```

If you would like the opening `<select>` to contain additional data, like an `id` attribute or JavaScript, you can pass it as a string in the fourth parameter:

```cfm
<cfset js = "id=""shirts"" onChange=""some_function();""">
<cfoutput>#form_dropdown("shirts", options, "large", js)#</cfoutput>
```

Or you can pass it as a map:

```cfm
<cfset js = StructNew()>
<cfset js["id"] = "shirts">
<cfset js["onChange"] = "some_function();">
<cfoutput>#form_dropdown("shirts", options, "large", js)#</cfoutput>
```

If the map passed as `options` holds a map as one of its values, then `form_dropdown()` produces an `<optgroup>` with the key as the label.

### form_multiselect()

`form_multiselect(name, options, selected, extra)`

- `name` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `options` (struct, optional): a map of the options to be listed.
- `selected` (string or array, optional): the value or the list of values to mark with the *selected* attribute.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML dropdown multiselect field tag.

Lets you create a standard multiselect field. The first parameter contains the name of the field, the second parameter contains a map of options, and the third parameter contains the value or values you wish to be selected.

The parameter usage is identical to using [`form_dropdown()`](#form_dropdown) above, except of course that the name of the field needs to use array syntax, such as `foo[]`.

### form_fieldset()

`form_fieldset(legendText, attributes)`

- `legendText` (string, optional, default empty string): the text to put in the `<legend>` tag.
- `attributes` (struct or string, optional): attributes to be set on the `<fieldset>` tag.
- Returns an HTML fieldset opening tag.

Lets you generate fieldset and legend fields.

Example:

```cfm
<cfoutput>
#form_fieldset("Address Information")#
<p>fieldset content here</p>
#form_fieldset_close()#
</cfoutput>
```

Produces:

```html
<fieldset>
<legend>Address Information</legend>
<p>fieldset content here</p>
</fieldset>
```

Similar to other functions, you can pass a map in the second parameter if you prefer to set additional attributes:

```cfm
<cfset attributes = StructNew()>
<cfset attributes["id"] = "address_info">
<cfset attributes["class"] = "address_info">

<cfoutput>
#form_fieldset("Address Information", attributes)#
<p>fieldset content here</p>
#form_fieldset_close()#
</cfoutput>
```

Produces:

```html
<fieldset id="address_info" class="address_info">
<legend>Address Information</legend>
<p>fieldset content here</p>
</fieldset>
```

### form_fieldset_close()

`form_fieldset_close(extra)`

- `extra` (string, optional, default empty string): anything to append after the closing tag, *as is*.
- Returns an HTML fieldset closing tag.

Produces a closing `</fieldset>` tag. The only advantage to using this function is that it permits you to pass data to it, which is added below the tag. For example:

```cfm
<cfset string = "</div></div>">
<cfoutput>#form_fieldset_close(string)#</cfoutput>
<!--- Would produce: </fieldset></div></div> --->
```

### form_checkbox()

`form_checkbox(data, value, checked, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `checked` (boolean, optional, default `false`): whether to mark the checkbox as *checked*.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML checkbox input tag.

Lets you generate a checkbox field. Simple example:

```cfm
<cfoutput>#form_checkbox("newsletter", "accept", true)#</cfoutput>
<!--- Would produce: <input type="checkbox" name="newsletter" value="accept" checked="checked"> --->
```

The third parameter contains a boolean `true`/`false` to determine whether the box should be checked or not.

Similar to the other form functions in this helper, you can also pass a map of attributes to the function:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "newsletter">
<cfset data["id"] = "newsletter">
<cfset data["value"] = "accept">
<cfset data["checked"] = true>
<cfset data["style"] = "margin:10px">

<cfoutput>#form_checkbox(data)#</cfoutput>
```

Would produce:

```html
<input type="checkbox" name="newsletter" value="accept" checked="checked" id="newsletter" style="margin:10px">
```

Also as with other functions, if you would like the tag to contain additional data like JavaScript, you can pass it as a string in the fourth parameter:

```cfm
<cfset js = "onClick=""some_function()""">
<cfoutput>#form_checkbox("newsletter", "accept", true, js)#</cfoutput>
```

Or you can pass it as a map:

```cfm
<cfset js = StructNew()>
<cfset js["onClick"] = "some_function();">
<cfoutput>#form_checkbox("newsletter", "accept", true, js)#</cfoutput>
```

### form_radio()

`form_radio(data, value, checked, extra)`

- `data` (string or struct, optional, default empty string): the field name, or a map of field attributes.
- `value` (string, optional, default empty string): the field value.
- `checked` (boolean, optional, default `false`): whether to mark the radio button as *checked*.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML radio input tag.

This function is identical in all respects to the [`form_checkbox()`](#form_checkbox) function above, except that it uses the "radio" input type.

### form_label()

`form_label(labelText, id, attributes)`

- `labelText` (string, optional, default empty string): the text to put in the `<label>` tag.
- `id` (string, optional, default empty string): the ID of the form element that the label is made for.
- `attributes` (struct, optional): HTML attributes.
- Returns an HTML field label tag.

Lets you generate a `<label>`. Simple example:

```cfm
<cfoutput>#form_label("What is your Name", "username")#</cfoutput>
<!--- Would produce: <label for="username">What is your Name</label> --->
```

Similar to other functions, you can pass a map in the third parameter if you prefer to set additional attributes.

Example:

```cfm
<cfset attributes = StructNew()>
<cfset attributes["class"] = "mycustomclass">
<cfset attributes["style"] = "color: ##000;">

<cfoutput>#form_label("What is your Name", "username", attributes)#</cfoutput>
```

Would produce:

```html
<label for="username" class="mycustomclass" style="color: #000;">What is your Name</label>
```

### form_submit()

`form_submit(data, value, extra)`

- `data` (string or struct, optional, default empty string): the button name, or a map of attributes.
- `value` (string, optional, default empty string): the button value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML input submit tag.

Lets you generate a standard submit button. Simple example:

```cfm
<cfoutput>#form_submit("mysubmit", "Submit Post!")#</cfoutput>
<!--- Would produce: <input type="submit" name="mysubmit" value="Submit Post!"> --->
```

Similar to other functions, you can pass a map in the first parameter if you prefer to set your own attributes. The third parameter lets you add extra data to your form, like JavaScript.

### form_reset()

`form_reset(data, value, extra)`

- `data` (string or struct, optional, default empty string): the button name, or a map of attributes.
- `value` (string, optional, default empty string): the button value.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML input reset button tag.

Lets you generate a standard reset button. Use is identical to [`form_submit()`](#form_submit).

### form_button()

`form_button(data, content, extra)`

- `data` (string or struct, optional, default empty string): the button name, or a map of attributes.
- `content` (string, optional, default empty string): the button label.
- `extra` (string or struct, optional): extra attributes to be added to the tag, either as a map or as a string.
- Returns an HTML button tag.

Lets you generate a standard button element. You can minimally pass the button name and content in the first and second parameter:

```cfm
<cfoutput>#form_button("name", "content")#</cfoutput>
<!--- Would produce: <button name="name" type="button">content</button> --->
```

Or you can pass a map that contains any data you want your form to contain:

```cfm
<cfset data = StructNew()>
<cfset data["name"] = "button">
<cfset data["id"] = "button">
<cfset data["value"] = "true">
<cfset data["type"] = "reset">
<cfset data["content"] = "Reset">

<cfoutput>#form_button(data)#</cfoutput>
```

Would produce:

```html
<button name="button" type="reset" id="button" value="true">Reset</button>
```

If you would like your form to contain some additional data, like JavaScript, you can pass it as a string in the third parameter:

```cfm
<cfset js = "onClick=""some_function()""">
<cfoutput>#form_button("mybutton", "Click Me", js)#</cfoutput>
```

### form_close()

`form_close(extra)`

- `extra` (string, optional, default empty string): anything to append after the closing tag, *as is*.
- Returns an HTML form closing tag.

Produces a closing `</form>` tag. The only advantage to using this function is that it permits you to pass data to it, which is added below the tag. For example:

```cfm
<cfset string = "</div></div>">
<cfoutput>#form_close(string)#</cfoutput>
<!--- Would produce: </form></div></div> --->
```

### set_value()

`set_value(field, default, htmlEscape)`

- `field` (string): the field name.
- `default` (string or array, optional, default empty string): the default value.
- `htmlEscape` (boolean, optional, default `true`): whether to turn on HTML escaping of the value.
- Returns the field value.

Lets you set the value of an input form or textarea. You must supply the field name in the first parameter. The second (optional) parameter lets you set a default value for the form. The third (optional) parameter lets you turn off HTML escaping of the value, in case you need to use this function together with, for example, [`form_input()`](#form_input) and want to avoid double escaping.

Example:

```cfm
<cfoutput><input type="text" name="quantity" value="#set_value("quantity", "0")#" size="50"></cfoutput>
```

The above form shows "0" when it is loaded for the first time.

!!! note
    The value comes from the old input of a redirect made with `withInput()` (see [`redirect()`](../general/common_functions.md#redirect)) or, if there is none, from the posted data of the current request.

!!! note "Difference from CodeIgniter"
    CFML has no null value. An old or posted value that is an empty string counts as "not set", so the default is returned for it.

### set_select()

`set_select(field, value, default)`

- `field` (string): the field name.
- `value` (string, optional, default empty string): the value to check for.
- `default` (boolean, optional, default `false`): whether the value is also a default one.
- Returns the string ` selected="selected"`, or an empty string.

If you use a `<select>` menu, this function lets you display the menu item that was selected.

The first parameter must contain the name of the select menu, the second parameter must contain the value of each item, and the third (optional) parameter lets you set an item as the default (use boolean `true`/`false`).

Example:

```cfm
<cfoutput>
<select name="myselect">
    <option value="one" #set_select("myselect", "one", true)#>One</option>
    <option value="two" #set_select("myselect", "two")#>Two</option>
    <option value="three" #set_select("myselect", "three")#>Three</option>
</select>
</cfoutput>
```

### set_checkbox()

`set_checkbox(field, value, default)`

- `field` (string): the field name.
- `value` (string, optional, default empty string): the value to check for.
- `default` (boolean, optional, default `false`): whether the value is also a default one.
- Returns the string ` checked="checked"`, or an empty string.

Lets you display a checkbox in the state it was submitted.

The first parameter must contain the name of the checkbox, the second parameter must contain its value, and the third (optional) parameter lets you set an item as the default (use boolean `true`/`false`).

Example:

```cfm
<cfoutput>
<input type="checkbox" name="mycheck[]" value="1" #set_checkbox("mycheck", "1")#>
<input type="checkbox" name="mycheck[]" value="2" #set_checkbox("mycheck", "2")#>
</cfoutput>
```

### set_radio()

`set_radio(field, value, default)`

- `field` (string): the field name.
- `value` (string, optional, default empty string): the value to check for.
- `default` (boolean, optional, default `false`): whether the value is also a default one.
- Returns the string ` checked="checked"`, or an empty string.

Lets you display radio buttons in the state they were submitted. This function is identical to the [`set_checkbox()`](#set_checkbox) function above.

Example:

```cfm
<cfoutput>
<input type="radio" name="myradio" value="1" #set_radio("myradio", "1", true)#>
<input type="radio" name="myradio" value="2" #set_radio("myradio", "2")#>
</cfoutput>
```

### validation_errors()

`validation_errors()`

- Returns the validation errors, as a map of field names and error messages.

Returns the validation errors. First, this function checks the validation errors that are stored in the session. To store the errors in the session, you need to use `withInput()` with [`redirect()`](../general/common_functions.md#redirect).

The returned map is the same as the one of the `getErrors()` method of the Validation library.

Example:

```cfm
<cfset errors = validation_errors()>
```

### validation_list_errors()

`validation_list_errors(template)`

- `template` (string, optional, default `list`): the validation template name.
- Returns the rendered HTML of the validation errors.

Returns the rendered HTML of the validation errors.

The parameter `template` is a validation template name. The templates belong to the Validation library, whose page is not written yet.

This function uses [`validation_errors()`](#validation_errors) internally.

Example:

```cfm
<cfoutput>#validation_list_errors()#</cfoutput>
```

### validation_show_error()

`validation_show_error(field, template)`

- `field` (string): the field name.
- `template` (string, optional, default `single`): the validation template name.
- Returns the rendered HTML of the validation error.

Returns a single error for the specified field in formatted HTML.

The parameter `template` is a validation template name. The templates belong to the Validation library, whose page is not written yet.

This function uses [`validation_errors()`](#validation_errors) internally.

Example:

```cfm
<cfoutput>#validation_show_error("username")#</cfoutput>
```
