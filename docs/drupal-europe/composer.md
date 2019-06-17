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
