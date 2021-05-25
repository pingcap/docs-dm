# TiDB Data Migration Documentation Contributing Guide

Welcome to [TiDB Data Migration](https://github.com/pingcap/tidb) documentation! We are excited about the prospect of you joining [TiDB Community](https://github.com/pingcap/community/).

## What you can contribute

You can start from any one of the following items to help improve [TiDB Data Migration Docs at the PingCAP website](https://docs.pingcap.com/tidb-data-migration/stable):

- Fix typos or format (punctuation, space, indentation, code block, etc.)
- Fix or update inappropriate or outdated descriptions
- Add missing content (sentence, paragraph, or a new document)
- Translate docs changes from Chinese to English, or from English to Chinese.
- Submit, reply to, and resolve [issues](https://github.com/pingcap/docs-dm/issues) in the docs-dm repo
- (Advanced) Review Pull Requests created by others

## Before you contribute

Before you contribute, please take a quick look at some general information about TiDB documentation maintenance. This can help you to become a contributor soon.

### Get familiar with style

- [Commit Message Style](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#how-to-write-a-good-commit-message)
- [Pull Request Title Style](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#pull-request-title-style)
- [Markdown Rules](/resources/markdownlint-rules.md)
- [Code Comment Style](https://github.com/pingcap/community/blob/master/contributors/code-comment-style.md)
- Diagram Style: [Figma Quick Start Guide](https://github.com/pingcap/community/blob/master/contributors/figma-quick-start-guide.md)

    To keep a consistent style for diagrams, we recommend using [Figma](https://www.figma.com/) to draw or design diagrams. If you need to draw a diagram, refer to the guide and use shapes or colors provided in the template.

### Learn about docs versions

Currently, we maintain three versions of TiDB Data Migration documentation, each with a separate branch:

| Branch name | Version description |
| :--- | :-- |
| `master` | the latest development version |
| `release-2.0` | the latest 2.0 stable version |
| `release-1.0` | the latest 1.0 stable version |

### Use cherry-pick labels

- If your changes apply to only one docs version, just submit a PR to the corresponding version branch.

- If your changes apply to multiple docs versions, you don't have to submit a PR to each branch. Instead, after you submit your PR, trigger the ti-chi-bot to submit a PR to other version branches by adding one or several of the following labels as needed. Once the current PR is merged, ti-chi-bot will start to work.
    - `needs-cherry-pick-master` label: ti-chi-bot will submit a PR to the `master` branch.
    - `needs-cherry-pick-release-2.0` label: ti-chi-bot will submit a PR to the `release-2.0` branch.
    - `needs-cherry-pick-release-1.0` label: ti-chi-bot will submit a PR to the `release-1.0` branch.

- If most of your changes apply to multiple docs versions but some differences exist among versions, you still can use cherry-pick labels to let ti-chi-bot create PRs to other versions. After the PR to another version is successfully submitted by ti-chi-bot, you can comment changes in that PR, and then the repo administrator will help to apply those comments.

## How to contribute

Please perform the following steps to create your Pull Request to this repository. If don't like to use commands, you can also use [GitHub Desktop](https://desktop.github.com/), which is easier to get started.

> **Note:**
>
> This section takes creating a PR to the `master` branch as an example. Steps of creating PRs to other branches are similar.

### Step 0: Sign the CLA

Your Pull Requests can only be merged after you sign the [Contributor License Agreement](https://cla-assistant.io/pingcap/docs) (CLA). Please make sure you sign the CLA before continuing.

### Step 1: Fork the repository

1. Visit the project: <https://github.com/pingcap/docs-dm>
2. Click the **Fork** button on the top right and wait it to finish.

### Step 2: Clone the forked repository to local storage

```
cd $working_dir # Comes to the directory that you want put the fork in, for example, "cd ~/Documents/GitHub"
git clone git@github.com:$user/docs-dm.git # Replace "$user" with your GitHub ID

cd $working_dir/docs-dm
git remote add upstream git@github.com:pingcap/docs-dm.git # Adds the upstream repo
git remote -v # Confirms that your remote makes sense
```

### Step 3: Create a new branch

1. Get your local master up-to-date with upstream/master.

    ```
    cd $working_dir/docs-dm
    git fetch upstream
    git checkout master
    git rebase upstream/master
    ```

2. Create a new branch based on the master branch.

    ```
    git checkout -b new-branch-name
    ```

### Step 4: Do something

Edit some file(s) on the `new-branch-name` branch and save your changes. You can use editors like Visual Studio Code to open and edit `.md` files.

### Step 5: Commit your changes

```
git status # Checks the local status
git add <file> ... # Adds the file(s) you want to commit. If you want to commit all changes, you can directly use `git add.`
git commit -m "commit-message: update the xx"
```

See [Commit Message Style](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#how-to-write-a-good-commit-message).

### Step 6: Keep your branch in sync with upstream/master

```
# While on your new branch
git fetch upstream
git rebase upstream/master
```

### Step 7: Push your changes to the remote

```
git push -u origin new-branch-name # "-u" is used to track the remote branch from origin
```

### Step 8: Create a pull request

1. Visit your fork at <https://github.com/$user/docs-dm> (replace `$user` with your GitHub ID)
2. Click the `Compare & pull request` button next to your `new-branch-name` branch to create your PR. See [Pull Request Title Style](https://github.com/pingcap/community/blob/master/contributors/commit-message-pr-style.md#pull-request-title-style).

Now, your PR is successfully submitted! After this PR is merged, you will automatically become a contributor to TiDB documentation.

## Contact

Join the Slack channel: [#sig-docs](https://slack.tidb.io/invite?team=tidb-community&channel=sig-docs&ref=pingcap-docs)
