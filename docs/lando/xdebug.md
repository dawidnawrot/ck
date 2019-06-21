---
id: xdebug
title: xDebug
sidebar_label: xdebug
---

## Enabling xDebug

xDebug is enabled by default, so you don't really need to do anything. It's in .lando.yml file. There's also extra part of php.ini provided with default configuration for xdebug. There's also a config file for VCS to make it work. If you're using different IDE you're on your own to figure out how to set it up, still should be pretty straight forward. 

Part of `.lando.yml` file resposible for enabling xdebug:

```
config:
  webroot: .
  xdebug: true
  config:
    vhosts: config/custom.conf
    php: .vscode/php.ini
```

`xdebug: true` and `php: ./vscode/php.ini` enables it and provides php config for xdebug.

Contents of `./vscode/launch.json` file:
```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9000,
      "log": true,
      "pathMappings": {
        "/app/sites/pl": "${workspaceFolder}/sites/pl",
      }
    }
  ]
}
```

This setup is pointing to pl site. If you want to switch to different site just change prefix for the directory in `pathMappings` property and it's done.

## Local variables issue

It's been noticed that xdebug does not always displays local variables. It turned out opCache is the culprit. I've disabled it on drupal level in settings.local.php file. This file is loaded only on local development. There's an `ini_set('opcache.enable', 0);` expression at the very end of the file which overrides php setting variable and makes opcache disabled. This makes xdebug work locally presented with all variables.
