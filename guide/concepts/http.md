# Working with HTTP Requests

In order to get the most out of ColdIgniter, you need to have a basic understanding of how HTTP requests and responses work. Since this is what you work with while developing web applications, understanding the concepts behind HTTP is a **must** for all developers that want to be successful.

The first part of this chapter gives an overview. After the concepts are out of the way, we will discuss how to work with the requests and responses within ColdIgniter.

## What is HTTP?

HTTP is simply a text-based convention that allows two machines to talk to each other. When a browser requests a page, it asks the server if it can get the page. The server then prepares the page and sends a response back to the browser that asked for it. That's pretty much it. Obviously, there are some complexities that you can use, but the basics are really pretty simple.

HTTP is the term used to describe that exchange convention. It stands for HyperText Transfer Protocol. Your goal when you develop web applications is to always understand what the browser is requesting, and be able to respond appropriately.

### The Request

Whenever a client (a web browser, smartphone app, etc) makes a request, it sends a small text message to the server and waits for a response.

The request would look something like this:

```text
GET / HTTP/1.1
Host: example.com
Accept: text/html
User-Agent: Chrome/46.0.2490.80
```

This message displays all of the information necessary to know what the client is requesting. It tells the method for the request (GET, POST, DELETE, etc), and the version of HTTP it supports.

The request also includes a number of optional request headers that can contain a wide variety of information such as what languages the client wants the content displayed as, the types of formats the client accepts, and much more. Wikipedia has an article that lists [all header fields](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields) if you want to look it over.

### The Response

Once the server receives the request, your application will take that information and generate some output. The server will bundle your output as part of its response to the client. This is also represented as a simple text message that looks something like this:

```text
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Thu, 05 Nov 2015 05:33:22 GMT
Content-Type: text/html; charset=UTF-8

<html>
    . . .
</html>
```

The response tells the client what version of the HTTP specification that it's using and, probably most importantly, the status code (200). The status code is one of a number of codes that have been standardized to have a very specific meaning to the client. This can tell them that it was successful (200), or that the page wasn't found (404). Head over to IANA for a [full list of HTTP status codes](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml).

## Working with Requests and Responses

ColdFusion MX 6.1 gives you the `url`, `form` and `cgi` scopes to read a request. ColdIgniter, like most frameworks, abstracts them so that you have a consistent, simple interface to them. The [IncomingRequest class](../incoming/incomingrequest.md) is an object-oriented representation of the HTTP request. It provides everything you need:

```cfm
<cfset request_ = request_()>

<!--- the URI path being requested (i.e., about) --->
<cfset path = request_.getPath()>

<!--- Retrieve url and form variables --->
<cfset foo = request_.getGet("foo")>
<cfset foo = request_.getPost("foo")>

<!--- Retrieve from both the url and form variables --->
<cfset foo = request_.getVar("foo")>

<!--- Retrieve JSON from AJAX calls --->
<cfset json = request_.getJSON()>

<!--- Retrieve server variables --->
<cfset host = request_.getServer("SERVER_NAME")>

<!--- Retrieve an HTTP Request header, with case-insensitive names --->
<cfset header = request_.header("host")>
<cfset header = request_.header("Content-Type")>

<!--- Checks the HTTP method --->
<cfset isGet = request_.is("get")>
<cfset isPost = request_.is("post")>
```

!!! note "Difference from CodeIgniter"
    The global function is `request_()`, with an underscore, because `request` is the name of a scope in CFML. There is no `getUri()` on the incoming request: use `getPath()` for the path.

The request class does a lot of work in the background for you, that you never need to worry about. The `isAJAX()` and `isSecure()` functions check several different ways to determine the correct answer.

!!! note
    The `isAJAX()` function depends on the `X-Requested-With` header, which in some cases is not sent by default in XHR requests via JavaScript (i.e., fetch). See the [AJAX Requests](../general/ajax.md) section on how to avoid this problem.

ColdIgniter also provides a [Response class](../outgoing/response.md) that is an object-oriented representation of the HTTP response. This gives you an easy and powerful way to construct your response to the client:

```cfm
<cfset response = service("response")>

<cfset response.setStatusCode(200)>
<cfset response.setBody(output)>
<cfset response.setHeader("Content-Type", "text/html")>
<cfset response.noCache()>

<!--- Sends the output to the browser --->
<!--- This is typically handled by the framework --->
<cfset response.send()>
```

!!! note "Difference from CodeIgniter"
    There is no `response()` global function and no `Response::HTTP_OK` constant. Get the response with `service("response")` and give the status code as a number.

In addition, the Response class allows you to work with the HTTP cache layer for the best performance.
