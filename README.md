# Release It!

CLI release tool for Git repos and npm packages.

![screencast](./release-it.gif)

Release-it automates the tedious tasks of software releases:

* Execute build commands
* Bump version (in e.g. `package.json`)
* Generate changelog
* Git commit, tag, push
* Create release at GitHub
* Upload assets to GitHub release
* Publish to npm
* Push build artefacts (bundles, docs) to a separate repository or branch.

## Usage

Release a new patch (increments from e.g. `1.0.4` to `1.0.5`):

```bash
release-it
```

Release a patch, minor, major, or specific version:

```bash
release-it minor
release-it 0.8.3
```

Create a pre-release using `prerelease`, `prepatch`, `preminor`, or `premajor`:

```bash
release-it premajor --prereleaseId="beta"
release-it premajor
```

The first example would increment from e.g. `1.0.6` to `2.0.0-beta.0`, the second from `2.0.0-beta.0` to `2.0.0-beta.1`.

See [node-semver](https://github.com/npm/node-semver#readme) for more details.

You can also do a "dry run", which won't write/touch anything, but does output the commands it would execute, and show the interactivity:

```bash
release-it --dry-run
```

## Configuration

Out of the box, release-it can do a lot, but has plenty of options to configure it.

All [default settings](conf/release-it.json) can be overridden with a config file. Put a `.release-it.json` file in the project root, and it will be picked up. You can use `--config` if you want to use another path for this file.

Any option can also be set on the command-line, and will have highest priority. Example:

```bash
release-it minor --src.tagName='v%s' --github.release
```

Boolean arguments (e.g. `--github.release`) can be negated by using the `no-` prefix:

```bash
release-it --no-npm.publish
```

## Interactive vs. non-interactive mode

By default, release-it is interactive and allows you to confirm each task before execution.

![interactive](./release-it.png)

Once you are confident release-it does the right thing, you can fully automate it by using the `--non-interactive` (or `-n`) option (as demonstrated in the animated image above). An overview of the tasks that will be executed:

Task | Option | Default | Prompt | Default
:--|:--|:-:|:--|:-:
Show staged files | - | | `prompt.src.status` | `N`
Git commit | `src.commit` | `true` | `prompt.src.commit` | `Y`
Git push | `src.push` | `true` | `prompt.src.push` | `Y`
Git tag | `src.tag` | `true` | `prompt.src.tag` | `Y`
GitHub release | `github.release` | `true` | `prompt.src.release` | `Y`
npm publish | `npm.publish` | `true` | `prompt.src.publish` | `Y`

Note that the `prompt.*` options are used for the default answers in interactive mode. You can still change the answer to either `Y` or `N` as the questions show up.

## Command Hooks

The command hooks are executed from the root directory of the `src` or `dist` repository, respectively:

* `src.beforeStartCommand`
* `buildCommand` - before files are staged for commit
* `src.afterReleaseCommand`
* `dist.beforeStageCommand` - before files are staged in dist repo
* `dist.afterReleaseCommand`

All commands can use configuration variables (like template strings):

```bash
"buildCommand": "tar -czvf foo-${src.tagName}.tar.gz ",
"afterReleaseCommand": "echo Successfully released ${version} to ${dist.repo}."
```

## SSH keys & git remotes

The tool assumes you've configured your GitHub SSH key and Git remotes correctly. In short: you're fine if you can `git push`. Otherwise, the following GitHub help pages might be useful: [SSH](https://help.github.com/articles/connecting-to-github-with-ssh/) and [Managing Remotes](https://help.github.com/categories/managing-remotes/).

## GitHub Release

See this project's [releases page](https://github.com/webpro/release-it/releases) for an example.

To create [GitHub releases](https://help.github.com/articles/creating-releases/):

* The `github.release` option must be `true`.
* Obtain a [GitHub access token](https://github.com/settings/tokens).
* Make this available as the environment variable defined with `github.tokenRef`. Example:

```bash
export GITHUB_TOKEN="f941e0..."
```

### Release Assets

To upload binary release assets with a GitHub release (such as compiled executables,
minified scripts, documentation), provide a [glob pattern](https://github.com/isaacs/node-glob#readme) for the `github.assets` option. After the release, the assets are available to download from the GitHub release page. Example:

```json
"github": {
  "release": true,
  "assets": "dist/*.zip"
}
```

## Distribution Repository

Some projects use a distribution repository. Reasons to do this include:

* Distribute to target specific package managers.
* Distribute to a Github Pages branch (also see [Using GitHub Pages, the easy way](https://medium.com/@webprolific/using-github-pages-the-easy-way-bb7acc46f45b)).

Overall, it comes down to a need to release generated files (such as compiled bundles, documentation) into a separate repository. Some examples include:

* Projects like Ember, Modernizr and Bootstrap are in separate [shim repositories](https://github.com/components).
* AngularJS maintains a separate [packaged angular repository](https://github.com/angular/bower-angular) for distribution on npm and Bower.

To use this feature, set the `dist.repo` option to a git endpoint. This can be a branch (also of the same source repository), like `"git@github.com:webpro/release-it.git#gh-pages"`. Example:

```json
"dist": {
  "repo": "git@github.com:components/ember.git"
}
```

The repository will be cloned to `dist.stageDir`, and the `dist.files` (relative to `dist.baseDir`) will be copied from the source repository. The files will then be staged, commited and pushed back to the remote distribution repository.

Make sure to set `dist.github.release` and `dist.npm.publish` to `true` as needed. The `dist.github.*` options will use the `github.*` values as defaults. Idem dito for `dist.npm.*` options, using `npm.*` for default values.

During the release of a source and distribution repository, some "dist" tasks are executed before something is committed to the source repo. This is to make sure you find out about errors (e.g. while cloning or copying files) as soon as possible, and not after a release for the source repository first.

## Notes

* The `"private": true` setting in package.json will be respected and the package won't be published to npm.
* You can use `src.pushRepo` option to set an alternative url or name of a remote as in `git push <src.pushRepo>`. By default this is `null` and  `git push` is used when pushing to the remote.

## Credits

Major dependencies:

* [ShellJS](http://documentup.com/shelljs/shelljs)
* [Inquirer.js](https://github.com/SBoudrias/Inquirer.js)
* [node-github](https://github.com/mikedeboer/node-github)

The following Grunt plugins have been a source of inspiration:

* [grunt-release](https://github.com/geddski/grunt-release)
* [grunt-release-component](https://github.com/walmartlabs/grunt-release-component)

## Why YA...

Why did I need to create yet another "release" tool/plugin? I think this tool stands out:

* As a user-friendly, stand-alone CLI tool.
* Making it simple to release the current project you're working at.
* Working without any configuration, but also provides many options.
* Creating a GitHub release, and uploading assets with it.
* Releasing a separate distribution repository (in a single run).
* Being as quiet or verbose as you want it to be.

## License

[MIT](http://webpro.mit-license.org/)
