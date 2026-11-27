# Honeypot Class

The Honeypot Class makes it possible to determine when a Bot makes a request to a ColdIgniter application, if it's enabled in **app/Config/Filters.cfc** file. This is done by attaching form fields to any form, and this form field is hidden from a human but accessible to a Bot. When data is entered into the field, it's assumed the request is coming from a Bot. The bundled Honeypot filter throws a `HoneypotException` with an HTTP status code of 403 (Forbidden).

## Enabling Honeypot

To enable a Honeypot, changes have to be made to the **app/Config/Filters.cfc**. Just add `honeypot` to the `globals` lists, in `before` and in `after`, like:

```cfm
<cfset this.globals = CreateObject("java", "java.util.LinkedHashMap").init()>
<cfset this.globals["before"] = ArrayNew(1)>
<cfset ArrayAppend(this.globals["before"], "honeypot")>
<cfset this.globals["after"] = ArrayNew(1)>
<cfset ArrayAppend(this.globals["after"], "honeypot")>
```

The file has both lists, with a comment that names the filters that can go in. See [Filters](../incoming/filters.md) for the `globals` section.

A sample Honeypot filter is bundled, as **system/Filters/Honeypot.cfc**, with the alias `honeypot`. If it is not suitable, make your own at **app/Filters/Honeypot.cfc**, and modify the `aliases` in **app/Config/Filters.cfc** appropriately.

The `before` filter checks the request: if the honeypot field has content, it throws the exception. The `after` filter adds the field to every `</form>` tag in the body of the response.

!!! note "Difference from CodeIgniter"
    A posted value is always text, so only an empty string, `"0"` and an empty array count as "no content". Any other value in the field means a bot.

## Customizing Honeypot

Honeypot can be customized. The fields below can be set either in **app/Config/Honeypot.cfc** or in **.env**.

- `hidden`: `true` or `false` to control visibility of the honeypot field. The default is `true`.
- `label`: HTML label for the honeypot field. The default is `Fill This Field`.
- `name`: name of the HTML form field used for the template. The default is `honeypot`.
- `template`: form field template used for the honeypot. The default is `<label>{label}</label><input type="text" name="{name}" value="">`.
- `container`: container tag for the template. The default is `<div style="display:none">{template}</div>`. If you enable CSP, you can remove `style="display:none"`.
- `containerId`: this setting is used only when you enable CSP. You can change the id attribute for the container tag. The default is `hpc`.

When CSP is enabled (`CSPEnabled` in **app/Config/App.cfc** and the policy of the response), the container gets the `id` attribute, and a `<style>` tag with the nonce of the policy is added before `</head>` to hide the container.
