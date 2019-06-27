---
id: uikit
title: UI Kit
sidebar_label: UI Kit
---



## Service

Lando creates separate container to serve and process uikit library which is used as core part of `circlek` drupal theme. Once lando is setup you can access: [`http://ckfront.lndo.site`](http://ckfront.lndo.site) site to get the file listing. It's listing `dist` directory only. CSS and JS files are used in circlek.libraries.yml file in our `circlek` theme.

## Development
To get runner work just shh to a ckfront service: `lando ssh -s ckfront`, go to `/app/uikit` dir and type: `yarn watch`. While providing changes your css and js is going to be rebuild on each file save.

## Lando part

This is the code responsible for setting up `node` container:
**Service**
```
services:
  ckfront:
    type: node:10
    webroot: ./uikit
    globals:
      http-server: latest
    command:
      cd ./uikit && hs ./dist -c-1
    scanner: false
```

**Proxy**
```
proxy:
  ckfront:
    - ckfront.lndo.site:8080 # This port needs to say here.
```
