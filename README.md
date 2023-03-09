# POC-conventional-commits

**Automate CHANGELOG generation, the creation of GitHub releases, and version bumps for your projects.**

### :crystal_ball: About

Doing your commits in [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) format can easily help you automatize CHANGELOG generation, version bumps and GitHub releases. In this POC we propose a setup to easily generate conventional commits, validate them and to automatize releases.

## Conventional commits types

Conventional Commits only require you to use `fix:` and `feat:` types but other types are allowed. The typical ones are [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional) (based on the [Angular convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)) recommends `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, and others.

Beware that commits with types other than `fix:`, `feat:`, `perf:` and `revert` won't generate a new version nor an entry on the changelogs [by default](https://github.com/conventional-changelog/conventional-changelog/blob/master/packages/conventional-changelog-conventionalcommits/writer-opts.js#L187) at least on JS projects. If you really wan't to change this behavior you will need to overwrite the default settings of your changelog generator tool.

## Commit messages recommendations  

When you write a commit do not resume your code changes. Instead show the intent. For example:

```
feat(gallery): set opacity to 0.5 on, and cursor pointer on hover.
```

You should instead write the intention of the change:

```
feat(gallery): Hint users that gallery images are clickable.
```

In a commit, **to show the intent and the reason of the change** is more important that the actual change. User can always go to the commit itself to see the changes.

Hint: Every time you write a commit image that what you are doing is adding a line to the changelog. That needs to be the mentality.

There are always exceptions to every rule. But they are usually on commits that don't make it into your changelog and in most of the case it is better to show intent. For example:

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
perf: Setup project references to improve build performance.
```

could be rewritten to:

```
perf: Improve TS build times.

Setup project references to only build packages that change in the monorepo
```

##### Package manager disclaimer

This POC has been created using [npm](https://www.npmjs.com/) as the package manager. The mentioned tools do work with other package manager like yarn and pnpm but to install the tools with them, please refer to the official documentation.

## Enforcing conventional commits convention with commitlint

Not everybody will want to use commitizen to create their commits therefore we need to add some validation to prevent invalid pushes to the git server. To do so we are going to use [commitlint](https://commitlint.js.org/#/).

##### Installation and setup

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

## Tools to easily create conventional commits - commitizen

To ensure that our commits follow [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) convention can become a tedious task and may be the cause of dislike/rejection of some of you fellow dev team members. Lucky for us there are 2 tools we can use to ease the creation of your conventional commits.

### Commitizen

One of the tools we can use is commitizen

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

## Creating a release workflow with Changelogs and semver version bump

Once we are ready to release our changes we will want to generate a changelog with changes and to bump our package version number. With `conventional-commits` we can automatize this process.

To do so we are going to rely on 2 new cli tools [conventional-changelog-cli](conventional-changelog-cli) and [conventional-recommended-bump](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-recommended-bump#readme).

##### Installation and setup

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

#### You can extend conventional-commits preset to match your needs

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
npm pkg set scripts.changelog="npx conventional-changelog -p changelog-preset.js -i CHANGELOG.md -s"
```

If we want, we can publish our custom preset as npm package with the [prefix](https://github.com/conventional-changelog/conventional-changelog/blob/master/packages/conventional-changelog-preset-loader/index.js#L40) `conventional-changelog-` for example `conventional-changelog-one-beyond` and use it in all our projects passing the preset `-p one-beyond`.

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

##### Installation and setup

```sh
npm install --save-dev conventional-github-releaser
npm pkg set scripts.postrelease="conventional-github-releaser -p conventionalcommits -r 0"
```

## standard-version

[Standard-version](https://github.com/conventional-changelog/standard-version) provides a way to automatizes the above steps without the need to understand each step of the process. Unfortunately it is deprecated. And you should probably avoid using it.

## release-please release workflows - For github users only

From [release-please official documentation](https://github.com/googleapis/release-please#release-please)

> Release Please automates CHANGELOG generation, the creation of GitHub releases, and version bumps for your projects.
> It does so by parsing your git history, looking for Conventional Commit messages, and creating release PRs.
> It does not handle publication to package managers or handle complex branch management.

There are 2 possible workflows with creating [github action](https://github.com/google-github-actions/release-please-action) (recommended way) and with their [CLI](https://github.com/googleapis/release-please/blob/main/docs/cli.md)

Before starting, you need to [create a new github token](https://github.com/settings/tokens/new)

### release-please cli workflow

##### Installation and setup

```sh
npm install --save-dev release-please
npx release-please bootstrap --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits --release-type=node
```

This will create a github PR with the config files:

![release-please bootstrap PR](/docs/assets/release-please_bootstrap_pr.png "release-please bootstrap PR")

##### Create a release-pr

```sh
 npx release-please release-pr --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits
```

**Note** if no release pr gets created, chances are you don't have feat/fix commits or breaking changes in your commit history.

#### Create a Github release

Once you've merged the PR you can create a github release with the following command

```sh
npx release-please github-release --token=$GITHUB_TOKEN --repo-url=carpasse/poc-conventional-commits
```

### release-please github action workflow

Go to [github action](https://github.com/google-github-actions/release-please-action)

## Semantic release

[semantic-release](https://semantic-release.gitbook.io/semantic-release/) automates the whole package release workflow including: determining the next version number, generating the release notes, and publishing the package.
