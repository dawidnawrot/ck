---
id: development
title: Docusaurus development
sidebar_label: Development
---

## General

Docusaurus framework was used to provide documentation for CircleK websites.

## Running local Docusaurus app

In order to extend documentation you need to have node.js installed on your local machine. If you don't have it go to [the page](https://nodejs.org/en/), download it and install. Node js comes with [npm](https://en.wikipedia.org/wiki/Npm_(software)) package manager. Vicariously can also use `yarn`.

Clone `https://github.com/dawidnawrot/ck` repository.
Go to `./website` dir and run `npm install` or `yarn install` command. This will download all required packages to make your local development work.
Run `npm run` or `yarn start` command. This will start your application and redirect to a browser where app is going to render.

## Files and directories of your interest

`./docs` - that's where the content of each pages is.
`./website/static/index.html` - that's where the redirect takes place to custom document.
`./website/sidebar.json` - that's where the **sidebars**, its categories and eventually docs are defined.
`./website/siteConfig.js` - that's where the config of a app is.

## Adding new custom page.

Let's assume you'd like to add a new page (eg. config workflow) to Drupal Europe documentation.

1. Create a file in `./docs/drupal-europe/config-workflow.md`
The contents of a file needs to include:

```
---
id: config-workflow
title: Drupal config workflow
sidebar_label: Config
---

Content of your documentation...
```

Once created it can be visited at: http://localhost:3000/ck/docs/drupal-europe/config-workflow

## How to add a custom page to sidebar

Open up `./website/sidebar.json` file and place it in the Drupal Europe section, like so:

```
"drupal-europe": {
  "Drupal Europe": [
    "drupal-europe/intro",
    "drupal-europe/troubleshooting",
    "drupal-europe/config-workflow"
  ]
},
```

The last element is a `config-workflow`.

**Important note**

`config-workflow` term in `"drupal-europe/config-workflow"` section is not the name of the file, it's an id of the document included in document itself. So the name of the file might be anything you want but when adding a doc to sidebar the concept is `directory/id` not ~~`directory/filename`~~.

## Troubleshooting

Docusaurus provides live reload option, however this applies to doc content only. So, if you create a new file to `./docs` directory and add it to sidebar.json file you have to stop `npm run` or `yarn start` with `CTRL+C` and rerun it again to see the changes. Once new doc is accessible you can edit its content and see the changes in real time in your browser.

## Commiting your changes.
Once the doc you created is ready to publish commit your changes, push it to repo and create new PR. I'll merge the change to master and publish it to gh pages.

## More info
For more information check the [documentation](https://docusaurus.io) for how to use Docusaurus.
