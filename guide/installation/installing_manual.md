# Manual Installation

The [ColdIgniter repository](https://github.com/ColdIgniter/coldigniter) holds the source of the framework and a starter application. Develop your application inside the `app/` folder, and the `public/` folder is your public-facing home directory. Do not change anything inside the `system/` folder.

!!! note "Difference from CodeIgniter"
    In CodeIgniter 4 the framework and the starter project are two repositories, and the framework can also come from Composer. In ColdIgniter one repository holds everything and there is no Composer.

## Installation

Download the repository from GitHub as a zip file, or clone it, and extract it to become your project root. No release is published yet, so the download is the current state of the `trunk` branch.

Folder and file names are not case sensitive on ColdFusion MX 6.1 under Windows, but keep the case the same as in the code so the site also works if it moves to a server that is.

## Initial Configuration

After installation, a few settings are needed. See [Initial Configuration](running.md#initial-configuration).

## Upgrading

Download a new copy of the framework and replace the `system/` folder.

Read the [upgrade instructions](upgrading.md) and the [change log](../changelogs/index.md), and check the changes that can break an application.

## Pros

Download and run.

## Cons

You need to check for file changes in the project space (the root, `app/`, `public/` and `writable/`) and merge them yourself.

## Structure

Folders in your project after set up:

- `app`, `public`, `system`, `tests`, `writable`

## Translations Installation

!!! warning "Not ported yet"
    The separate translations repository of CodeIgniter 4 has no counterpart yet. Only the English system messages ship, in `system/Language/en/`.
