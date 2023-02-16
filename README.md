# POC-conventional-changelog

**Automate CHANGELOG generation, the creation of GitHub releases, and version bumps for your projects.**

### :crystal_ball: About

Doing your commits in [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) format can easly help you automatize CHANGELOG generation, version bumps and GitHub releases. In this POC we propose a setup to easily generate conventional commits, validate them and to automatize releases.

#### Package manager disclaimer

This POC has been created using npm as the package manager. The mentioned tools do work with other package manager like yarn and pnpm but to install the tools with them, please refer to the official documentation.

### How to enforce conventional commits convention - commitlint

Not everybody will want to use commitizen to create their commits therefore we need to add some validation to prevent invalid pushes to the git server. To do so we are going to use [commitlint](https://commitlint.js.org/#/).

#### Installation

```sh
# Install commitlint cli and conventional config
npm install --save-dev @commitlint/{config-conventional,cli}
# For Windows:
npm install --save-dev @commitlint/config-conventional @commitlint/cli

# Configure commitlint to use conventional config
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

To run the linter on each commit we will use husky's commit-msg hook

```sh
npm install husky --save-dev
npm pkg set scripts.infra:husky="husky install"
npm run infra:husky
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

Now, if you try to commit an invalid message you get a validation error :-D

![commitlint validation error](/docs/assets/commitizen-prompt.png "commitlint validation error")
