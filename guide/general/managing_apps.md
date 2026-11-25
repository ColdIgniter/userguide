# Managing your Applications

By default, it is assumed that you only intend to use ColdIgniter to manage one application, which you will build in your **app** directory. It is possible, however, to have multiple sets of applications that share a single ColdIgniter installation, or even to rename or relocate your application directory.

## Renaming or Relocating the Application Directory

If you would like to rename your application directory or even move it to a different location on your server, other than your project root, open your main **app/Config/Paths.cfc** and set a *full server path* in the `appDirectory` property:

```cfm
<cfset this.appDirectory = "/path/to/your/app">
```

You will need to modify one additional file in your project root, so that it can find the **Paths** configuration file:

- **public/index.cfm** is the front controller for your webapp. It includes **Paths.cfm**, the small file next to **Paths.cfc** that creates the component. Change the path of this line:

```cfm
<cfinclude template="../app/Config/Paths.cfm">
<!--- ^^^ Change this line if you move your application folder --->
```

!!! note "Difference from CodeIgniter"
    There is no `spark` file. Keep **Paths.cfm** in the same folder as **Paths.cfc**: a component can only be created by a file in its own folder tree.

## Running Multiple Applications with one ColdIgniter Installation

If you would like to share a common ColdIgniter framework installation, to manage several different applications, simply put all of the directories located inside your application directory into their own (sub)-directory.

For example, let's say you want to create two applications, named **blog** and **shop**. You could structure your application project directories like this:

```text
blog/
    app/
    public/
    tests/
    writable/
    env
shop/
    app/
    public/
    tests/
    writable/
    env
system/
```

This would have two apps, **blog** and **shop**, both having standard application directories and a **public** folder, and sharing a common **system** folder.

The `systemDirectory` property in **app/Config/Paths.cfc** inside each of those would be set to refer to the shared common **system** folder:

```cfm
<cfset this.systemDirectory = __DIR__ & "/../../../system">
```

Only when you change the Application Directory, see [Renaming or Relocating the Application Directory](#renaming-or-relocating-the-application-directory) and modify the path in the **index.cfm**.

!!! note "Difference from CodeIgniter"
    There is no Composer and no `COMPOSER_PATH` constant, so there is nothing else to change. `__DIR__` is the folder of the **Paths.cfc** file.

## Changing the Location of the .env File

If necessary, you can change the location of the **.env** file by adjusting the `envDirectory` property in **app/Config/Paths.cfc**.

By default, the framework loads environment settings from a **.env** file located one level above the **app** directory (in the `ROOTPATH`). This is a safe location when your domain is correctly pointed to the **public** directory, as recommended. See [Running Your App](../installation/running.md).

In practice, however, some applications are served from a subdirectory (e.g., `http://example.com/myapp`) rather than from the main domain. In such cases, placing the **.env** file within the `ROOTPATH` may expose sensitive configuration data if the web server protections are misconfigured.

To avoid this risk in such setups, it is recommended that you ensure the **.env** file is located outside any web-accessible directories.

!!! warning
    If you change the location of the **.env** file, make absolutely sure it is not publicly accessible. Exposure of this file could lead to compromised credentials and access to critical services, such as your database, mail server, or third-party APIs.
