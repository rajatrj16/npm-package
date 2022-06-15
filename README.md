# Create your first npm package

First npm-package and register locally

_Make sure you already have installed nodejs and npm_

clone the repo: using `git clone git@github.com:rajatrj16/npm-package.git`

Here I have created two dir `my-registry` and `my-registry-testing` 

_Below commands will help you creating your own package_

> cd npm-package/my-registry-testing

> npm install ../my-registry

> node app.js

**Know more indepth at [HERE](https://dev.to/therealdanvega/creating-your-first-npm-package-2ehf)**


# Install, package and publish node application to GitHub registry using github action

### Step 1: Configuring the destination repository
If you don't provide the `repository` key in your package.json file, then GitHub Packages publishes a package in the GitHub repository you specify in the `name` field of the package.json file. For example, a package named `@rajatrj16/my-registry-testing` is published to the `rajatrj16/my-registry-testing` GitHub repository.

However, if you do provide the `repository` key, then the repository in that key is used as the destination npm registry for GitHub Packages. For example, publishing the below package.json results in a package named `my-registry-testing` published to the `rajatrj16/npm-package` GitHub repository.
```  
{
  "name": "@rajatrj16/my-registry-testing",
  "version": "1.0.2",
  "description": "",
  "main": "index.js",
  "repository": {
    "type": "git",
    "url": "https://github.com/rajatrj16/npm-package.git"
  },
  // If you want to use local `.npmrc` you should comment below publishconfig content.
  "homepage": "https://github.com/rajatrj16/npm-package#readme",
  "publishConfig": {
    "registry":"https://npm.pkg.github.com/"
  }
```

### Step 2: Authenticating to the destination repository
To perform authenticated operations against the GitHub Packages registry in your workflow, you can use the `GITHUB_TOKEN`. The `GITHUB_TOKEN` secret is set to an access token for the repository each time a job in a workflow begins.

#### Example
This example stores the `GITHUB_TOKEN` secret in the `NODE_AUTH_TOKEN` environment variable. When the `setup-node` action creates an _.npmrc_ file, it references the token from the `NODE_AUTH_TOKEN` environment variable.

```
name: Publish package to GitHub Packages
on:
  push:
    branches: ['*']
jobs:
  build:
    runs-on: ubuntu-latest 
    permissions: 
      contents: read
      packages: write 
    steps:
      - uses: actions/checkout@v3
      # Setup .npmrc file to publish to GitHub Packages
      - uses: actions/setup-node@v3
        with:
          node-version: '12.21.0'
          registry-url: 'https://npm.pkg.github.com/'
          # Defaults to the user or organization that owns the workflow file
          scope: '@rajatrj16'
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - run: npm install
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
The `setup-node` action creates an _.npmrc_ file on the runner. When you use the `scope` input to the `setup-node` action, the _.npmrc_ file includes the scope prefix. By default, the `setup-node` action sets the scope in the _.npmrc_ file to the account that contains that workflow file.

```
registry=https://registry.npmjs.org/
@rajatrj16:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```
### Step 3: 
Purge the older versions of the package
```
- name: purge packages
  # You may pin to the exact commit or the version.
  uses: actions/delete-package-versions@v3
  with:
    # Number of versions to keep starting with the latest version By default keeps no version. To delete all versions set this as 0.
    min-versions-to-keep: 2
    package-name: 'my-registry-testing'
    # Deletes only pre-release versions. The number of pre-release versions to keep can be specified by min-versions-to-keep. When this is set num-old-versions-to-delete and ignore-versions will not be taken into account. By default this is set to false
    delete-only-pre-release-versions: true
    token: ${{ secrets.GITHUB_TOKEN }}
```

**Reference:** https://docs.github.com/en/actions/publishing-packages/publishing-nodejs-packages
