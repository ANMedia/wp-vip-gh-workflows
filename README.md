# WordPress VIP Github Action Workflows

Repository to store [reusable Github Action Workflow](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) configuration files for DMG WordPress VIP applications.

## Example Project File

Each WordPress VIP application should contain a workflow file within `/.github/workflows` that calls the necessary reusable workflows. Some projects may not call all workflows (e.g. PHPUnit)

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
        #needs: [do-lint]
        uses: ANMedia/wp-vip-gh-workflows/.github/workflows/deploy-built.yml@main
        secrets:
            COMPOSER_AUTH: ${{ secrets.COMPOSER_AUTH }}

    do-release:
        name: Trigger Release Workflow
        needs: [ do-built-deploy ]
        if: ${{ github.ref_name == 'production' }}
        uses: ANMedia/wp-vip-gh-workflows/.github/workflows/release.yml@main
        with:
            commit_changelog: false
            update_wiki: true

    do-new-relic:
        name: Run New Relic Workflow
        needs: [ do-built-deploy ]
        uses: ANMedia/wp-vip-gh-workflows/.github/workflows/new-relic.yml@main
        secrets:
            NEW_RELIC_API_KEY: ${{ secrets.NEW_RELIC_API_KEY }}
            NEW_RELIC_DEPLOYMENT_ENTITY_GUID: ${{ secrets.NEW_RELIC_DEPLOYMENT_ENTITY_GUID }}
            NEW_RELIC_TEST1_ENTITY_GUID: ${{ secrets.NEW_RELIC_TEST1_ENTITY_GUID }}
            NEW_RELIC_TEST2_ENTITY_GUID: ${{ secrets.NEW_RELIC_TEST2_ENTITY_GUID }}
            NEW_RELIC_TEST3_ENTITY_GUID: ${{ secrets.NEW_RELIC_TEST3_ENTITY_GUID }}
            NEW_RELIC_TEST4_ENTITY_GUID: ${{ secrets.NEW_RELIC_TEST4_ENTITY_GUID }}
            NEW_RELIC_DEVELOP_ENTITY_GUID: ${{ secrets.NEW_RELIC_DEVELOP_ENTITY_GUID }}
            NEW_RELIC_PREPROD_ENTITY_GUID: ${{ secrets.NEW_RELIC_PREPROD_ENTITY_GUID }}

    do-reset:
        name: Trigger Reset Workflow
        needs: [ do-built-deploy ]
        uses: ANMedia/wp-vip-gh-workflows/.github/workflows/reset.yml@main
```

## Workflows

Summary descriptions of the resuable workflows.

### Deploy to Built

Build the application with Composer and NPM.
Push to the WordPress VIP deployment branch `-built` using the WP VIP-hosted script.

### Lint

Run the PHP and JS linters. Requires commands like:

`composer run lint:php`
`composer run phpstan`
`npm run lint:js`

@TODO normalize on a runner or use language-specific runners?

### PHPUnit

Execute PHPUnit tests via `composer run phpunit`

### Release

- Create a git tag for the current sha.
- Build and commit a changelog based on the current and last tag.
- Promote this tag to a release.
- Set a deployment marker in New Relic.

### New Relic

- Sets a deployment marker in New Relic for the given branch.
- Markers are required for the `production` branch. The `NEW_RELIC_API_KEY` and `NEW_RELIC_DEPLOYMENT_ENTITY_GUID` secrets must be set in the repository for this branch.
- Markers are also supported for the `test$N` and `develop` and `preprod` branches, but you need to set the corresponding New Relic entity GUIDs as secrets in the repository. If you don't, the deployment marker step will be skipped for those branches.

### Reset

If a `test$N` or `develop` branch, reset to the HEAD of the `production` branch
If a production release, reset all `test$N` and `develop` branches to `production` HEAD.
