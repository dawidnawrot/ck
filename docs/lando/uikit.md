---
id: uikit
title: UI Kit
sidebar_label: UI Kit
---

## Service

Lando creates separate container to serve and process uikit library which is used as core part of `circlek` drupal theme. Once lando is setup you can access: [`http://ckfront.lndo.site`](http://ckfront.lndo.site) site to get the file listing. It's listing `dist` directory only. CSS and JS files are used in circlek.libraries.yml file in our `circlek` theme.

## Development

To get runner work just type `lando yarn watch` or shh to a ckfront service: `lando ssh -s ckfront`, go to `/app/uikit` dir and type: `yarn watch`. While providing changes your css and js is going to be rebuild on each file save.

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

Service uses [`http-server`](https://github.com/indexzero/http-server) simple package to serve static files which is installed as global package and command to run server is hs. `-c-1` parameter force server not to cache anything (by default server caches served data).

There was an issue with files not refreshed after save. It turned out uikit uses [npm-watch](https://www.npmjs.com/package/npm-watch) package which uses nodemon to monitor the changes. npm-watch requires a script and additional block in package.json to make it work. At the end of this doc you can find a diff between default one provided by uikit and the one adjusted by me.

It does work on node installed directly on machine, however when node is installed in the subsystem (eg. docker container) sometimes there is an issue with restarting after file save. To make it work [legacy-watch](https://github.com/remy/nodemon#application-isnt-restarting) option needs to be turned on. Despite the fact that this option comes from nodemon it was also added to [npm-watch](https://www.npmjs.com/package/npm-watch#legacywatch), but uikit people didn't add it. So I had to adjust 
package.json and it works fine now. This is kind of a bug and will report it to uikit team.

### Default
```
"scripts": {
  "watch": "npm-watch",
},
"watch": {
  "compile-js": "src/js/**/*.js",
  "compile-less": {
    "patterns": "**/*.less",
    "extensions": "less"
  }
}
```

### Adjusted
```
"scripts": {
  "watch": "npm-watch",
},
"watch": {
  "compile-js": {
    "patterns": "src/js/**/*.js",
    "extensions": "js",
    "legacyWatch": true
  },
  "compile-less": {
    "patterns": "**/*.less",
    "extensions": "less",
    "legacyWatch": true
  }
}
```
