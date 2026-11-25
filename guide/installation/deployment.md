# Deployment

## Optimization

Before you deploy a ColdIgniter application to production, there are a few things you can do to make it run more efficiently.

!!! note "Difference from CodeIgniter"
    CodeIgniter 4 has a `spark optimize` command, Composer optimization and PHP preloading. ColdIgniter has no command line tool, no Composer and nothing to preload, so these are left out.

### Production environment

Set `CI_ENVIRONMENT` to `production` in the `.env` file. This is the default. In production, errors are not shown, the debug toolbar is off and fewer messages are logged. See [Running Your App](running.md#set-to-development-mode).

### Config Caching

!!! warning "Not ported yet"
    The caching of Config objects in `app/Config/Optimize` is not ported.

### FileLocator Caching

!!! warning "Not ported yet"
    The FileLocator class and its cached version are not ported.

### Page caching

A controller can cache a whole page. This is the one optimization that ColdIgniter has, outside the cache library. See [Web Page Caching](../general/caching.md).

## Deployment to Shared Hosting Services

!!! important
    `index.cfm` is not in the root of the project. It is in the `public/` folder, for better security and separation of components.

    This means you must set the web server to use the `public/` folder as the home directory of the site, and not the project root.

### Specifying the Document Root

The best way is to set the home directory of the site to the `public/` folder in the web server settings:

```text
example.com/ (project folder)
    public/  (home directory)
```

Check with your hosting provider whether you can change the home directory. If you cannot, use the next way.

### Using Two Directories

The second way is to use two directories and adjust the path. One is for the application and the other is the default home directory of the site.

Upload the contents of the `public/` folder to the default home directory, and the other folders to the directory for the application:

```text
example.com/ (for the application)
    app/
    system/
    writable/
public_html/ (the default home directory)
    index.cfm
    robots.txt
```

Then change the `cfinclude` line in `index.cfm` that names `app/Config/Paths.cfm`, so it reaches the `app/` folder in its new place. The paths in `app/Config/Paths.cfc` are relative to the project root, so check them too.

!!! note
    Put the `.env` file in the application directory, next to `app/`. The setting `envDirectory` in `app/Config/Paths.cfc` names its folder.
