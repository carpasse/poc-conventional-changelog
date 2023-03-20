# POC-conventional-commits

\***\*Automate** CHANGELOG generation, GitHub releases, and version bumps for your projects with conventional commits.\*\*

### :crystal_ball: About

Using [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) format can help you easily automate CHANGELOG generation, version bumps, and GitHub releases. This POC provides a sample setup to introduce conventional commits and some of the tooling around them.

## Conventional Commit Types

Conventional Commits only require you to use `fix:` and `feat:` types but other types are allowed. The recommended ones, according to [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional) (based on the [Angular convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)), are `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, and others.

Please note that commits with types other than `fix:`, `feat:`, `perf:` and `revert` will not generate a new version or an entry on the changelog [by default](https://github.com/conventional-changelog/conventional-changelog/blob/master/packages/conventional-changelog-conventionalcommits/writer-opts.js#L187), at least on JavaScript projects. If you want to change this behavior, you will need to overwrite the default settings of your changelog generator tool.

## Commit Message Recommendations

When you write a commit, do not describe your code changes. Instead, show the intent and explain the reason behind the change. For example:

```sh
feat(gallery): set opacity to 0.5 on, and cursor pointer on hover.
```

You should instead write the intention of the change:

```sh
feat(gallery): Hint users that gallery images are clickable.
```

In a commit, **showing the intent and the reason for the change is more important than the actual change**. Users can always go to the commit itself to see how the changes were made.

Hint: Every time you write a commit, imagine that you are adding a line to the changelog. This should be your mentality.

There are always exceptions to every rule. But they are usually on commits that don't make it into your changelog (`chore`, `perf`, `style`) and in most cases, it is also possible to show the intent. For example:

```
chore: Upgrade jest to version 28.x.x
```

could be rewritten to:

```
chore: Keep testing library up to date

Current version is 28.x.x.
```

or

```
chore: Setup project references to improve build performance.
```

could be rewritten to:

```
chore: Improve TS build times.

Setup project references to only build packages that change in the monorepo.
```

##### Package Manager Disclaimer

This POC has been created using [npm](https://www.npmjs.com/) as the package manager. The mentioned tools do work with other package managers like yarn and pnpm, but to install the tools with them, please refer to the official documentation.

## Enforcing Conventional Commits Convention with `commitlint`

Not everybody will want to use commitizen to create their commits. Therefore, we need to add some validation to prevent invalid pushes to the git server. To do so, we will use [`commitlint`](https://commitlint.js.org/#/).

##### Installation and Setup

```sh
npm install --save-dev @commitlint/config-conventional @commitlint/cli

# Configure commitlint to use conventional config
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js
```

To run the linter on each commit we will use husky's commit-msg hook

```sh
npm install husky --save-dev
npx husky install
npm pkg set scripts.prepare="husky install"
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

Now, if you try to commit an invalid message you get a validation error :-D

![commitlint validation error](/docs/assets/commit-lint-validation-error.png "commitlint validation error")

## Tools to Easily Create Conventional Commits - `commitizen`

To follow [Conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) convention can become a tedious task and may be the cause of dislike or rejection by some of you fellow dev team members. Lucky for us there are tools we can use to ease the creation of your conventional commits. My favorite is [`commitizen`](https://www.npmjs.com/package/commitizen).

### Commitizen

##### Installation and setup

```sh
npm install --save-dev commitizen
npx commitizen init cz-conventional-changelog --save-dev --save-exact
npm pkg set scripts.cz="cz"

```

If you don't want to add a pkg script you can also do:

```sh
npx cz
```

![commitizen prompt](/docs/assets/commitizen-prompt.png "commitizen prompt")

Note: there is a cz preset for [JIRA](https://www.npmjs.com/package/@digitalroute/cz-conventional-changelog-for-jira).

## Creating a Release Workflow with Changelogs and [Semver](https://semver.org/lang/es/) Version Bump

If our commits follow the `conventional-commits` convention, we can automatize the generation of a changelog with the changes and even bump the package version following the [semver](https://semver.org/lang/es/) convention.

To do so we are going to rely on 2 new cli tools [conventional-changelog-cli](conventional-changelog-cli) and [conventional-recommended-bump](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-recommended-bump#readme).

##### Installation and Setup

```sh
npm install --save-dev conventional-changelog-cli conventional-recommended-bump
```

and now lets prepare our package to release our changes with a single command.

```sh
npm pkg set scripts.changelog="npx conventional-changelog -p conventionalcommits -i CHANGELOG.md -s"
npm pkg set scripts.preversion="npm run lint"
npm pkg set scripts.version="npm run build && npm run changelog && git add ."
npm pkg set scripts.postversion="git push && git push --tags && npm publish"
npm pkg set scripts.release="npm version $(npx conventional-recommended-bump -p conventionalcommits)"
```

#### How to extend `conventional-commits` Preset to Match your Needs

You could create your own preset using conventional-commits preset as a start.

For example create a file named `changelog-preset.js` file in the root of the project and past the following config:

```js
"use strict";
const config = require("conventional-changelog-conventionalcommits");

module.exports = config({
  types: [
    { type: "feat", section: "Features" },
    { type: "feature", section: "Features" },
    { type: "fix", section: "Bug Fixes" },
    { type: "perf", section: "Performance Improvements" },
    { type: "revert", section: "Reverts" },
    { type: "docs", section: "Documentation", hidden: false },
    { type: "style", section: "Styles", hidden: true },
    { type: "chore", section: "Miscellaneous Chores", hidden: true },
    { type: "refactor", section: "Code Refactoring", hidden: false },
    { type: "test", section: "Tests", hidden: true },
    { type: "build", section: "Build System", hidden: true },
    { type: "ci", section: "Continuous Integration", hidden: true },
  ],
  issuePrefixes: ["TEST-"],
  issueUrlFormat: "https://myBugTracker.com/{{prefix}}{{id}}",
});
```

We can now override our `changelog` script to use our custom preset:

```sh
npm pkg set scripts.changelog="npx conventional-changelog -p $(pwd)/changelog-preset.js -i CHANGELOG.md -s"
```

**IMPORTANT**: Notice the pwd concatenation to generate an absolute path. `conventional-changelog` only accepts absolute paths or package names with the `conventional-changelog-` [prefix](https://github.com/conventional-changelog/conventional-changelog/blob/master/packages/conventional-changelog-preset-loader/index.js#L40) for example `conventional-changelog-one-beyond` and use it in all our projects passing the preset `-p one-beyond`.

The release workflow we've added is the following:

1. Validate our project (preversion script);
2. Build the project(version script)
3. Generate/update our CHANGELOG.md file (version script)
4. Bump the package version, create a a tag and aversion commit with version bump, build files and CHANGELOG.MD changes (release script)
5. Push the changes and the version tag (postversion script)
6. Publish the package into our private verdaccio registry (postversion script)

#### ~~Publish your release into Github~~

I wanted to use [conventional-github-releaser](https://github.com/conventional-changelog/releaser-tools/tree/master/packages/conventional-github-releaser#conventional-github-releaser) to create a github release but unfortunately the repo is not being actively maintain or at least doesn't look like it. And github rejects the release because the access token is passed via the Authorization header.

![conventional-github-releaser error](/docs/assets/conventional-github-releaser_error.png "conventional-github-releaser error")

If they ever fix the issue to use the tool is super easy.

##### Installation and Setup

```sh
npm install --save-dev conventional-github-releaser
npm pkg set scripts.postrelease="conventional-github-releaser -p conventionalcommits -r 0"
```

## `standard-version`

[Standard-version](https://github.com/conventional-changelog/standard-version) provides a way to automatizes the above steps without the need to understand each step of the process. Unfortunately it is deprecated. And you should probably avoid using it.

## `release-please` Manual Release Workflows - For github users only

From [release-please official documentation](https://github.com/googleapis/release-please#release-please)

> `release-please` automates CHANGELOG generation, semver version bumps and release publishing to GitHub.
> It does so by parsing your git history, looking for Conventional Commit messages, and creating release PRs.
> It does not handle publication to package managers or handle complex branch management.

There are 2 possible workflows with creating [github action](https://github.com/google-github-actions/release-please-action) (recommended way) and with their [CLI](https://github.com/googleapis/release-please/blob/main/docs/cli.md)

Before starting, you need to [create a new github token](https://github.com/settings/tokens/new)

### release-please CLI Workflow

##### Installation and Setup

```sh
npm install --save-dev release-please
npx release-please bootstrap --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits --release-type=node
```

This will create a github PR with the config files:

![release-please bootstrap PR](/docs/assets/release-please_bootstrap_pr.png "release-please bootstrap PR")

##### Create a Release PR

```sh
 npx release-please release-pr --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits
```

**Note** if no release pr gets created, chances are you don't have feat/fix commits or breaking changes in your commit history.

#### Create a Github Release

Once you've merged the PR you can create a github release with the following command

```sh
npx release-please github-release --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits
```

### `release-please` Github Action Workflow

Go to [github action](https://github.com/google-github-actions/release-please-action)

## Semantic Release

[semantic-release](https://semantic-release.gitbook.io/semantic-release/) automates the whole package release workflow including: determining the next version number, generating the release notes, and publishing the package.
