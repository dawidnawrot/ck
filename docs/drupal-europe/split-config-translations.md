---
id: split-config-translations
title: Split config translations
sidebar_label: Split config translations
---
### Status: TODO

## Issue

Split config exports yaml files based on gray or black list convention. It works fine for english language however it's getting complicated when another language is added. In that scenario when exporting happens, `language` dir is created in `sync` diretory and only these translation which were black listed or gray listed are stored in split directory. The rest of it is placed in main `sync` directory. It's not a big issue as these configs
holds translations only, but to be precise it should be placed in split direcotory.

## Solution

I believe the one way to overcome this issue is to create another custom module based on config_filter (the same as it's used by config_split) and force `config-split:export` command to place translated configs in site repo. There's not much documentation about it, but there is a config_filter_split_test module in tests directory of config_fiter module, also config_split and [config_ignore](https://git.drupalcode.org/project/config_ignore/tree/8.x-2.x) uses config_fiter.
