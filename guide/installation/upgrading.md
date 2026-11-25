# Upgrading From a Previous Version

ColdIgniter uses [semantic versioning](https://semver.org/): MAJOR.MINOR.PATCH.

- A patch version has fixes that change nothing an application relies on.
- A minor version has new features, such as a newly ported class.
- A major version has changes that can break an application.

The numbers are ColdIgniter's own and do not follow the numbers of CodeIgniter. Each release says in [CHANGELOG.md](https://github.com/ColdIgniter/coldigniter/blob/trunk/CHANGELOG.md) which version of CodeIgniter its code was ported from.

Read the entry of every version between yours and the new one before you upgrade. See also the [change logs](../changelogs/index.md).

The framework is the `system/` folder. To upgrade, download the new version and replace `system/`. Then compare the files of the project space (`app/`, `public/` and the root files such as `env`) with the new ones and merge the changes into your own files.

For example, compare your `app/Config/App.cfc` with the one of the new download, and add your own settings to the new one.

!!! note
    If you do not know which version you run, open a page in development mode and read the version in the debug toolbar. It is also the value `CI_VERSION` of the component `ColdIgniter\ColdIgniter`.

## Upgrade notes

### From 4.0.0 to 4.1.0

Version 4.1.0 is not released yet. It adds the html and text helpers and the config `ForeignCharacters`, and `Config\DocTypes` now decides whether `img()` and the other html tags close with ` /`. If your views call `img()` or the other html tags, check how they close after the upgrade.
