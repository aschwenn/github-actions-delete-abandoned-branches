# delete-abandoned-branches

Github action to delete abandoned branches.

## Warning

This action WILL delete branches from your repository, so you need to make your due diligence when choosing to use it and with which settings. I am not responsible for any mishaps that might occur.

## Abandoned branches

A branch must meet all the following criteria to be deemed abandoned and safe to delete:

* Must NOT be the default branch (eg `master` or `main`, depending on your repository settings)
* Must NOT be a protected branch
* Must NOT have any open pull requests
* Must NOT be the base of an open pull request of another branch. The base of a pull request is the branch you told GitHub you want to merge your pull request into.
* Must NOT be in an optional list of branches to ignore
* Must be older than a given amount of days

## Inputs

* mandatory

| Name  | Description | Example |
| ------------- | ------------- | ------------- |
| `ignore_branches`  | Comma-separated list of branches to ignore and never delete. You don't need to add your protected branches here.  | `foo,bar`
| `last_commit_age_days` | How old in days must be the last commit into the branch for the branch to be deleted. Default: `60` | 90
| `dry_run` | Whether we're actually deleting branches at all. Possible values: `yes, no` (case sensitive). Default: `yes` | `no`
| `github_token` | The github token to use on requests to the github api. You can use the one github actions provide | `${{ github.token }}`

### Note: dry run

By default, the action will only perform a dry run. It will go in, gather all branches that qualify for deletion and give you the list on the actions' output, but without actually deleting anything. Make sure you configure your stuff correctly before setting `dry_run` to `yes`

## Example

The following workflow will run on a schedule (daily at 00:00) and will delete all abandoned branches older than 100 days:

```yaml
name: Delete abandoned branches

on:
  schedule:
  - cron: "0 * * * *"

  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
      tags:
        description: 'Test scenario tags'

jobs:
  check_dry_run_no_branches:
    runs-on: ubuntu-latest
    name: Runs the action with no ignore branches
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Delete those pesky dead branches
        uses: phpdocker-io/github-actions-delete-abandoned-branches
        id: delete_stuff
        with:
          github_token: ${{ github.token }}
          last_commit_age_days: 100
          dry_run: yes

      - name: Get output
        run: "echo 'Deleted branches: ${{ steps.delete_stuff.outputs.deleted_branches }}'"

```