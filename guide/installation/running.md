# Running Your App

A ColdIgniter app runs on a web server that hands `.cfm` files to ColdFusion MX 6.1. This page covers the first settings and how to host the site on IIS 4 on Windows NT 4.

!!! important
    Be careful about the case of file names. Windows is not case sensitive, but a server that is will not find a file whose name has the wrong case. Write the file names as they are in the code.

If you are new to ColdIgniter, read the [Getting Started](../intro/index.md) section of the guide first.

!!! note "Difference from CodeIgniter"
    CodeIgniter 4 can run with its own command line development server, with worker mode, on Apache or on nginx. ColdIgniter runs only under a web server that works with ColdFusion MX 6.1. The parts below are written for IIS 4.

## Initial Configuration

### Configure for Your Site URIs

Open the file `app/Config/App.cfc` with a text editor.

1. `baseURL`
    Set `this.baseURL` to the address of the site. If you need more flexibility, set it in the `.env` file as `app.baseURL = 'http://example.com/'`. **Always use a trailing slash on your base URL.**

    !!! note
        If `baseURL` is not set correctly, the links the framework makes point to the wrong place. In development mode the debug toolbar may then not load properly.

2. `indexPage`
    The default is `this.indexPage = "index.cfm"`. If you do not want `index.cfm` in your site URIs, set it to an empty string. The setting is used when the framework makes site URIs.

    !!! note
        You must set up URL rewriting on the web server so the site can be reached without `index.cfm`. See [Removing the index.cfm](#removing-the-indexcfm).

### Configure Database Connection Settings

If you want to use a database, open the file `app/Config/Database.cfc` with a text editor. Set `DBDriver` to `SQLSRV` and `DSN` to the name of a data source of the ColdFusion Administrator. The login is kept in the data source. See [Server Requirements](../intro/requirements.md#supported-databases).

### Set to Development Mode

If the site is not on a production server, set `CI_ENVIRONMENT` to `development` in the `.env` file to get the debugging tools:

```ini
CI_ENVIRONMENT = development
```

!!! important
    On a production server, errors must not be shown. Set the environment to `production`. This is the default when `.env` does not name an environment.

### Set Writable Folder Permission

The code of a site runs under the account of the ColdFusion MX service. That account must be able to write in the `writable/` folder of the project, which holds the cache, the logs, the sessions and the debug toolbar files.

### Checking Server Settings

!!! warning "Not ported yet"
    ColdIgniter has no check of server settings. ColdFusion MX 6.1 is set in the ColdFusion Administrator, and nothing in it has to be changed to run ColdIgniter.

## Local Development Server

ColdIgniter has no built-in development server and no command line tool. To work on a site on your own machine, install ColdFusion MX 6.1 with IIS and set the site up as described below, with a host name such as `myproject.local`.

## Hosting with IIS

A ColdIgniter site is hosted on a web server. The parts below describe IIS 4 on Windows NT 4 with ColdFusion MX 6.1.

### Configure Main Config File

#### Enabling URL rewriting

URLs without `index.cfm` need a rewrite filter on the web server. The file `public/.htaccess` holds the rewrite rules for [URL Magic](https://github.com/muncherelli/urlmagic4), a rewrite filter for IIS that reads rules in the style of Apache. Install the filter in IIS for the site, and it reads that file. Without a rewrite filter the site still works: use addresses with `index.cfm`, such as `http://myproject.local/index.cfm/news`.

The rules do three things: they remove a trailing slash with a redirect, they redirect `www.example.com` to `example.com`, and they send every request that is not for a real file or folder to `index.cfm`. The file has a `RewriteBase` line, commented out, for an installation in a subfolder.

#### Setting Document Root

The home directory of the site must be the `public/` folder, and `index.cfm` must be a default document of the site. The folders `app/`, `system/` and `writable/` must stay outside of the home directory.

### Hosting with VirtualHost

Run each app as a web site of its own, with its own host name.

#### Adding Host Alias

Add a host name for the site to the DNS of your network, or on a development machine to the `hosts` file at `%SystemRoot%\system32\drivers\etc\hosts` (`C:\WINNT` on a default installation). Add a line such as this one, with a name of your choice:

```text
192.0.2.10 myproject.local
```

#### Setting VirtualHost

In the administration tool of IIS, add a web site for the project. Set its home directory to the `public/` folder of the project, for example `D:\Sites\myproject\public`, and give it the host name `myproject.local`. Make sure the folders are laid out like this:

```text
D:\Sites\
    myproject\            (project folder)
        app\
        public\           (home directory of myproject.local)
        system\
        writable\
```

Restart the web site after you change its settings.

#### Testing

With the settings above the site is at `http://myproject.local/` in your browser. Set `app.baseURL` in `.env` to that address.

### Hosting with Subfolder

If you want a base URL such as `http://localhost/myproject/`, keep the project folder outside the home directory of the site and add a virtual directory named `myproject` to the site, with the `public/` folder of the project as its path. Then set `app.baseURL = 'http://localhost/myproject/'` in `.env`.

Do not put the project folder itself in a folder that the web server serves. The files of `app/`, `system/` and `writable/` and your `.env` file would then be reachable by visitors.

### Removing the index.cfm

Set `indexPage` in `app/Config/App.cfc` to an empty string, and set up URL rewriting as described under [Enabling URL rewriting](#enabling-url-rewriting). See also [ColdIgniter URLs](../general/urls.md).

### Setting Environment

Set `CI_ENVIRONMENT` in the `.env` file next to the `app/` folder. See [Handling Multiple Environments](../general/environments.md).

!!! note "Difference from CodeIgniter"
    ColdIgniter reads the environment only from the `.env` file. It does not read a server variable, because a client could set one with a request header.

## Hosting with Apache and nginx

!!! warning "Not ported yet"
    There is no description for these web servers. The file `public/.htaccess` has the rewrite rules in the syntax of Apache, but it is described here for IIS only.

## Deployment to Shared Hosting Services

See [Deployment](deployment.md#deployment-to-shared-hosting-services).

## Bootstrapping the App

!!! warning "Not ported yet"
    CodeIgniter 4 has two bootstrap scripts to load the framework without running the whole application, for tests and for tools. ColdIgniter has neither. The front controller `public/index.cfm` is the only entry point.
