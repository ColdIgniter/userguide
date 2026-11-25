# Throttler

The Throttler component gives a very simple way to limit an activity to a certain number of attempts within a set period of time. This is most often used for rate limiting an API, or for limiting the attempts a user can make against a form to help prevent brute force attacks. The component can be used for anything that you need to throttle based on actions within a set time interval.

## Overview

The Throttler implements a simplified version of the [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket) algorithm. This treats each action that you want to limit as a bucket. When you call the `check()` method, you tell it how large the bucket is, how many tokens it holds and the time interval. Each `check()` call uses 1 of the available tokens, by default. Here is an example to make this clear.

Say we want an action to happen once every second. The first call to the Throttler looks like the following. The first argument is the bucket name, the second is the number of tokens the bucket holds, and the third is the time it takes the bucket to refill:

```cfm
<cfset throttler = service("throttler")>
<cfset allowed = throttler.check(Hash(request_().getIPAddress()), 60, MINUTE)>
```

Here we use one of the [global constants](../general/common_functions.md) for the time, to make it a little more readable. This says that the bucket allows 60 actions every minute, or 1 action every second.

Say that a third-party script was trying to hit a URL repeatedly. At first, it would be able to use all 60 of those tokens in less than a second. After that the Throttler would only allow one action per second, which may slow down the requests enough that the attack is no longer worth it.

!!! note
    For the Throttler to work, the [Cache](caching.md) library must be set up to use a handler other than `dummy`. The default handler of ColdIgniter is `file`.

!!! note "Difference from CodeIgniter"
    `service("throttler")` returns the shared instance. The Throttler is built on the shared `cache` service. The bucket name is joined to the prefix `throttler_` to make the key in the cache. Use `Hash()` on an IP address, as above, so the key has no characters the cache may refuse.

## Rate Limiting

The Throttler component does not do any rate limiting or request throttling on its own, but it is the key to making one work. Here is an example [filter](../incoming/filters.md) that implements a very simple rate limit of one request per second per IP address, and how to set it up and start using it in your application.

### The Code

You can make your own Throttler filter in `app/Filters/Throttle.cfc`, along the lines of:

```cfm
<cfcomponent output="no" parent="ColdIgniter\Filters\FilterInterface">

	<cffunction name="before" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">
		<cfset var throttler = service("throttler")>

		<!--- restrict an IP address to no more than 1 request per second --->
		<cfif NOT throttler.check(Hash(arguments.request_.getIPAddress()), 1, SECOND)>
			<cfreturn service("response").setStatusCode(429)>
		</cfif>

		<cfreturn "">
	</cffunction>

	<cffunction name="after" access="public" returntype="any" output="no">
		<cfargument name="request_" type="any" required="yes">
		<cfargument name="response" type="any" required="yes">
		<cfargument name="arguments_" type="any" required="no" default="">

		<cfreturn "">
	</cffunction>

</cfcomponent>
```

When run, the `before()` method first gets the shared Throttler. Next, it uses the IP address as the bucket name, and limits the client to one request per second. If the Throttler rejects the check by returning `false`, we return a response with the status code set to 429 (Too Many Requests). The request ends before it reaches the controller. This example throttles based on a single IP address across all requests made to the site, not per page.

### Applying the Filter

We do not need to throttle every page on the site. For many web applications it makes the most sense to apply it only to POST requests, though an API may want to limit every request made by a user. To apply the filter to incoming requests, edit `app/Config/Filters.cfc` and first add an alias for the filter:

```cfm
<cfset this.aliases["throttle"] = "App\Filters\Throttle">
```

Next, assign it to all POST requests made on the site:

```cfm
<cfset postFilters = ArrayNew(1)>
<cfset ArrayAppend(postFilters, "throttle")>
<cfset this.methods["POST"] = postFilters>
```

!!! note
    Filters set in `methods` run only as before filters, which is what the Throttler needs. See [Filters](../incoming/filters.md#methods).

That is all there is to it. Now all POST requests made on the site have to pass the rate limit.

## Class Reference

### check()

`check(key, capacity, seconds, cost)`

- `key` (string): the name of the bucket.
- `capacity` (numeric): the number of tokens the bucket holds.
- `seconds` (numeric): the number of seconds it takes for a bucket to completely fill.
- `cost` (numeric, optional, default `1`): the number of tokens that are spent on this action.
- Returns `true` if the action can be performed, `false` if not.

Checks to see if there are any tokens left within the bucket, or if too many have been used within the allotted time limit. During each check the available tokens are reduced by `cost` if it succeeds.

### getTokenTime()

Returns the number of seconds until another token should be available (a number).

After `check()` has been run and returned `false`, this method can be used to find the time until a new token should be available and the action can be tried again. The minimum wait time is one second. After a `check()` that returned `true`, it returns `0`.

```cfm
<cfif NOT throttler.check(key, 60, MINUTE)>
	<cfset waitSeconds = throttler.getTokenTime()>
</cfif>
```

### remove()

`remove(key)`

- `key` (string): the name of the bucket.
- Returns the Throttler itself.

Removes and resets the bucket. It does not fail if the bucket does not exist.

### setTestTime()

`setTestTime(time)`

- `time` (numeric): the timestamp (seconds) the Throttler uses as the current time.
- Returns the Throttler itself.

This is for testing. It makes the Throttler use the given time instead of the real time, so you can test the refill of a bucket without waiting.

!!! note "Difference from CodeIgniter"
    There are no interfaces on ColdFusion MX 6.1. `ColdIgniter\Throttle\ThrottlerInterface` is a base component with `check()` and `getTokenTime()`, and `ColdIgniter\Throttle\Throttler` extends it. A throttler of your own extends it and has to define both methods, or they throw a `ColdIgniter.LogicException`.
