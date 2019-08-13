---
id: migrations
title: Drupal migrate
sidebar_label: Migrate
---

## There's a lot of traps and bugs and tiny details you can trip off

First of all even though migrate is in drupal core there's couple of contrib modules that are helpful. I use three of these:

 - migrate_plus
 - migrate_tools
 - migrate_source_csv

I might not be sure, but migrate_tools provides UI for migrations (or migrate_plus). CSV is used for importing data from CSV files and migrate_plus allows to register migrations in UI.

There's also several drush commands that migrations comes with.

## Where to start

First of all we need to include these three contrib modules. Once it's downloaded and enabled we can start developing our own custom module with migration.

Migration file is a .yml file that might exist in yourmodule/config/install dir or yourmodule/migrations/
It's worth to note that if the migration file name is just a name of your migration like ck_menu_migrate.yml it's not going to be visible in UI migrations however it's still gonna be visible while listing migrations with drush, btw drush command:

```
drush @pl.local ms
```

To make it visible in UI the file name needs to be adjusted like so:

```
migrate_plus.migration.ck_menu_migrate.yml
```

In .module file there's not much to discover. If we want to use CSV file as a source of our data we can use hook_migration_plugins_alter to change the location of csv file. Why? Because when you define your data in migration yml file it's going to use a start point relative to your drupal root, so you need to provide full path - althout it's relative it's not relative to your module but your drupal root, like so: `profiles/custom/ck/modules/custom/ck_migrate/data/something.csv`. To avoid that we can use `hook_migration_plugins_alter` to change the path. The trick is it's working only for migrations that are not visible in UI, so migrations placed in yourmodule/migrations/your_migration.yml file. If you provide your migration in `yourmodule/config/install/your_migration.yml` this hook has no effect at all. Can't really explain it - anyway we need to stay with a path relative to your drupal root.

.info.yml file doesn't need any extra info, except the dependent modules listed above.

There's also an install file which provides hook_uninstall() that removes config. This is important to remove the config while module is uninstalled. Bare in mind that the migration group will stay anyway, so I guess it needs another line to remove group too, but don't know how yet. 

## Migration yml file

The most important part is migration yml file. There are several different methods to provide it. Let's consider the easy one for csv plugin as source. Here is the example file for migration flat menu with several comments:

```
# Import menu links to default pages, like About us.

# This is the id and it needs to be unique. Probably the name of the file in this case should also contain the id of the migration so it'd be: migrate_plus.migration.ck_menu_top_migrate.yml
id: ck_menu_top_migrate

# This is the group of migration. In the first place I thought this is required to show it in the UI, but that's the name of the file determins whether to show it or not. Still it's better to have it.
migration_group: ck_migrate

# Label of migration
label: 'Create top menu links'

# Source of migration
source:
  # Here is the plugin. Migrate comes with several ready to use different plugins. Each plugin might be changed in php code or you can provide the custom one. It's quite easy to provide custom one. The only thing is to provide a custom drupal Plugin. Starting from beginning, each plugin extends SourcePluginBase class - you can see the example in ./core/modules/migrate/src/Plugin/source/EmbeddedDataSource.php. As you can see this is a literal plugin. CVS class which is provided by migrate_source_csv is also extends SourcePluginBase. So to provide our own plugin we can extend CVS class with our own class and provide plugin name here. Yes, becasue this value is the id of the drupal plugin. Implementation example might be found in ck_migrate module, however I'm extending EmbeddedDataSource.
  plugin: csv

  # as mentioned before it needs to be relative to your drupal root if it's in config/install or might be overriden in hook_migration_plugins_alter if provided in ./migrations dir
  path: profiles/custom/ck/modules/custom/ck_migrate/data/top_menu.csv

  // id column from csv file, for EmbeddedDataSource it's different, invesigate another migrate file in ck_migrate module
  ids: [id]
process:
  # here is described the process of matching drupal fields with csv fields. I don't really know how to get correct drupal fields. I found this example somewhere on the web for menu, however I spent couple of hours for searching up to date structure.
  bundle:
    plugin: default_value
    default_value: menu_link_content
  title: title
  menu_name: menu_name
  link/uri:
    plugin: link_uri
    source:
      - link
  weight: weight
  expanded: expanded
  link/options: '@opts'
destination:
  plugin: entity:menu_link_content

```

You definitely should check another migration yml file included in ck_migrate module for better understanding custom plugins.

## Pitfalls, error handling and drush usage

1. If you provided changes to yml file migration you need to reinstall module. If the module doesn't have hook_uninstall code you need to first remove migration from UI (if it was UI way defined) and then reinstall. If it was not registered in UI then only reinstall module will do the job:

```
lando drush @pl.local dre ck_migrate
```

You can also easily update your config and import it into drupal by importing partial config from config/install dir - this method is not working recursivly, so it needs to be triggered in every separated dir.

```
lando drush @pl.local config-import --partial --source=profiles/custom/ck/modules/custom/ck_migrate/config/install
```

2. If for some reason you'll end up with `The ids must a flat array with unique string values.` error in UI when browsing the source of migration this is the core issue combined with `migrate_source_csv` module. Migration should still work just fine in drush. Here is more info: [https://www.drupal.org/project/migrate_source_csv/issues/3068017](https://www.drupal.org/project/migrate_source_csv/issues/3068017) 

3. To execute migration:
```
lando drush @pl.local migrate:import id_of_migration
```

4. To rollback migration:
```
lando drush @pl.local migrate:rollback id_of_migration
```

5. Sometimes for some reason migration will stuck on import error. And when you list migrations with `drush ms` it will display status `importing` even if it's not. In that case we need to reset the migration with:
```
lando drush @pl.local migrate:reset-status id_of_migration
```

6. If you want to debug custom plugin you can provide ksm on prepareRow() method and make a migration from UI. After the migration data should be displayed.

7. `failed to open stream: No such file or directory CSV.php:262` - can't find the csv file, something is wrong with location of the file.

8. `In DiscoveryTrait.php line 53: The "ck_menu_item" plugin does not exist.` or `In DiscoveryTrait.php line 53: The "" plugin does not exist.` - this has been because module was disabled but the migration stayed or (second issue) in hook_migration_plugins_alter I was refering to unexistant migration.

In general there are several issues you can face. This is not that complex, but there's a lot of pitfalls and bugs and errors does not tell much. Working migration can also be found in migrate_source_csv module as a test module migrate_source_csv_test. There's also a documetation (a bit out of date but still) here: [https://www.liip.ch/en/blog/using-the-new-drupal-8-migration-api-module](https://www.liip.ch/en/blog/using-the-new-drupal-8-migration-api-module) and modules examples.

