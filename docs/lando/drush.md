---
id: drush-aliases
title: Drush aliases
sidebar_label: Drush aliases
---

## How to run drush

Lando by default provides drush with drupal8 reciepe. However it's drush 8 version. If composer file of installed site defines its own drush this one is going to be used. To force lando to use different global drush version we can define it in .lando.yml file like so:

```
name: lm
recipe: drupal8
config:
  webroot: .
  xdebug: true
  drush: "*" #this defines the newest drush global version and it's always going to be updated on lando rebuild once new version is released.
```

## How to setup drush aliases

Lando documentation defines how to do it, but the documentation and default version of drush is based on drush 8. However drush 9 does not use .php files anymore. It switched to `.yml` files. So, once we defined newest lando version in `.lando.yml` file it's going to be at least version 9. But before we define aliases we need to define location of drush aliases file in `drush.yml` file which is the config for drush. Now, drush comes with it's default version of `drush.yml` file, but it can be overriden. There are few locations that drush will look up for overriden version of `drush.yml`. One of the locations is (more info [here](https://github.com/drush-ops/drush/blob/master/examples/example.drush.yml)):

```
~/.drush/drush.yml
```

Once the file is there (lando is symlinking `drush.yml` file from ./drush/drush.yml to `~/.drush/drush.yml` during the lando build process defined in appserver build) we can define aliases files. Aliases file for .pl is also placed in lando repo in `./drush/site-aliases/pl.site.yml` and is also symlinked, but this time all the directory is symlinked rather than a single alias file. Once dir is symlinked we should be able to run drush commands with aliases and it should work from anywhere. Because drush command is added to `$PATH` it's available anywhere on the systetem and because aliases and drush config exists in `~/.drush` dir it's triggered during drush command. So, in case of .pl we run:

```
drush @pl.local status
```

and we should get the status of the site. 

Contents of `pl.site.yml` looks like this:

```
local:
  root: /app/sites/pl/web
  uri: http://pl.lm.lndo.site

```

If we would like to define another site we just add another file to `./drush/site-aliases` dir in lando repo and define it similar way as .pl.
In the appserver build lando process I delete default `site-aliases` dir and then symlink is created.
