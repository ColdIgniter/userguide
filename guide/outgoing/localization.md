# Localization

## Working with Locales

ColdIgniter has tools to help you localize your application for different languages. A complete localization of an application is a large subject, but it is simple to swap the strings of your application for those of another language.

### Configuring the Locale

#### Setting the Default Locale

Every site has a default language (locale) that it works in. Set it in **app/Config/App.cfc**:

```cfm
<cfset this.defaultLocale = "en">
```

The value can be any string that your application uses to manage text strings and other formats. A [BCP 47](http://www.rfc-editor.org/rfc/bcp/bcp47.txt) language code is recommended. This gives codes like en-US for American English, or fr-FR for French in France. A readable introduction is on the [site of the W3C](https://www.w3.org/International/articles/language-tags/).

If an exact match cannot be found, the system falls back to a more general language code. If the locale is `en-US` and you only have language files for `en`, those are used, because nothing exists for the more specific `en-US`. If a folder **app/Language/en-US** exists, it is used first.

### Locale Detection

There are two ways to detect the correct locale during the request.

1. [Content Negotiation](#content-negotiation): a "set and forget" way that negotiates the correct locale for you.
2. [In Routes](#in-routes): a way that lets you put a segment in your routes that sets the locale.

If you need to set the locale yourself, see [Setting the Current Locale](#setting-the-current-locale).

The `setValidLocales()` function of the request sets (and resets) the valid locales, which come from the `supportedLocales` setting of **app/Config/App.cfc**.

#### Content Negotiation

You can make the content negotiation happen by itself with two settings in **app/Config/App.cfc**. The first tells the request that you want to negotiate a locale. Set it to true:

```cfm
<cfset this.negotiateLocale = true>
```

When this is on, the system negotiates the correct language from the array of locales that you define in `supportedLocales`. If no match is found between the languages that you support and the language that was asked for, the first item of `supportedLocales` is used. In the example below, `en` is used if there is no match:

```cfm
<cfset this.supportedLocales = ArrayNew(1)>
<cfset ArrayAppend(this.supportedLocales, "en")>
<cfset ArrayAppend(this.supportedLocales, "es")>
<cfset ArrayAppend(this.supportedLocales, "fr-FR")>
```

#### In Routes

!!! warning "Not ported yet"
    The `{locale}` placeholder in a route and the `useSupportedLocalesOnly()` function of the route collection are not ported. Use content negotiation, or set the locale yourself as shown below.

### Setting the Current Locale

#### IncomingRequest Locale

To set the locale yourself, use the `setLocale_()` function of the request:

```cfm
<cfset this.request_.setLocale_("fr")>
```

Before you set the locale, the valid locales must be set. If you try to set a locale that is not valid, the default locale is set instead. By default, the valid locales are the ones in the `supportedLocales` setting of **app/Config/App.cfc**:

```cfm
<cfset this.supportedLocales = ArrayNew(1)>
<cfset ArrayAppend(this.supportedLocales, "en")>
<cfset ArrayAppend(this.supportedLocales, "es")>
<cfset ArrayAppend(this.supportedLocales, "fr-FR")>
```

The `setValidLocales()` function of the request sets (and resets) the valid locales. Use it to change the valid locales while the request runs.

!!! note "Difference from CodeIgniter"
    `setLocale()` and `getLocale()` are CFML built-in functions, so the components use `setLocale_()` and `getLocale_()`. The function sets the locale of the request only, and does not change the locale of the server.

#### Language Locale

The Language component that the `lang()` function uses also has a current locale. It is set to the locale of the request when the component is made. To change the locale after that, use the `setLocale_()` function of the Language component:

```cfm
<cfset service("language").setLocale_("fr")>
```

### Retrieving the Current Locale

You can always read the current locale from the request with the `getLocale_()` function. In a controller that has `ColdIgniter\Controller` as its parent, the request is `this.request_`:

```cfm
<cfset locale = this.request_.getLocale_()>
```

You can also get the current request from the services:

```cfm
<cfset locale = service("request").getLocale_()>
```

## Language Localization

### Creating Language Files

Language strings are stored in the **app/Language** folder, with a sub-folder for each supported language (locale):

```text
app/
    Language/
        en/
            App.cfm
        fr/
            App.cfm
```

!!! note
    Language files do not have namespaces.

There is no required naming for language files. Name the file in a way that describes what it holds. For example, a file with error messages can be called **Errors.cfm**.

In the file you set the variable `lines` to a struct. Each key of the struct is a language key, and its value is the string to return:

```cfm
<!--- app/Language/en/Errors.cfm --->
<cfset lines = StructNew()>

<cfset lines["errorEmailMissing"] = "You must submit an email address">
<cfset lines["errorURLMissing"] = "You must submit a URL">
<cfset lines["errorUsernameMissing"] = "You must submit a username">
<cfset lines["nonErrorString"] = "Here is a string that has no error">
```

!!! note "Difference from CodeIgniter"
    A language file is a **.cfm** file that sets `lines`. It does not return an array. Write the keys with brackets, so that their case is kept.

Nested definitions are also supported:

```cfm
<!--- app/Language/en/Errors.cfm --->
<cfset lines = StructNew()>
<cfset lines["nested"] = StructNew()>
<cfset lines["nested"]["error"] = StructNew()>

<cfset lines["nested"]["error"]["message"] = "A specific error message">
```

### Basic Usage

You can use the `lang()` function to get text from any language file. Pass the name of the file and the language key, separated by a period (`.`), as the first argument.

For example, to load the `errorEmailMissing` string from the **Errors.cfm** language file:

```cfm
<cfoutput>#lang("Errors.errorEmailMissing")#</cfoutput>
```

For a nested definition:

```cfm
<cfoutput>#lang("Errors.nested.error.message")#</cfoutput>
```

If the language key does not exist in the file for the current locale (after [Language Fallback](#language-fallback)), the string is passed back unchanged. In this example it would return `Errors.errorEmailMissing` or `Errors.nested.error.message`.

#### Replacing Parameters

You can pass values as the second argument of `lang()` to replace placeholders in the language string. The values are a struct for named placeholders, or an array for numbered placeholders.

```cfm
<!--- app/Language/en/Errors.cfm --->
<cfset lines["itemsCount"] = "There are {0} items in the list">
<cfset lines["welcome"] = "Welcome back, {name}">
```

A numbered placeholder is the position of the value in the array, counted from zero:

```cfm
<cfset values = ArrayNew(1)>
<cfset ArrayAppend(values, 4)>

<cfoutput>#lang("Errors.itemsCount", values)#</cfoutput>
<!--- There are 4 items in the list --->
```

You can also use names to keep things clear:

```cfm
<cfset values = StructNew()>
<cfset values["name"] = "Alan">

<cfoutput>#lang("Errors.welcome", values)#</cfoutput>
<!--- Welcome back, Alan --->
```

A placeholder that has no matching value stays as it is written.

!!! warning "Not ported yet"
    The `MessageFormatter` of the intl extension is not ported. Only `{name}` and `{0}` are replaced. The formats such as `{0, number, integer}`, `{0, number, currency}`, `{0, number, percent}`, dates, times, `spellout`, `ordinal`, `duration` and `select` are not known and stay as they are written. For plurals, only the English form `{n, plural, =1{# year} other{# years}}` is known. It uses the English rule (one for 1, else other), and a `#` is the number.

#### Specifying Locale

To use a different locale when you replace parameters, give the locale as the third argument of the `lang()` function. Pass an empty string as the second argument if there are no values.

```cfm
<cfset values = ArrayNew(1)>
<cfset ArrayAppend(values, 4)>

<cfoutput>#lang("Errors.itemsCount", values, "fr")#</cfoutput>
```

If you want to change the current locale, see [Language Locale](#language-locale).

#### Nested Arrays

Language files also allow nested arrays, to make lists easier to work with:

```cfm
<!--- app/Language/en/Fruit.cfm --->
<cfset lines = StructNew()>
<cfset lines["list"] = ArrayNew(1)>

<cfset ArrayAppend(lines["list"], "Apples")>
<cfset ArrayAppend(lines["list"], "Bananas")>
<cfset ArrayAppend(lines["list"], "Grapes")>
<cfset ArrayAppend(lines["list"], "Lemons")>
<cfset ArrayAppend(lines["list"], "Oranges")>
<cfset ArrayAppend(lines["list"], "Strawberries")>
```

```cfm
<cfset fruit = lang("Fruit.list")>
```

### Language Fallback

If you have a set of messages for a locale, for instance **Language/en/App.cfm**, you can add variants of that locale, each in its own folder, for instance **Language/en-US/App.cfm**. You only need the messages that are different in that variant. A message that is missing is taken from the main locale.

The fallback goes all the way to English (**en**), in case new messages are added to the framework and you have not yet translated them for your locale.

So, if you use the locale `fr-CA`, a message is first looked for in the **Language/fr-CA** folder, then in the **Language/fr** folder, and last in the **Language/en** folder.

### System Message Translations

!!! warning "Not ported yet"
    The "official" set of system message translations of CodeIgniter (the `codeigniter4/translations` package) is not ported. ColdIgniter has the system messages in English only, in **system/Language/en**. To add a language, create the files for that locale in **app/Language** yourself.

### Overriding System Message Translations

The framework has its system messages in **system/Language/en**. If you want to override some of them, create language files with the same names in the **app/Language/en** folder, and set only the lines that you want to change. The files of **system** and of **app** are merged, and the lines of **app** win.

### Generating Translation Files via Command

!!! warning "Not ported yet"
    The `lang:find` command does not exist, because there is no command line. Write the language files by hand.

#### Synchronization Translation Files via Command

!!! warning "Not ported yet"
    The `lang:sync` command does not exist, because there is no command line. To make the files for another language, copy the files of the finished language and translate them by hand.
