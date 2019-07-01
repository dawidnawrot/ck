---
id: environment-global-setting
title: Environment global setting
sidebar_label: Environment global setting
---

## settings.php

Global environment setting was defined in `settings.php` file. The setting is based on environmental variable `WORKING_ENVIRONMENT`. This is the code:

```
$settings['ck_environment'] = getenv('WORKING_ENVIRONMENT');
```

If you need to make a condition anywhere in the code that relates to different values on different environemtns use following syntax:

```
use Drupal\Core\Site\Settings;
...
if (Settings::get('ck_environment') === 'local') {
  // Do anything you want
}
```

## Future plans

I suppose we're gonna have separate settings.php files based on each environment, but in some cases this is useful. It's been used in preprocess_html to load correct library based on environment.
