# Content Security Policy

## What is Content Security Policy?

One of the best protections you have against XSS attacks is a Content Security Policy (CSP) on the site. It requires you to name and approve each source of content in the HTML of your site, such as images, stylesheets and JavaScript files. The browser refuses content from sources that are not approved. The approval is written in the `Content-Security-Policy` header of the response, which has many options.

This can sound complex, and on some sites it is hard to do. For many simple sites, where all content comes from the same domain (for example **http://example.com**), it is easy.

This subject is large, so this guide does not cover all of it. For more information, visit these sites:

- [Content Security Policy main site](https://content-security-policy.com/)
- [W3C Specification](https://www.w3.org/TR/CSP)
- [Introduction at HTML5Rocks](https://www.html5rocks.com/en/tutorials/security/content-security-policy/)
- [Article at SitePoint](https://www.sitepoint.com/improving-web-security-with-the-content-security-policy/)

## Turning CSP On

!!! important
    The debug toolbar writes inline scripts and styles. When CSP is on, a CSP nonce is output for the toolbar by itself. If you do not use a CSP nonce yourself, this changes the CSP header to something you did not intend, and it behaves differently than in production. To check the CSP behavior, turn the debug toolbar off.

By default, CSP is off. To turn it on in your application, set the `CSPEnabled` value in **app/Config/App.cfc**:

```cfm
<cfset this.CSPEnabled = true>
```

When it is on, the response component holds an instance of `ColdIgniter\HTTP\ContentSecurityPolicy`. The values set in **app/Config/ContentSecurityPolicy.cfc** are applied to it. If you need no changes while the request runs, the correct header is sent and you are done.

With CSP on, two header lines can be added to the HTTP response. One is a **Content-Security-Policy** header, with the policies that name the content types or origins that are explicitly allowed in different contexts. The other is a **Content-Security-Policy-Report-Only** header, with the content types or origins that are allowed but also reported to the place of your choice. A header is sent only when it has at least one directive.

The default treatment can be changed with the `reportOnly()` function. When you add an entry to a CSP directive, as shown below, it goes to the header that blocks or prevents. You can change that for each call with the optional second argument of the add function.

## Runtime Configuration

If your application needs to make changes while the request runs, you can reach the instance with `this.response.getCSP()` in your controllers. The component has many functions that map clearly to the value of the header. The examples below show different combinations of arguments. Each add function takes a source or an array of sources.

```cfm
<!--- get the CSP instance --->
<cfset csp = this.response.getCSP()>

<!--- specify the default directive treatment --->
<cfset csp.reportOnly(false)>

<!--- specify the origin to use if none is given for a directive --->
<cfset csp.setDefaultSrc("cdn.example.com")>

<!--- specify the URL that "report-only" reports are sent to --->
<cfset csp.setReportURI("http://example.com/csp/reports")>

<!--- specify that HTTP requests are upgraded to HTTPS --->
<cfset csp.upgradeInsecureRequests(true)>

<!--- add types or origins to CSP directives; the default treatment is to block --->
<cfset csp.addBaseURI("example.com", true)><!--- report only --->
<cfset csp.addChildSrc("https://youtube.com")><!--- blocked --->
<cfset csp.addConnectSrc("https://*.facebook.com", false)><!--- blocked --->
<cfset csp.addFontSrc("fonts.example.com")>
<cfset csp.addFormAction("self")>
<cfset csp.addFrameAncestor("none", true)><!--- report this one --->
<cfset csp.addImageSrc("cdn.example.com")>
<cfset csp.addMediaSrc("cdn.example.com")>
<cfset csp.addManifestSrc("cdn.example.com")>
<cfset csp.addObjectSrc("cdn.example.com", false)><!--- reject from here --->
<cfset csp.addPluginType("application/pdf", false)><!--- reject this media type --->
<cfset csp.addScriptSrc("scripts.example.com", true)><!--- allow but report requests from here --->
<cfset csp.addStyleSrc("css.example.com")>

<cfset sandbox = ArrayNew(1)>
<cfset ArrayAppend(sandbox, "allow-forms")>
<cfset ArrayAppend(sandbox, "allow-scripts")>
<cfset csp.addSandbox(sandbox)>

<!--- the CSP3 directives --->
<cfset csp.addScriptSrcAttr("trusted.com")>
<cfset csp.addScriptSrcElem("trusted.com")>
<cfset csp.addStyleSrcAttr("trusted.com")>
<cfset csp.addStyleSrcElem("trusted.com")>
<cfset csp.addWorkerSrc("workers.example.com")>
```

The first argument of each "add" function is a source (a string), or an array of them.

!!! note "Difference from CodeIgniter"
    The response has no `$csp` property of its own. `this.response.getCSP()` makes the instance the first time you call it.

### Report Only

The `reportOnly()` function sets the default reporting treatment for the sources that you add after it, unless you override it in a call. For example, you can allow youtube.com, and then add several sources that are allowed but reported:

```cfm
<!--- get the CSP instance --->
<cfset csp = this.response.getCSP()>

<cfset csp.addChildSrc("https://youtube.com")><!--- allowed --->

<cfset csp.reportOnly(true)>
<cfset csp.addChildSrc("https://metube.com")><!--- allowed but reported --->
<cfset csp.addChildSrc("https://ourtube.com", false)><!--- allowed --->
```

### Reporting Directives

To set the URL that the reports are sent to, use the `setReportURI()` function.

CSP Level 3 replaces the `report-uri` directive with `report-to`. You can use the `setReportToEndpoint()` function to set the reporting endpoint for CSP reports. Before you use it, define the reporting endpoints with the `addReportingEndpoints()` function. It takes a struct of the name of the endpoint to its URL. If the endpoint is not defined, `setReportToEndpoint()` throws an `InvalidArgumentException`.

```cfm
<!--- get the CSP instance --->
<cfset csp = this.response.getCSP()>

<cfset csp.setReportURI("https://example.com/csp-reports")>

<!--- or use a reporting endpoint --->
<cfset endpoints = StructNew()>
<cfset endpoints["default"] = "https://example.com/csp-reports">
<cfset endpoints["reports"] = "https://example.com/other-csp-reports">

<cfset csp.addReportingEndpoints(endpoints)>
<cfset csp.setReportToEndpoint("default")>
```

For browsers that do not know the `report-to` directive, ColdIgniter also sets the `report-uri` directive when you use `setReportToEndpoint()`.

### Clear Directives

To clear the directives that are already set, use the `clearDirective()` function:

```cfm
<!--- get the CSP instance --->
<cfset csp = this.response.getCSP()>

<cfset csp.clearDirective("style-src")>
```

## Inline Content

A site can be set to not protect even the inline scripts and styles of its own pages, because they can be the result of content that users wrote. To protect against this, CSP lets you give a nonce in the `<style>` and `<script>` tags, and add the same values to the header of the response.

### Using Placeholders

This is hard to do by hand, and it is most secure when the nonce is made on the fly. To make it simple, put a `{csp-style-nonce}` or `{csp-script-nonce}` placeholder in the tag, and it is replaced for you:

```html
<!-- Original -->
<script {csp-script-nonce}>
    console.log("Script won't run as it doesn't contain a nonce attribute");
</script>

<!-- Becomes -->
<script nonce="Eskdikejidojdk978Ad8jf">
    console.log("Script won't run as it doesn't contain a nonce attribute");
</script>

<!-- OR -->
<style {csp-style-nonce}>
    . . .
</style>
```

!!! warning
    If an attacker injects a string like `<script {csp-script-nonce}>`, it can become the real nonce attribute with this function. You can change the placeholder with the `scriptNonceTag` and `styleNonceTag` settings in **app/Config/ContentSecurityPolicy.cfc**.

### Using Functions

If you do not want the automatic replacement, turn it off by setting `this.autoNonce = false` in **app/Config/ContentSecurityPolicy.cfc**. Then you can use the functions `csp_script_nonce()` and `csp_style_nonce()`, which return the `nonce="..."` attribute. They return an empty string when CSP is off.

```cfm
<!-- Original -->
<script <cfoutput>#csp_script_nonce()#</cfoutput>>
    console.log("Script won't run as it doesn't contain a nonce attribute");
</script>

<!-- Becomes -->
<script nonce="Eskdikejidojdk978Ad8jf">
    console.log("Script won't run as it doesn't contain a nonce attribute");
</script>

<!-- OR -->
<style <cfoutput>#csp_style_nonce()#</cfoutput>>
    . . .
</style>
```
