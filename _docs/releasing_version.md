---
layout: post
title: Releasing a new version of a module
date: 2016-01-01
summary: How to perform a complete version release, including modulesync and publication.
---

Creating a release is a two step process:
1. Prepare the release — setup everything so that peer review can happen and when everything is ready…
2. Do the actual release

## Preparing a release

Run modulesync to ensure the dotfiles are up-to-date.

Checkout an updated copy of master

```bash
git checkout master; git fetch origin; git pull origin master
```

Create a 'release pr'.

```bash
git checkout -b release_1_2_3
```

This pull request updates the changelog and bumps the
version number to the target version, removing all release candidate
identifiers, i.e. from `0.10.7-rc0` to `0.10.7`. Here's an example:
[puppet-extlib's 0.10.7 release](https://github.com/voxpupuli/puppet-extlib/pull/43).
In most cases it is sufficient to update metadata.json. We try
to respect [semantic versioning](http://semver.org/) and decided that dropping ruby1.8
support is a major change and requires a major version bump for the module.
(Only the minor version should be bumped if the module is pre version 1.0 and
ruby 1.8 support has been dropped.)

If necessary, run `bundle install` before continuing. If you want you can also only install the needed gems:

```bash
# You can add --jobs with the number of cores you have to run this much faster in parallel.
bundle install --path .vendor --jobs 8
```

And in case you installed the gems before:

```bash
rm -fr .vendor
bundle install --path .vendor
```

If the module contains a Puppet Strings generated documentation, please
ensure the file is up-to-date. A good indicator for a Puppet Strings
documentation is the existence of a REFERENCE.md file. You can automatically
generate the documentation by running the following rake task:

```bash
bundle exec rake reference
```

We can generate the changelog after updating the metadata.json with a rake task
(in most cases, this requires a
[GitHub access token (docs on how to create one)](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line):
the changelog generator expects the token in the environment variable `CHANGELOG_GITHUB_TOKEN`)

```bash
CHANGELOG_GITHUB_TOKEN='mytoken' bundle exec rake changelog
```

The changelog generator checks for certain labels on closed issues and PRs
since the last release and groups them together. If the changes were neither
backwards-incompatible nor only bug fixes, make a minor release. Check the
generated diff for the CHANGELOG.md. If your chosen release version doesn't
match the generated changelog, update metadata.json and run the changelog task
again. Ensure the pull requests in the diff have the appropriate GitHub labels.
If not, update the label for the PR and re-run the changelog task.

Get community feedback on the release pr, label it with `skip-changelog`, get it merged.

## Doing the release


If you do the merge, you should do the following release rake task.

Run the rake target `release`. This will:

* create a new tag using the current version
* bump the current version to the next PATCH version and add `-rc0` to the end
* commit the change,
* and push it to origin.

*Please note that in order to execute this rake task you must be in the __Collaborators__ group on GitHub for the module in question.*

*Please also note that the task requires a configured gpg key in your local git settings*

```bash
bundle exec rake release
```

GitHub Actions (.github/workflows/release.yml in every module) will then kick
off a build against the new tag created and deploy that build to the forge.
Caution: The Vox Pupuli repo has to be the configured default branch in your
local clone. Otherwise, you will try to release to your fork.
