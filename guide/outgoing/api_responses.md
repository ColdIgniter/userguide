# API Responses

Much of modern web development requires building APIs, whether simply to provide data for a javascript-heavy
single page application, or as a standalone product. ColdIgniter provides a set of methods that can be
used in a controller to make common response types simple, with no need to remember which HTTP status code
should be returned for which response types.

!!! note "Difference from CodeIgniter"
    In CodeIgniter the `ResponseTrait` can be used with any controller. In ColdIgniter the trait is a template file
    (**system/API/ResponseTrait.cfm**) that is included by `ColdIgniter\RESTful\ResourceController`. So a controller that
    extends `ResourceController` has the methods on this page, as the `Api/Customers` example of the
    [RESTful](../incoming/restful.md) page does. The methods are private: call them from inside the controller.

## Response Examples

The following example shows a common usage pattern within your controllers.

```cfm
<cfcomponent output="no" parent="ColdIgniter\RESTful\ResourceController">

	<cffunction name="createUser" access="public" returntype="any" output="no">
		<cfset var data = StructNew()>

		<!--- Create a new resource --->
		<cfset data["id"] = 7>
		<cfset data["username"] = "admin">

		<cfreturn respondCreated(data)>
	</cffunction>

</cfcomponent>
```

In this example, an HTTP status code of 201 is returned, with the generic status message, 'Created'. Methods
exist for the most common use cases:

```cfm
<cfreturn failValidationErrors(errors)>
<cfreturn failNotFound("No customer with this id")>
<cfreturn failServerError()>
```

## Handling Response Types

When you pass your data in any of these methods, they will determine the data type to format the results as based on
the following criteria:

* The format is determined according to the controller's `this.format_` value.
  If that is empty, it will try to negotiate the content type with what the
  client asked for, defaulting to the first element (JSON by default) in the
  `supportedResponseFormats` property within **app/Config/Format.cfc**.
* The data will be formatted according to the format. If the format is not JSON
  and data is a string, it will be treated as HTML to send back to the client. If the format is JSON, a string stays JSON, unless
  the controller sets `this.stringAsHtml` to true.

!!! note "Difference from CodeIgniter"
    The property is `this.format_`, with an underscore, because `format()` is a method of the trait.
    A number cannot be told from a string in CFML, so with a format other than JSON a number is sent as text.
    An empty string stands for CodeIgniter's `null`.

To define the formatter that is used, edit **app/Config/Format.cfc**. The `supportedResponseFormats` contains a list of
mime types that your application can automatically format the response for. By default, the system knows how to
format both XML and JSON responses:

```cfm
<cfset this.supportedResponseFormats = ArrayNew(1)>
<cfset ArrayAppend(this.supportedResponseFormats, "application/json")>
<cfset ArrayAppend(this.supportedResponseFormats, "application/xml")>
<cfset ArrayAppend(this.supportedResponseFormats, "text/xml")>
```

The setting `jsonEncodeDepth` in the same file defines the maximum depth of the JSON, with a default of `512`.

This is the list that is used during [Content Negotiation](../incoming/content_negotiation.md) to determine which
type of response to return. If no matches are found between what the client requested and what you support, the first
format in this list is what will be returned.

Next, you need to define the class that is used to format the data. This must be a fully qualified class
name, and the class must implement `ColdIgniter\Format\FormatterInterface`. Formatters come out of the box that
support both JSON and XML. They are set in the `formatters` setting of **app/Config/Format.cfc**:

```cfm
<cfset this.formatters["application/json"] = "ColdIgniter\Format\JSONFormatter">
<cfset this.formatters["application/xml"] = "ColdIgniter\Format\XMLFormatter">
<cfset this.formatters["text/xml"] = "ColdIgniter\Format\XMLFormatter">
```

So, if your request asks for JSON formatted data in an **Accept** header, the data you pass any of the
`respond*` or `fail*` methods will be formatted by the `ColdIgniter\Format\JSONFormatter` class. The resulting
JSON data will be sent back to the client.

## Class Reference

### setResponseFormat

`setResponseFormat(format)`

- `format` (string): the type of response to return, either `json` or `xml`

This defines the format to be used when formatting data in responses. If you leave `format` empty, it will be
automatically determined through content negotiation.

```cfm
<cfset setResponseFormat("json")>
```

### respond

`respond(data [, statusCode [, message]])`

- `data` (any): the data to return to the client, a string, a list or a struct
- `statusCode` (number): the HTTP status code to return, 200 by default
- `message` (string): a custom "reason" message to return

This is the method used by all other methods in this trait to return a response to the client.

The `data` element can be a string, a list or a struct. By default, a list or a struct will be turned into JSON,
unless [Content Negotiation](../incoming/content_negotiation.md) determines it should be returned in a different format.
A string is sent as it is when the format is not JSON (as HTML), see [Handling Response Types](#handling-response-types).

If a `message` string is passed, it will be used in place of the standard IANA reason codes for the
response status. Not every client will respect the custom codes, though, and will use the IANA standards
that match the status code.

If you leave out both the data and the status code, the status code is 404.

!!! note
    Since it sets the status code and body on the active Response instance, this should always
    be the final method in the script execution. Return its value from the action.

!!! note "Difference from CodeIgniter"
    The response has no `setJSON()` or `setXML()` in ColdIgniter, so the formatted text is set with `setBody()`.

### fail

`fail(messages [, status [, code [, message]]])`

- `messages` (string, list or struct): a string or a list of strings that contain error messages encountered
- `status` (number): the HTTP status code to return, 400 by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return
- Returns a multi-part response in the client's preferred format

This is the generic method used to represent a failed response, and is used by all of the other "fail" methods.

The `messages` element can be a string or a list of strings.

The `status` parameter is the HTTP status code that should be returned.

Since many APIs are better served using custom error codes, a custom error code can be passed in the third
parameter. If no value is present, it will be the same as `status`.

If a `message` string is passed, it will be used in place of the standard IANA reason codes for the
response status. Not every client will respect the custom codes, though, and will use the IANA standards
that match the status code.

The response has three elements: `status`, `error`, and `messages`.

- The `status` element contains the status code of the error.
- The `error` element contains a custom, API-specific error code, or the status code.
- The `messages` element contains the error messages. A single string is sent as `{"error": "..."}`.

Depending on the number of error messages, the response would look something like:

```json
{
    "status": 400,
    "error": 400,
    "messages": {
        "error": "Invalid data"
    }
}
```

### respondCreated

`respondCreated([data [, message]])`

- `data` (any): the data to return to the client
- `message` (string): a custom "reason" message to return
- Returns the response, to be returned from the action

Sets the appropriate status code to use when a new resource was created, typically 201:

```cfm
<cfset user = StructNew()>
<cfset user["id"] = 12>
<cfset user["name"] = "Example">

<cfreturn respondCreated(user)>
```

### respondDeleted

`respondDeleted([data [, message]])`

- `data` (any): the data to return to the client
- `message` (string): a custom "reason" message to return
- Returns the response, to be returned from the action

Sets the appropriate status code to use when a resource was deleted as the result of this API call, typically 200.

```cfm
<cfset result = StructNew()>
<cfset result["id"] = 12>

<cfreturn respondDeleted(result)>
```

### respondUpdated

`respondUpdated([data [, message]])`

- `data` (any): the data to return to the client
- `message` (string): a custom "reason" message to return
- Returns the response, to be returned from the action

Sets the appropriate status code to use when a resource was updated as the result of this API call, typically 200.

```cfm
<cfreturn respondUpdated(user)>
```

### respondNoContent

`respondNoContent([message])`

- `message` (string): a custom "reason" message to return, `No Content` by default
- Returns the response, to be returned from the action

Sets the appropriate status code to use when a command was successfully executed by the server but there is no
meaningful reply to send back to the client, typically 204.

```cfm
<cfreturn respondNoContent()>
```

### failUnauthorized

`failUnauthorized([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Unauthorized` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when the user either has not been authorized,
or has incorrect authorization. Status code is 401.

```cfm
<cfreturn failUnauthorized("Sign in first")>
```

### failForbidden

`failForbidden([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Forbidden` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Unlike `failUnauthorized()`, this method should be used when the requested API endpoint is never allowed.
Unauthorized implies the client is encouraged to try again with different credentials. Forbidden means
the client should not try again because it won't help. Status code is 403.

```cfm
<cfreturn failForbidden("This endpoint is closed")>
```

### failNotFound

`failNotFound([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Not Found` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when the requested resource cannot be found. Status code is 404.

```cfm
<cfreturn failNotFound("No customer with id " & arguments.id)>
```

### failValidationErrors

`failValidationErrors(errors [, code [, message]])`

- `errors` (string, list or struct): the error message or messages to show the user
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when data the client sent did not pass validation rules. Status code is typically 400.

```cfm
<cfreturn failValidationErrors(this.model.errors())>
```

### failResourceExists

`failResourceExists([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Conflict` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when the resource the client is trying to create already exists.
Status code is typically 409.

```cfm
<cfreturn failResourceExists("A customer with this name exists")>
```

### failResourceGone

`failResourceGone([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Gone` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when the requested resource was previously deleted and
is no longer available. Status code is typically 410.

```cfm
<cfreturn failResourceGone("This customer was deleted")>
```

### failTooManyRequests

`failTooManyRequests([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Too Many Requests` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when the client has called an API endpoint too many times.
This might be due to some form of throttling or rate limiting. Status code is typically 429.

```cfm
<cfreturn failTooManyRequests("Slow down")>
```

### failServerError

`failServerError([description [, code [, message]]])`

- `description` (string): the error message to show the user, `Internal Server Error` by default
- `code` (string): a custom, API-specific, error code
- `message` (string): a custom "reason" message to return

Sets the appropriate status code to use when there is a server error.

```cfm
<cfreturn failServerError("Something went wrong")>
```

## Pagination Responses

When returning paginated results from an API endpoint, you can use the `paginate()` method to return the
results along with the pagination information. This helps to keep consistent responses across your API, while
providing all of the information that clients will need to properly page through the results.

### Example Usage

```cfm
<cfcomponent output="no" parent="ColdIgniter\RESTful\ResourceController">

	<cffunction name="index" access="public" returntype="any" output="no">
		<cfset var model = new_("App\Models\UserModel")>

		<cfreturn paginate(model, 20)>
	</cffunction>

</cfcomponent>
```

A typical response might look like:

```json
{
    "data": [
        {
            "id": 1,
            "username": "admin",
            "email": "admin@example.com"
        },
        {
            "id": 2,
            "username": "user",
            "email": "user@example.com"
        }
    ],
    "meta": {
        "page": 1,
        "perPage": 20,
        "total": 2,
        "totalPages": 1
    },
    "links": {
        "self": "http://example.com/users?page=1",
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    }
}
```

The `paginate()` method will always wrap the results in a `data` element, and will also include `meta`
and `links` elements to help the client page through the results. The response also has a `Link` header and an
`X-Total-Count` header. The current page is read from the `page` query parameter.

!!! note "Difference from CodeIgniter"
    The link to the previous page is named `prev` here.

You can also pass it a Builder instance instead of a Model, as long as the Builder is properly configured with the table
name and any necessary joins or where clauses.

```cfm
<cfset builder = db_connect().table("users")>
<cfset builder.where("active", 1)>

<cfreturn paginate(builder, 20)>
```

### Class Reference

#### paginate

`paginate(resource [, perPage [, transformWith]])`

- `resource` (Model or Builder): the resource to paginate, either a Model or a Builder instance
- `perPage` (number): the number of items to return per page, 20 by default
- `transformWith` (string): optional transformer class name to transform the results

Generates a paginated response from the given resource. The resource can be either a Model or a Builder
instance. The method will automatically determine the current page from the request's query parameters.
The response will include the paginated data, along with metadata about the pagination state and links
to navigate through the pages.

If you provide a `transformWith` parameter with a transformer class name, each item in the paginated
results will be transformed using that transformer before being returned. This is useful for controlling
the structure and content of your API responses. See [API Resources](api_transformers.md) for more
information on creating and using transformers.

Example with transformer:

```cfm
<cfreturn paginate(model, 20, "App\Transformers\UserTransformer")>
```
