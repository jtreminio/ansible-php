jtreminio.php
===============

This role installs and configures PHP, PHP-FPM and PEAR/PECL modules. It features
sane defaults while allowing a user to create a highly configured installation.

You can also install and configure Composer and Xdebug.

Requirements
------------

`jtreminio.common` for some Python modules and firewall management, if wanted.

Install
-------

requirements.yml:

```yaml
- name: jtreminio.php
  src: https://github.com/jtreminio/ansible-php
```

Usage
-----

Sane defaults come included, but everything is configurable.

All PHP settings are controlled by dict `_php_user_settings`,
Composer by dict `_composer_user_settings` and
Xdebug by dict `_xdebug_user_settings`.

```yaml
- set_fact:
    _php_user_settings:
      settings:
        version: '7.2'
      modules:
        php:
          - gd
      ini:
        display_errors: 'On'
      fpm:
        ini:
          error_log: /var/log/php-fpm.log
        pools:
          -
            name: www
            ini:
              prefix: '/var/$pool'
              listen: '127.0.0.1:9000'
    _composer_user_settings:
      install: 1
      settings:
        path: /usr/local/bin/composer
    _xdebug_user_settings:
      install: 1
      settings:
        xdebug.default_enable: '1'
        xdebug.remote_autostart: '0'
        xdebug.remote_connect_back: '1'
        xdebug.remote_enable: '1'
        xdebug.remote_handler: dbgp
        xdebug.remote_port: '9000'

- include_role:
    name: jtreminio.php
```

PHP Settings
========

Please look in `defaults/main.yml` for all available options.

Controlled via `_php_user_settings.settings` dict.

```yaml
_php_user_settings:
  settings:
    version: 7.2
```

**`version`**

Version to install.

Valid versions for Ubuntu can be found at
https://launchpad.net/~ondrej/+archive/ubuntu/php

Valid versions for RHEL can be found at
http://rpms.famillecollet.com/enterprise/7/

String

Default: `7.2`

Modules
----------

```yaml
_php_user_settings:
  modules:
    php:
      - cli
    pear:
      - log
    pecl:
      - apc
```

**`php`**

Controlled via `_php_user_settings.modules.php` list.

Installs modules from distro repositories (via `apt` and `yum`).

Available modules for Ubuntu can be found at
https://launchpad.net/~ondrej/+archive/ubuntu/php

Available modules for RHEL can be found at
http://rpms.famillecollet.com/enterprise/7/

You should only enter the module name, without the `php-` or `php#.#-` prefix.

For example, if you want the `gd` package, use `gd`, and not `php7.2-gd` or `php-gd`.

The prefix is automatically added for you.

String[]

Default:
```yaml
[
  - cli
]
```

**`pear`**

Controlled via `_php_user_settings.modules.pear` list.

Installs a module from PEAR.

If you want beta or alpha versions of a package, append it to the name: `log` -> `log-beta`.

String[]

Default: `[]`

**`pecl`**

Controlled via `_php_user_settings.modules.pecl` list.

Installs a module from PECL.

If you want beta or alpha versions of a package, append it to the name: `apc` -> `apc-beta`.

String[]

Default: `[]`

PHP INI
----------

Controlled via `_php_user_settings.ini` dict.

```yaml
_php_user_settings:
  ini:
    display_errors: 'On'
    error_reporting: '-1'
    date.timezone: UTC
    session.save_path: '/var/lib/php/session'
```

Custom INI settings written to custom INI file such as
`/etc/php/7.2/mods-available/zzzz-custom.ini`.

String{}

Default:
```yaml
{
  date.timezone: UTC
}
```

PHP-FPM Settings
========

Controlled via `_php_user_settings.fpm` dict.

PHP-FPM is installed only if `fpm` appears in the
`_php_user_settings.modules.php` list!

```yaml
_php_user_settings:
  fpm:
    ini:
      error_log: /var/log/php-fpm.log
    pools:
      -
        name: www
        ini:
          prefix: '/var/$pool'
          listen: '127.0.0.1:9000'
```

**`ini`**

See `_php_user_settings.ini` section.

All INI settings here apply only to PHP-FPM.

String{}

Default:
```yaml
{
  error_log: /var/log/php-fpm.log
}
```

Pools
----------

Controlled via `_php_user_settings.fpm.pools` list.

**`name`**

Name of pool.

Required.

String

Default: `www`

**`ini`**

Similar to `_php_user_settings.ini`.

Pool-specific INI settings.

Required.

String{}

Default:
```yaml
{
  prefix: '/var/$pool'
  listen: '127.0.0.1:9000'
  listen.owner: www-data
  listen.group: www-data
  security.limit_extensions: .php
  user: www-data
  group: www-data
  pm: dynamic
  pm.max_children: 5
  pm.start_servers: 2
  pm.min_spare_servers: 1
  pm.max_spare_servers: 3
}
```

Composer
========

Controlled via `_composer_user_settings` dict.

```yaml
_composer_user_settings:
  install: 1
  settings:
    path: /usr/local/bin/composer
```

**`install`**

Install Composer.

Int

Default: `0`

Settings
----------

Controlled via `_composer_user_settings.settings` dict.

See https://github.com/geerlingguy/ansible-role-composer

**`path`**

The path where composer will be installed and available to your system.
Should be in your user's `$PATH` so you can run commands simply with
`composer` instead of the full path.

String

Default: `/usr/local/bin/composer`

**`keep_updated`**

Set this to `true` to update Composer to the latest release every time the playbook is run.

String

Default: `false`

**`version`**

You can install a specific release of Composer, e.g. `composer_version: '1.0.0-alpha11'`.
If left empty the latest development version will be installed. Note that
`_composer_user_settings.settings.keep_updated` will override this variable,
as it will always install the latest development version.

String

Default: `~`

**`home_path`**

The `COMPOSER_HOME` path and directory ownership; this is the directory where global
packages will be installed.

String

Default: `~/.composer`

**`home_owner`**

See `_composer_user_settings.settings.home_path`

String

Default: `root`

**`home_group`**

See `_composer_user_settings.settings.home_path`

String

Default: `root`

**`add_to_path`**

If `true`, and if there are any configured `_composer_user_settings.settings.global_packages`,
the `vendor/bin` directory inside `_composer_user_settings.settings.home_path` will be added to
the system's default `$PATH` (for all users).

String

Default: `true`

**`add_project_to_path`**

Path to a composer project.

String

Default: `false`

**`github_oauth_token`**

GitHub OAuth token, used to avoid GitHub API rate limiting errors when building and
rebuilding applications using Composer. Follow GitHub's directions to
[Create a personal access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)
if you run into these rate limit errors.

String

Default: `~`

**`global_packages`**

A list of packages to install globally (using `composer global require`). If you want to
install any packages globally, add a list item with a dictionary with the `name` of the
package and a `release`, e.g. `- { name: phpunit/phpunit, release: "4.7.*" }`. The
'release' is optional, and defaults to `@stable`.

String

Default: `{}`

Xdebug
========

Xdebug CLI tool is installed at `/usr/local/bin/xdebug`.

CLI debugging will be automatically available. Simply run `xdebug foo.php`.

For PHPStorm instructions look here: https://www.jetbrains.com/help/phpstorm/10.0/configuring-xdebug.html

Controlled via `_xdebug_user_settings` dict.

```yaml
_xdebug_user_settings:
  install: 1
  settings:
    xdebug.default_enable: '1'
    xdebug.remote_autostart: '0'
    xdebug.remote_connect_back: '1'
    xdebug.remote_enable: '1'
    xdebug.remote_handler: dbgp
    xdebug.remote_port: '9000'
```

**`install`**

Install Xdebug.

Int

Default: `0`

**`settings`**

INI flags.

See `_php_user_settings.ini`.

String{}

Default:
```yaml
{
  xdebug.default_enable: '1'
  xdebug.remote_autostart: '0'
  xdebug.remote_connect_back: '1'
  xdebug.remote_enable: '1'
  xdebug.remote_handler: dbgp
  xdebug.remote_port: '9000'
}
```
