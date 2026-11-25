# Troubleshooting

Here are some common installation problems, and suggested workarounds.

## How do I know if my install is working?

Open the address of the site in your browser, for example `http://myproject.local/`. It should show the default welcome page of ColdIgniter.

## I have to include index.cfm in my URL

If a URL like `/mypage/find/apple` does not work, but the similar URL `/index.cfm/mypage/find/apple` does, the URL rewriting of the web server is not set up. See [Enabling URL rewriting](running.md#enabling-url-rewriting) and [ColdIgniter URLs](../general/urls.md).

## Only the default page loads

If no matter what you put in your URL only the default page loads, the web server may not give the framework the request URI that it expects. Open `app/Config/App.cfc` and look at `uriProtocol`. The default is `REQUEST_URI`. The other values are `QUERY_STRING` and `PATH_INFO`. Try them one by one.

!!! warning
    With `PATH_INFO`, URIs are always URL-decoded.

## My app works fine locally but not on the production server

Make sure that the case of the folder and file names matches the code.

Windows is not case sensitive, but other servers can be. For example, when you have `app/Controllers/Product.cfc`, you must use `Product` as the class name, not `product`.

If the case of a file name is wrong, the file is not found on such a server.

## The tutorial gives 404 errors everywhere :(

The tutorial uses URLs without `index.cfm`. Without URL rewriting on the web server, only URLs with `index.cfm` in them reach the framework. Set up the rewriting as described in [Running Your App](running.md#enabling-url-rewriting), or set `indexPage` in `app/Config/App.cfc` to `index.cfm` (the default) and use `index.cfm` in the URLs.

## What's with an unhelpful "Whoops!" page?

You find that your app shows a page with "Whoops!" and then the text line "We seem to have hit a snag. Please try again later...".

That is a sign that you are in production mode and have hit an unrecoverable error, which is not shown to the visitor, for better security.

You can see the error in the log file. See [ColdIgniter Error Logs](#coldigniter-error-logs) below.

If you reach this page while developing, change your environment to `development` in `.env`. See [Set to Development Mode](running.md#set-to-development-mode). After that, reload the page. You will see the error and the trace.

## ColdIgniter Error Logs

The logger writes one file per day to the folder `writable/logs/`, named like `log-2026-11-25.log`. In `production` the threshold is 4, so emergencies, alerts, critical errors and errors are logged. In other environments it is 9. The setting is `threshold` in `app/Config/Logger.cfc`. See [Logging Information](../general/logging.md).

If no log file appears, check that the account of the ColdFusion MX service can write in `writable/logs/`.
