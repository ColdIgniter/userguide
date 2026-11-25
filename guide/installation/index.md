# Installation

## The folders

| Folder | What it holds |
|---|---|
| `app/` | Your application: configuration, controllers, models and views. |
| `public/` | The only folder the web server serves. It holds the front controller, `index.cfm`. |
| `system/` | The framework. Do not change files here. |
| `writable/` | Files the application writes: cache, logs, sessions and uploads. |

## Steps

1. Copy the folders to the server.
2. Make `public/` the home directory of the web site. The folders `app/`, `system/` and `writable/` must stay outside of it, so that no visitor can ask for their files.
3. Copy the file `env` to `.env`, next to the `app/` folder.
4. In `.env`, set the address of the site:

    ```ini
    app.baseURL = 'http://example.com/'
    ```

5. Open the site in a browser. The welcome page shows that the install works.

!!! warning "Keep the `.env` file private"
    Never put the `.env` file under version control, and never in a folder that the web server serves.

## The environment

The setting `CI_ENVIRONMENT` in `.env` picks the environment. The default is `production`.

```ini
CI_ENVIRONMENT = development
```

In `development`, errors are shown in detail and the debug toolbar is on. Use `production` for a live site.
