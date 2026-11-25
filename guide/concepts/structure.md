# Application Structure

To get the most out of ColdIgniter, you need to understand how the application is structured, by default, and what you can change to meet the needs of your application.

## Default Directories

A fresh install has five directories:

- **app**
- **public**
- **writable**
- **tests**
- **system**

Each of these directories has a very specific part to play.

### app

The **app** directory is where all of your application code lives. This comes with a default directory structure that works well for many applications.

The following folders make up the basic contents:

```text
app/
    Config/         Stores the configuration files
    Controllers/    Controllers determine the program flow
    Database/       Stores the database migrations and seeds files
    Helpers/        Helpers store collections of standalone functions
    Views/          Views make up the HTML that is displayed to the client
```

A fresh install holds only some of these folders. Create **Models**, **Filters**, **Libraries**, **Helpers** and others when you need them. Models are normally kept in **app/Models**, as in the example application.

Two files sit at the top of **app**:

- **Common.cfm** is a place for your own global functions. It is loaded early and is the place to replace a function of **system/Common.cfm**.
- **autoload.cfm** is a helper of the autoloader. Do not edit it. See [Autoloading Files](autoloader.md).

The classes of **app** are found through the `App` namespace. A class file is found by its name: the class `App\Models\CustomerModel` is the file **app/Models/CustomerModel.cfc**. You are free to change the structure of this directory to suit your application. For example, you might decide to use the Repository pattern and Entities to work with your data. In this case, you could rename the **Models** directory to **Repositories**, and add a new **Entities** directory. Then create the classes with their new names, for example `App\Repositories\UserRepository`.

All files in this directory live under the `App` namespace, though you are free to change that: the name is set by `APP_NAMESPACE` in **app/Config/Constants.cfm**.

!!! note "Difference from CodeIgniter"
    A namespace in ColdIgniter is not a language feature. It is the first part of a class name written as a text, such as `App\Models\CustomerModel`, and the autoloader maps it to a folder. Files are `.cfc` (components) and `.cfm` (templates and function files).

### system

This directory stores the files that make up the framework, itself. While you have a lot of flexibility in how you use the application directory, the files in the system directory should never be modified. Instead, you should extend the classes, or create new classes, to provide the desired functionality.

All files in this directory live under the `ColdIgniter` namespace.

The directory holds one folder for each area of the framework (**Cache**, **Database**, **Filters**, **HTTP**, **Router**, **Session**, **Validation**, **View** and so on). Next to them are **Polyfills**, a folder of small functions that stand in for the functions of the original code that CFML lacks, and **Common.cfm**, the global functions such as `service()` and `esc()`.

!!! note "Difference from CodeIgniter"
    There is no Composer, so the framework is never in a **vendor** folder. The **system** folder is always next to **app**.

<a id="application-structure-public"></a>

### public

The **public** folder holds the browser-accessible portion of your web application, preventing direct access to your source code.

It contains **index.cfm**, the front controller, and any application assets that you add, like CSS, javascript, or images.

This folder is meant to be the "web root" of your site, and your web server would be configured to point to it.

!!! note "Difference from CodeIgniter"
    The framework does not ship a **.htaccess** file in **public**. Setting up the web server so that addresses reach **index.cfm** is part of the installation, see [Installation](../installation/index.md).

### writable

This directory holds any directories that might need to be written to in the course of an application's life. It has the folders **cache**, **debugbar**, **logs** and **session**. These hold the cache files, the debug toolbar data, the log files and the session files.

You should add any other directories that your application will need to write to here. This allows you to keep your other primary directories non-writable as an added security measure.

### tests

This directory is set up to hold your test files. A fresh install holds **lint.py**, a script of static checks for the CFML files, and a README. The mock classes of the framework are in **system/Test/Mock**.

This directory does not need to be transferred to your production servers.

## Modifying Directory Locations

If you've relocated any of the main directories, you can change the configuration settings inside **app/Config/Paths.cfc**. It holds the locations of the system, app, writable, tests, view and env directories. **app/Config/Paths.cfm** creates it, and **public/index.cfm** includes **Paths.cfm**: that include line is the one to change when you move the **app** folder.

Please read [Managing your Applications](../general/managing_apps.md).
