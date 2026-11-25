# API Resources

When you build an API, you often need to turn your data into a steady format before you send it to the client. API Resources, made with transformers, convert your rows, structs or components into structured API responses. They keep the inside shape of your data apart from what the API shows, so the API is easier to maintain and change over time.

## Quick Example

The example below shows the usual way to use a transformer in your application.

```cfm
<!--- app/Transformers/UserTransformer.cfc --->
<cfcomponent output="no" parent="ColdIgniter\API\BaseTransformer">

	<cffunction name="toArray" access="public" returntype="any" output="no">
		<cfargument name="resource" type="any" required="yes">
		<cfset var data = StructNew()>

		<cfset data["id"] = arguments.resource["id"]>
		<cfset data["name"] = arguments.resource["name"]>
		<cfset data["email"] = arguments.resource["email"]>
		<cfset data["created_at"] = arguments.resource["created_at"]>

		<cfreturn data>
	</cffunction>

</cfcomponent>
```

```cfm
<!--- in a controller action --->
<cfset var user = this.model.find_(1)>
<cfset var transformer = new_("App\Transformers\UserTransformer")>

<cfreturn respond(transformer.transform(user))>
```

The `UserTransformer` defines which fields of a user go into the API response. The `transform()` function converts one resource. The `transformMany()` function converts an array of resources.

!!! note "Difference from CodeIgniter"
    A transformer is a component made with `new_()`, not with `new`. The "array" that `toArray()` returns is a struct, and a list of resources is a CFML array. Write the struct keys with brackets, as in `data["id"]`, so that their case is kept.

## Creating a Transformer

To create a transformer, make a component that has `ColdIgniter\API\BaseTransformer` as its parent, and write the `toArray()` function that defines the structure of your API resource. The function receives the resource that is transformed, so you can read and change its data.

### Basic Transformer

```cfm
<cfcomponent output="no" parent="ColdIgniter\API\BaseTransformer">

	<cffunction name="toArray" access="public" returntype="any" output="no">
		<cfargument name="resource" type="any" required="yes">
		<cfset var data = StructNew()>

		<cfset data["id"] = arguments.resource["id"]>
		<cfset data["name"] = arguments.resource["name"]>
		<cfset data["email"] = arguments.resource["email"]>

		<cfreturn data>
	</cffunction>

</cfcomponent>
```

The `toArray()` function receives the resource (a struct or a component) and defines the structure of your API response. You can put any field of the resource in it, and you can rename or change the values.

### Generating Transformer Files

!!! warning "Not ported yet"
    The `make:transformer` command does not exist, because there is no command line. Create the file by hand in `app/Transformers/`.

#### Command Options

!!! warning "Not ported yet"
    The options `--suffix`, `--namespace` and `--force` do not exist, because the command does not exist.

#### Subdirectories

!!! warning "Not ported yet"
    Making a transformer in a sub-folder with the command does not exist. Make the folder by hand in `app/Transformers/`, and use its name in the component path, as in `new_("App\Transformers\Api\V1\User")`.

### Using Transformers in Controllers

When the transformer exists, use it in a controller to change the data before you return it to the client.

```cfm
<cfcomponent output="no" parent="ColdIgniter\RESTful\ResourceController">
	<cfset this.modelName = "App\Models\UserModel">

	<cffunction name="show" access="public" returntype="any" output="no">
		<cfargument name="id" type="any" required="no" default="">
		<cfset var user = this.model.find_(arguments.id)>
		<cfset var transformer = "">

		<cfif IsSimpleValue(user)>
			<cfreturn failNotFound("User not found")>
		</cfif>

		<cfset transformer = new_("App\Transformers\UserTransformer")>

		<cfreturn respond(transformer.transform(user))>
	</cffunction>

	<cffunction name="index" access="public" returntype="any" output="no">
		<cfset var transformer = new_("App\Transformers\UserTransformer")>

		<cfreturn respond(transformer.transformMany(this.model.findAll()))>
	</cffunction>

</cfcomponent>
```

The response functions such as `respond()` and `failNotFound()` are described in [API Responses](api_responses.md).

## Field Filtering

The transformer supports field filtering by itself, through the `fields` query variable of the current URL. A client can ask for only the fields it needs, which sends less data. You write no extra code for it.

A request to `/users/1?fields=id,name` returns only:

```json
{
    "id": 1,
    "name": "John Doe"
}
```

!!! note "Difference from CodeIgniter"
    A CodeIgniter request to `?fields=` is read as a list with one empty name, and it returns no field. In ColdIgniter an empty `fields` counts as not given, and all fields are returned.

### Restricting Available Fields

By default, a client can ask for any field that your `toArray()` function defines. To restrict the fields, write a `getAllowedFields()` function in your transformer. It returns an array of names.

```cfm
<cfcomponent output="no" parent="ColdIgniter\API\BaseTransformer">

	<cffunction name="toArray" access="public" returntype="any" output="no">
		<cfargument name="resource" type="any" required="yes">
		<cfset var data = StructNew()>

		<cfset data["id"] = arguments.resource["id"]>
		<cfset data["name"] = arguments.resource["name"]>
		<cfset data["email"] = arguments.resource["email"]>

		<cfreturn data>
	</cffunction>

	<cffunction name="getAllowedFields" access="private" returntype="any" output="no">
		<cfset var allowed = ArrayNew(1)>

		<cfset ArrayAppend(allowed, "id")>
		<cfset ArrayAppend(allowed, "name")>

		<cfreturn allowed>
	</cffunction>

</cfcomponent>
```

Now, if a client asks for `/users/1?fields=email`, an `ApiException` is thrown, because `email` is not in the allowed fields.

## Including Related Resources

A transformer can load related resources through the `include` query variable. Clients use it to say which relations they want in the answer. Relations are the most common use, but you can include any extra data when you write your own include functions.

### Defining Include Methods

To support related resources, write functions whose name is `include` followed by the name of the resource. Inside them, the resource that is transformed is in `this.resource`.

```cfm
<cfcomponent output="no" parent="ColdIgniter\API\BaseTransformer">

	<cffunction name="toArray" access="public" returntype="any" output="no">
		<cfargument name="resource" type="any" required="yes">
		<cfset var data = StructNew()>

		<cfset data["id"] = arguments.resource["id"]>
		<cfset data["name"] = arguments.resource["name"]>
		<cfset data["email"] = arguments.resource["email"]>

		<cfreturn data>
	</cffunction>

	<cffunction name="includePosts" access="private" returntype="any" output="no">
		<cfset var posts = new_("App\Models\PostModel").where("user_id", this.resource["id"]).findAll()>

		<cfreturn new_("App\Transformers\PostTransformer").transformMany(posts)>
	</cffunction>

	<cffunction name="includeComments" access="private" returntype="any" output="no">
		<cfset var comments = new_("App\Models\CommentModel").where("user_id", this.resource["id"]).findAll()>

		<cfreturn new_("App\Transformers\CommentTransformer").transformMany(comments)>
	</cffunction>

</cfcomponent>
```

The include functions use `this.resource["id"]` to read the id of the user that is transformed. The transformer sets `this.resource` when you call `transform()`.

Clients can now ask for `/users/1?include=posts,comments`. The response contains:

```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "posts": [
        {
            "id": 1,
            "title": "First Post"
        }
    ],
    "comments": [
        {
            "id": 1,
            "content": "Great article!"
        }
    ]
}
```

!!! note
    The `fields` and `include` query variables describe the **root** resource only. A transformer that is made inside an `include*()` function (such as `PostTransformer` above) is a nested resource. When it is made **without an explicit request**, it does **not** take over the `fields` and `include` of the root request. This stops the query variables of the parent from leaking into the related resources, where they could filter the wrong fields or start includes that loop. If you pass a request to a nested transformer on purpose, the scope of that request is used.

!!! note "Difference from CodeIgniter"
    The include functions are found by name, and the requested name must be made of letters, digits and underscores. The functions can be `private`. An empty `include` counts as not given.

### Restricting Available Includes

As with the fields, you can restrict which relations a client can include. Write a `getAllowedIncludes()` function.

```cfm
<cffunction name="getAllowedIncludes" access="private" returntype="any" output="no">
	<cfset var allowed = ArrayNew(1)>

	<cfset ArrayAppend(allowed, "posts")>

	<cfreturn allowed>
</cffunction>
```

To switch off all includes, return an empty array:

```cfm
<cffunction name="getAllowedIncludes" access="private" returntype="any" output="no">

	<cfreturn ArrayNew(1)>
</cffunction>
```

### Include Validation

The transformer checks that each include that a client asks for has an `include*()` function in your transformer. If it has not, an `ApiException` is thrown.

For example, if a client asks for:

```text
GET /api/users?include=invalid
```

and your transformer has no `includeInvalid()` function, an exception is thrown with the message "Missing include method for: invalid". This helps you find typing errors and stops unexpected behavior.

## Transforming Collections

The `transformMany()` function transforms an array of resources:

```cfm
<cfset var users = this.model.findAll()>
<cfset var transformer = new_("App\Transformers\UserTransformer")>

<cfreturn respond(transformer.transformMany(users))>
```

It applies the same logic to each item, with the field filtering and the includes that the request asks for.

!!! note "Difference from CodeIgniter"
    `transformMany()` takes a CFML array only.

## Working with Different Data Types

Transformers can handle more than rows.

### Transforming Entities

When you pass an Entity to `transform()`, the transformer calls the `toArray()` function of the entity and passes the result to your `toArray()` function:

```cfm
<cfset var data = StructNew()>
<cfset var user = new_("App\Entities\User")>
<cfset var transformer = new_("App\Transformers\UserTransformer")>
<cfset var result = "">

<cfset data["id"] = 1>
<cfset data["name"] = "John Doe">
<cfset data["email"] = "john@example.com">

<cfset user.fill(data)>
<cfset result = transformer.transform(user)>
```

### Transforming Arrays

You can also transform plain structs:

```cfm
<cfset var data = StructNew()>
<cfset var transformer = new_("App\Transformers\UserTransformer")>
<cfset var result = "">

<cfset data["id"] = 1>
<cfset data["name"] = "John Doe">
<cfset data["email"] = "john@example.com">

<cfset result = transformer.transform(data)>
```

### Transforming Objects

!!! warning "Not ported yet"
    Casting any object to an array is not ported. A component that has a `toArray()` function is transformed as described under Transforming Entities, and any other value goes to your `toArray()` function as it is.

### Using toArray() Only

If you do not pass a resource to `transform()`, it uses the data from your own `toArray()` function:

```cfm
<cfcomponent output="no" parent="ColdIgniter\API\BaseTransformer">

	<cffunction name="toArray" access="public" returntype="any" output="no">
		<cfargument name="resource" type="any" required="no" default="">
		<cfset var data = StructNew()>

		<cfset data["version"] = "1.0">
		<cfset data["status"] = "active">
		<cfset data["message"] = "API is running">

		<cfreturn data>
	</cffunction>

</cfcomponent>
```

```cfm
<cfset transformer = new_("App\Transformers\StaticDataTransformer")>
<cfset result = transformer.transform()>
```

## Class Reference

### BaseTransformer

The component is `ColdIgniter\API\BaseTransformer`.

**`__construct([request_])`**

- `request_`: an optional request. If you leave it out, the shared request is used.

Sets up the transformer and reads the `fields` and `include` query variables of the request.

!!! note
    When no request is passed, the `fields` and `include` of the shared request are read **only for the root transformer**. A transformer that is made while a nested transformation runs (inside an `include*()` function), without an explicit request, uses the shared request but does not read `fields` and `include` from it. This stops the scope of the root from leaking. A request that you pass is always used, whatever the nesting.

**`toArray(resource)`**

- `resource`: the resource that is transformed (a struct, a component, or an empty string when there is none)
- Returns: a struct

You write this function in your transformer to define the structure of the API resource. Return a struct with the fields that you want in the response, and read the data from the `resource` argument.

```cfm
<cffunction name="toArray" access="public" returntype="any" output="no">
	<cfargument name="resource" type="any" required="yes">
	<cfset var data = StructNew()>

	<cfset data["id"] = arguments.resource["id"]>
	<cfset data["name"] = arguments.resource["name"]>

	<cfreturn data>
</cffunction>
```

**`transform([resource])`**

- `resource`: the resource to transform (a struct, a component, or nothing)
- Returns: the transformed struct

Transforms the resource by calling `toArray()` with its data. If there is no resource, an empty string is passed to `toArray()`. If the resource is a component with a `toArray()` function, such as an Entity, its result is used. The resource is also stored in `this.resource`, so that the include functions can read it. The function applies the field filtering and the includes of the query variables.

```cfm
<cfset result = transformer.transform(user)>
```

**`transformMany(resources)`**

- `resources`: an array of resources
- Returns: an array of transformed structs

Transforms a list of resources by calling `transform()` on each item. The field filtering and the includes apply the same way to all items.

```cfm
<cfset result = transformer.transformMany(users)>
```

**`getAllowedFields()`**

- Returns: an array of allowed field names, or an empty string to allow all fields

Write this function in your transformer to restrict the fields that the `fields` query variable can ask for. Return an empty string (the default) to allow all fields of `toArray()`. Return an array of names to make a list of the allowed fields. The function is `private`.

**`getAllowedIncludes()`**

- Returns: an array of allowed include names, or an empty string to allow all includes

Write this function in your transformer to restrict the related resources that the `include` query variable can ask for. Return an empty string (the default) to allow all includes that have a function. Return an array of names to make a list of the allowed includes. Return an empty array to switch off all includes. The function is `private`.

## Exception Reference

### ApiException

The component is `ColdIgniter\API\ApiException`. It does not make an exception object. Each function throws the exception itself, with the type `ColdIgniter.API.ApiException`, and you catch it with `<cfcatch type="ColdIgniter.API.ApiException">`.

!!! note "Difference from CodeIgniter"
    In CodeIgniter the exception extends `FrameworkException`, so a catch of that class finds it. Here the type is its own.

**`forInvalidFields(field)`**

- `field`: the name or names of the invalid fields

Thrown when a client asks, with the `fields` query variable, for a field that is not in the allowed fields.

**`forInvalidIncludes(include)`**

- `include`: the name or names of the invalid includes

Thrown when a client asks, with the `include` query variable, for an include that is not in the allowed includes.

**`forMissingInclude(include)`**

- `include`: the name of the missing include function

Thrown when a client asks, with the `include` query variable, for an include whose `include*()` function does not exist in the transformer. This check makes sure that each requested include has a function that handles it.
