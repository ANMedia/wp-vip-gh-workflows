# WordPress VIP Github Action Workflows

Repository to store reusable Github Action workflow configuration files for our WordPress VIP applications. 


## Example Project file

Example WP VIP Application project file:

```
name: Build and Deploy

on:
  push:
    branches-ignore:
      - "**-built"
      - "**-built-**"
  workflow_dispatch:

permissions: write-all

jobs:

  do-lint:
    name: Lint PHP & JS
    uses: ANMedia/wp-vip-gh-workflows/.github/workflows/lint.yml@main
    secrets: 
      COMPOSER_AUTH: ${{ secrets.COMPOSER_AUTH }}

  do-phpunit:
    name: Run PHPUnit
    uses: ANMedia/wp-vip-gh-workflows/.github/workflows/phpunit.yml@main
    secrets: 
      COMPOSER_AUTH: ${{ secrets.COMPOSER_AUTH }}

  do-built-deploy:
    name: Deploy to -built
    needs: [do-lint]
    uses: ANMedia/wp-vip-gh-workflows/.github/workflows/deploy-built.yml@main
    secrets: 
      COMPOSER_AUTH: ${{ secrets.COMPOSER_AUTH }}
  
  do-release:
    name: Trigger Release Workflow
    needs: [do-built-deploy]
    if: ${{ github.ref_name == 'production' }}
    uses: ANMedia/wp-vip-gh-workflows/.github/workflows/release.yml@main
    secrets: 
      NEW_RELIC_API_KEY: ${{ secrets.NEW_RELIC_API_KEY }}
      NEW_RELIC_DEPLOYMENT_ENTITY_GUID: ${{ secrets.NEW_RELIC_DEPLOYMENT_ENTITY_GUID }}
  
  do-reset:
    name: Trigger Reset Workflow
    needs: [do-built-deploy]
    uses: ANMedia/wp-vip-gh-workflows/.github/workflows/reset.yml@main
```


## Workflows

Summary descriptions of the available workflows

### Deploy to Built

Build the application with Composer and NPM. Push to the WordPress VIP deployment branch `-built`

### Lint

Run the PHP and JS linters. Requires commands like:

`composer run lint:php`
`composer run phpstan`
`npm run lint:js`

### PHPUnit

Execute PHPUnit tests via `composer run phpunit`

### Release

* Create a git tag for the current sha.
* Build and commit a changelog based on the current and last tag.
* Promote this tag to a release.
* Set a deployment marker in New Relic.

### Reset

If a test$N or develop branch, reset to the HEAD of the production branch
If a production release, reset all test$N and develop branches to production HEAD.
