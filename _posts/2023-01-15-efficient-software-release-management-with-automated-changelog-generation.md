---
layout: post
title: "Efficient Software Release Management with Automated Changelog Generation"
excerpt: "In this article, we'll explore how automated changelog generation, commitlint and git hooks can help optimize the software release process."
tags: [software release process, automated changelog generation, commitlint, git hooks, release, conventional commits, release-it]
date: 2023-01-15T15:22:08+01:00
comments: true
image:
  feature: posts/release-process/cover-release-process.jpg
  credit: PngTree
  creditlink: https://pngtree.com/free-backgrounds
---

The software release process is a critical aspect of any software development project. A manual software release process can be time-consuming, error-prone, and often leaves room for human error. This is where an automated release process comes in. The tools mentioned in this article (used in TypeScript/Javascript projects) can help streamline and optimize the software release process, making it more efficient and less prone to errors, by reducing human involvement.

## In this article

1. [Versioning](#versioning)
2. [A convention to use in all commits](#a-convention-to-use-in-all-commits)
3. [Making sure that the Conventional Commit rules are followed](#making-sure-that-the-conventional-commit-rules-are-followed)
4. [Local Git hooks](#local-git-hooks)
    - [Configuring](#configuring)
5. [Way of working](#way-of-working)
    - [Branch naming conventions](#branch-naming-conventions)
    - [Pull Request title](#pull-request-title)
    - [Making sure the Pull Request title format is followed](#making-sure-the-pull-request-title-format-is-followed)
    - [Pull Request content](#pull-request-content)
    - [Merging strategy](#merging-strategy)
    - [Status checks on Pull Requests](#status-checks-on-pull-requests)

## Versioning

When thinking about a software release, it all starts with deciding how the versioning will be handled. I've been following [Semantic Versioning (semver)](https://semver.org/) in my projects for years now and recommend it.

This means that:

```
Given a version number MAJOR.MINOR.PATCH, increment the:

1. MAJOR version when you make incompatible API changes
2. MINOR version when you add functionality in a backwards compatible manner
3. PATCH version when you make backwards compatible bug fixes
Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.
```

Ps: There's also a variation of it for purely [frontend repositories](https://github.com/AshCoolman/semantic-frontend-versioning).

## A convention to use in all commits

In most projects, if the team wants to keep some kind of changelog, it's necessary to write the latest changes somewhere manually (Confluence, Notion, a CHANGELOG.md file, Slack, etc).
We want to automate that and remove the manual steps required in order to create a release, but we'll get to this part on how to automate the changelog generation in a bit.
There's one step which is a requirement for us to get there:

The [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification provides a group of rules on how to format every commit message so that they all follow the same pattern across the project. It is used by the [Angular](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.t7ifoyph8bd3) team and many other big projects.

Example of a commit message: ```<type>[optional scope]: <description>```

A commit contains the following structural elements, to communicate intent:

1. fix: a commit of the type fix patches a bug in your codebase (this correlates with **PATCH** in Semantic Versioning).
2. feat: a commit of the type feat introduces a new feature to the codebase (this correlates with **MINOR** in Semantic Versioning).
3. **BREAKING CHANGE**: a commit that has a footer ```BREAKING CHANGE:```, or appends a ```!``` after the type/scope, introduces a breaking API change (correlating with **MAJOR** in Semantic Versioning). A BREAKING CHANGE can be part of commits of any type.
4. types other than ```fix:``` and ```feat:``` are allowed, for example [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional) (based on the [Angular convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)) recommends ```build:```, ```chore:```, ```ci:```, ```docs:```, ```style:```, ```refactor:```, ```perf:```, ```test:```, and others.
5. footers other than ```BREAKING CHANGE: <description>``` may be provided and follow a convention similar to [git trailer format](https://git-scm.com/docs/git-interpret-trailers).

Additional types are not mandated by the Conventional Commits specification, and have no implicit effect in Semantic Versioning (unless they include a BREAKING CHANGE). A scope may be provided to a commit's type, to provide additional contextual information and is contained within parenthesis, e.g.,
```feat(parser): add ability to parse arrays.```

## Making sure that the Conventional Commit rules are followed

In order to enforce that the correct format is followed, the commitlint library can be automatically called via **git hooks** locally when the developer tries to create a new commit. It works as a linter, but for commit messages!

Install the following packages:

- [@commitlint/cli](https://www.npmjs.com/package/@commitlint/cli)
- [@commitlint/config-conventional](https://www.npmjs.com/package/@commitlint/config-conventional)

Then add a file called ```commitlint.config.js``` to the root of your project, with the following content:

```javascript
module.exports = { extends: ['@commitlint/config-conventional'] };
```

## Local Git hooks

Git hooks are scripts that run automatically before or after executing Git commands like Commit and Push. With Git hook scripts, users can customize Git's internal behavior by automating specific actions at the level of programs and deployment, like for example validating that the commit is following the standards and not breaking the tests.

They can be managed with the use of a library called [husky](https://www.npmjs.com/package/husky).

The hooks I usually configure are: pre-commit, commit-msg and pre-push

- **pre-commit**: When the developer tries to commit what was implemented, the pre-commit hook will be automatically called and run eslint with the predefined rules. If the code is invalidated by these rules, the commit doesn't happen until the issues are fixed. It's better to have the linter at commit time to allow for smaller and timely iterations instead of only checking these during the push to the repository
- **commit-msg**: When the developer writes the commit message, it is automatically checked against commitlint, which validates the commit message against the Conventional Commits specification. If the commit message is not valid, the commit doesn't happen until the message is fixed.
- **pre-push**: When the developer tries to push the commit(s) to the repository, the pre-push hook is called automatically and runs "npm test". The push to the repository only happens if all tests are passing.

### Configuring

Install the husky library as a dev dependency:

```bash
npm i husky --save-dev
```

Run the following to add the following to the package.json file:

```bash
npm pkg set scripts.prepare="husky install"
npm run prepare
```

Add the pre-commit hook that runs the linter:

```bash
npx husky add .husky/pre-commit "npm run lint"
```

Ps: My eslintrc example can be found [here](https://gist.github.com/jonathas/c6b5f110e1eaf92d94ac976a19a3a178).

Add the commit-msg hook that runs commitlint:

```bash
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```

Then finally the pre-push hook that runs the tests:

```bash
npx husky add .husky/pre-push "npm test"
```

After these steps, commit the .husky directory to the git repository.

## Way of working

For the automated changelog generation to work well, it's important to have a well defined development process.

When a developer starts to work on this ticket, a new branch needs to be created in the git repository. This branch must be created out of the **develop branch**.

### Branch naming conventions

The name of this new branch that the developer will create can be anything. It doesn't really matter in this case, as it must be automatically deleted after the PR is merged anyway. What I recommend, though, is to follow a similar format to what is done in the commits, so that it's easy to identify what that branch is about.

For example, if it's a new feature:

```feat/name-of-the-feature```

if it's a fix:

```fix/name-of-the-fix```

### Pull Request title

When the developer is ready to submit changes to the repository, a new Pull Request needs to be created.
The Pull Request title, matters more than the name of the branch, since that's what will end up in our git history and, consequently in our automatically generated changelog! Hence, when creating a Pull Request, its name must follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0) specification.

An example of how the PRs following Conventional Commits would look like in the repository history:

![Repository history](/images/posts/release-process/repo-history.png "Repository history")

If you'd like to also link the PRs to Jira tickets, it's recommended to add the Jira ticket to the PR title as well, following this format:
```<type>(scope): description [jiraticket-number]```

For example:
```fix(calendar): default pagination limit [AG-1103]```

AG-1103 in this case being the Jira ticket.

Ps: You'll need to, of course, configure this sync between Github and Jira separately. This part is not covered by this article.

### Making sure the Pull Request title format is followed

Using Github Actions, it's easy to implement a workflow that checks the Pull Request title and validates it against the Conventional Commits specification.

File: **.github/workflows/lint-pr-title.yml**

```yaml
name: "Lint PR Title"
on:
  pull_request_target:
    types:
      - opened
      - edited
      - synchronize

jobs:
  lint-pr-title:
    if: ${{ github.actor != 'dependabot[bot]' }}
    runs-on: ubuntu-latest
    steps:
      - uses: dreampulse/action-lint-pull-request-title@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Whenever a PR is created, this workflow runs and doesn't allow the PR to be merged if the title is incorrect.

Imagine dependabot creates 20 PRs and all of them run your Github Actions workflows. That would spend a lot of precious Github Actions minutes, which cost money!
So as a bonus, this workflow doesn't run on PRs created automatically by [dependabot](https://github.blog/2020-06-01-keep-all-your-packages-up-to-date-with-dependabot/), which should already be following the Conventional Commits specification.

### Pull Request content

A good pull request should allow developers to review it quickly, so it needs to be small and well explained.
The best way of thinking about it to keep it smaller is that it should cover one thing only, as in the Single Responsibility Principle (SRP).

![PR size](/images/posts/release-process/pr-size.png "PR size")

Here's an interesting and worth reading article I found the other day that gets deeper into this topic: [The anatomy of a perfect pull request](https://hugooodias.medium.com/the-anatomy-of-a-perfect-pull-request-567382bb6067)

### Merging strategy

In order for the changelog generation process to work correctly, the git history needs to be clean, so there must be no merge commits in the develop nor in the master/main branches.
The repository needs to be configured to only allow squash merging, so that only PRs end up in the git history.
In Github that can be done in the following area:

![Github config](/images/posts/release-process/github-squash-merge-config.png "Github config")

And then the developers can configure git locally to always use rebase so that they won't need to create merge commits when fixing conflicts in PRs.
This can be done globally for all projects by running the following command:

```bash
git config --global pull.rebase true
```

Then when synching the changes from the develop branch to the branch they're working on, they can just pull the latest changes from develop into their branch and the rebase will happen automatically:

```bash
git checkout feat/my-feature-branch
git pull origin develop
```

### Status checks on Pull Requests

As with the Github Action workflow to validate the PR title, it's recommended to run other status checks on PRs to validate that the PR can only be merged to the develop branch if all the status checks are passing.
These are recommended to be added to the repository:

- Run integration tests
- Run unit tests
- SonarCloud Code Analysis

I won't go deeper in details about them as it's not the focus of this article.

## Conclusion

In conclusion, the software release process is a critical aspect of software development, but it can be time-consuming, error-prone and often leaves room for human error. Automated changelog generation, commitlint, and git hooks are tools that can help streamline and optimize the software release process, making it more efficient and less prone to errors. By using these tools, teams can improve the software release process and deliver high-quality software faster and with fewer mistakes. If you want to optimize your software release process, consider implementing automated changelog generation, commitlint and git hooks. These tools can help you improve the efficiency, accuracy and quality of your software release process.
