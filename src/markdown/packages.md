---
path: "/guides/packages"
title: "How the Nulogy Design Ops team releases packages"
---

When we set out to design our release solution, we had several goals in mind:

- Follow an existing standard
- Create and release packages based on that standard without human intervention
- Provide our developers access to code as fast as possible

To accomplish these goals, we used:

- Semver and Conventional Commits
- semantic-release
- Github Actions

### Conventional Commits

[Conventional Commits](https://www.conventionalcommits.org/) is a specification for adding human and machine readable meaning to commit messages. For example, instead of writing `added a Datepicker component`, following Conventional Commits you’d write `feat: added a Datepicker component`. A commit can be designated a fix, feature, and/or a breaking change, which will correlate with Semver's `patch`, `minor`, and `major` changes. This allows tooling to be built to automatically version and release packages.

### semantic-release

The tooling we’re using is called [semantic-release](https://github.com/semantic-release/semantic-release), which “automates the whole package release workflow including:

- determining the next version number,
- generating the release notes,
- and publishing the package.”

After ensuring commit messages are written with Conventional Commits, it’s possible to automatically package and release software through Continous Integration on every merge to master. This ensures your consumers are getting access to changes literally as fast as possible.

To set this up:

#### Add semantic-release

```
yarn add semantic-release --dev
```

_Since semantic-release uses git tags and npm to decide on the package version, you'll want to set the `version` in `package.json` to something like `0.0.0` to discourage manually updating it._

#### Add plugins

semantic-release can do a lot of things, so it's made up of opt-in plugins. We'd like it to do everything, so we'll use the following plugins, in order:

- [@semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer) to look through the commit message and decide what kind of change is being added.
- [@semantic-release/release-notes-generator](https://github.com/semantic-release/release-notes-generator) to generate release notes based on the commit message.
- [@semantic-release/changelog](https://github.com/semantic-release/changelog) to update a persistant `CHANGELOG.md` file using those release notes.
- [@semantic-release/git](https://github.com/semantic-release/git) to commit the changelog to the git repository.
- [@semantic-release/github](https://github.com/semantic-release/github) to tag and publish a Github release.
- [@semantic-release/npm](https://github.com/semantic-release/npm) to publish the package to npm.

These can all be added with:

```
yarn add--dev @semantic-release/changelog @semantic-release/commit-analyzer
@semantic-release/changelog @semantic-release/git @semantic-release/github
@semantic-release/npm
```

Then tell semantic-release to use them, by adding a `.releaserc.json` to the root directory with the following:

```{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/git",
    "@semantic-release/github",
    "@semantic-release/npm"
  ]
}
```

Now that it's intsalled, we can run it on CI after our build.

### Github Actions

[Github Actions](https://github.com/features/actions) is Github’s free Continous Integration service and will allow us to run semantic-release automatically on merge to master.

#### Add secret key

Generate an NPM secret key from your NPM account settings. Then go to the Settings > Secrets page in Github and add a token called `NPM_TOKEN`.

#### Add workflow

Add a file called `/.github/workflows/release.yml` with the following, replacing the command in the Build step with your relevant build command:

```
name: Release
on:
  push:
    branches:
      - master
jobs:
  release:
    name: Release
    runs-on: ubuntu-18.04
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 12
      - name: Install dependencies
        run: yarn
      - name: Build
        run: yarn build
      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

That's it. Automated releases with changelogs published on every merge to master with no human intervention.

A test repo of this code can be found at [https://github.com/nulogy/nds-test-releases](https://github.com/nulogy/nds-test-releases).
