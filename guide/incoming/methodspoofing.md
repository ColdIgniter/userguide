# HTTP Method Spoofing

When you work with HTML forms you can only use the GET or POST HTTP verbs. In most cases this is fine. To support RESTful routing you need other verbs, like DELETE or PUT. Browsers do not support them, so ColdIgniter lets you spoof the method. You make a POST request, but tell the application to treat it as a different request type.

To spoof the method, add a hidden input named `_method` to the form. Its value is the HTTP verb you want the request to be:

```cfm
<form action="" method="post">
    <input type="hidden" name="_method" value="PUT">
</form>
```

This form is turned into a PUT request. As far as the routing and the [IncomingRequest](incomingrequest.md) class are concerned, it is a true PUT request.

The form must use the POST method. GET requests cannot be spoofed. Only the values `PUT`, `PATCH` and `DELETE` are accepted, and they must be in upper case. Any other value is ignored and the request stays a POST.

!!! note
    Check the configuration of your web server. Some servers do not support all HTTP verbs with the default configuration and need extra setup.
