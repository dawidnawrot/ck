---
id: composer
title: Composer
sidebar_label: Composer
---

## Single file composer

Although there are several projects we're working on there's just one common composer file shared across all the sites.

## Justification

This is becasue it's easy to keep it up to date and easy to maintain. The one and only disadvantage of that kind of approach is that some (not much) projects / modules are downloaded and never used / installed. I've tried to implement multiple composer files but it's difficult to maintain (you can read about that kind of approach in "Pitfalls" section). Currently composer.json and composer.lock files are part of profile repository. It's placed in the root dir of this repo. During the deployment it's coppied to single site directory and then installed. If you provide any new projects to composer you supposed create a new branch in profile repo and copy it back (both composer.json and composer.lock).

## Pitfalls

The initial idea was to create one global composer.json which would be part of profile (similarily to the one above). However this composer.json file would use [`wikimedia/composer-merge-plugin`](https://github.com/wikimedia/composer-merge-plugin) plugin which allows to `include` external (additional) composer files. So, when deployment process happens for the first time composer.json and composer.lock are copied from profile into site directory, but there's also an additional composer.local.json file that's part of site repo. In the main composer.json file in extra section we have:

```
"extra": {
        "merge-plugin": {
            "include": [
                "composer.local.json"
            ],
            "recurse": false,
            "replace": false
        }
    }
```

This allows to include local composer.local.json file and dependencies are downloaded. It works great, but the process of moving main composer.json file back to profile is tedious. This plugin allows to combine several external composer.json files, but we can only have single composer.lock file. So, in order to move it back we would have to edit manually composer.lock file and remove dependencies that are part of local site. Moreover this gets us to the point where we need to specify exact version of the dependency / module in composer.local.json file, we can't use range because eventually next developer might end up with different version of the module / dependency. So, this approach makes it quite complex and that's why it's easier to maintain just a single composer.json and composer.lock files.

## UPDATE 1.09.2019

After deep investigation I've decided to stay with just a single composer file that is storred in profile `./composer.json` main directory. I've tried to create a separate composer for site and for profile, but it didn't work. Composer file for profile contained all the dependencies including drupal core, modules, patches etc. Site composer file on the other hand contained a profile composer dependency and other site dependencies like additional modules specific to the site, like so:

```
"repositories": [
    {
        "type": "git",
        "url": "git@github.com:dawidnawrot/ck_profile.git"
    },
    {
          "type": "composer",
          "url": "https://packages.drupal.org/8"
    }
  ],
  "require": {
      "dawidnawrot/ck_profile": "dev-drupal_only",
      "drupal/admin_toolbar": "^2.1"
  },
```

The problem was and `extra` section in profile composer file:

```
"extra": {
    "installer-paths": {
        "web/core": ["type:drupal-core"]
    }
}
```

When I ran `composer install` from main profile directory my drupal core end up in /web directory, however when I ran `composer install` from site (which requires profile composer) my drupal core (and other drupal specific elements like modules, themes, etc which are not listed here just for simplicity reasons) end up in main directory instead of ./web dir. It turned out that composer ignores `extra` section for transitive dependencies. So, it's not just installer-paths but also if there were defined any drupal patches it also woudn't be installed. Only the last composer file and its `extra` section is triggered. Of course I could include overrides in last composer file which is the site composer and it would work for location of drupal core but not for patches. And that is just one issue, who knows what else we will have in extra section in the future. So with that said it's easier to stay with a single composer file, of course the consequence of keepnig single composer file is that all contrib modules are going to be shared in all sites, but not all of them are going to be enabled. For now there's no such module which is not enabled because we're working on core functionalities, but maybe in the future there's going to be a contrib module that is specific to certain site, but it's spread across all sites - the thing is it's going to be enabled on particular site, becasue that's what config_split allows for, so not much of a pain. There also was an option to bring some post install hooks and move files accordingly, but it's too much work and not much of advantage.

