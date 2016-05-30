---
layout: page
title: Documentation
---

This page is far from finished but contains some basic information on getting
started. If you have any questions reach out to us in #voxpupuli on Freenode.

* TOC
{:toc}

## Who can join?
Anyone can participate in voxpupuli. We currently have four levels of
participation, orchestrated by github teams.

* Everyone with a github account can submit pull requests and review code.
* Module contributors have commit/merge access to a subset of the modules in
  voxpupuli, usually the modules they maintained before joining
  voxpupuli.
* Collaborators have commit/merge access to a subset of the modules in
  voxpupuli.
* Administsrators have the authorization to create new repositories and
  publish modules to the forge under the puppet namespace.

## Migrating a module to voxpupuli
You will have someone by your side in this process. The general flow is to…

* Ask one of the Administrators to add you to the modules/admin team.
* At that point you can transfer your own repository
* If migrating a module from puppetlabs, re-enable github issues.
* Verify that all webhooks except travis are disabled.
* Release a 999.999.999 version of your modules to the forge, so everyone
  knows to stop using it.
* Release a copy of your module to the 'puppet' forge account.
* Add the module to our [modulesync setup](https://github.com/voxpupuli/modulesync_config/blob/master/managed_modules.yml)
* Add the module to our [plumbing repository](https://github.com/voxpupuli/plumbing/blob/master/share/modules)(handles travis secrets)
* Ask one of the Admins to add the module to the collaborators Team on github

If you have many modules you wish to migrate, this will be cumbersome.
In this case we will generally create a separate group and give you
administrator access to speed things up.

##  Publishing a module - setup
Forge publishing is handled by travis and puppet-blacksmith.

To guarantee a frictionless process across all modules, we use [modulesync](https://github.com/voxpupuli/modulesync). Our modulesync configuration is available at [modulesync_config](https://github.com/voxpupuli/modulesync_config).

Most modulesync'ed settings can be overridden through a [.sync.yml](https://github.com/voxpupuli/puppet-extlib/blob/master/.sync.yml). You may also need to (re)define your travis testing matrix with respect to puppet version. This prevents the deploy hook from running once for each version of puppet defined in your testing.

Travis needs to be aware of the rename, this can be done by pushing a single commit. Travis needs to be enabled for the new repository, you can do that [here](https://travis-ci.org/profile/voxpupuli).

The secure line is unique per repository and often the only line in .sync.yml. To get a secure line:

Ask an admin (or submit a PR) to add your module to the list [here](https://github.com/voxpupuli/plumbing/blob/master/share/modules). Then an admin will run the encrypt_travis.sh script and push a new version of [this](https://github.com/voxpupuli/plumbing/blob/master/share/travis_secrets) which you can then copy and paste your travis secure line from.

Note that you need to mask your ``secure:`` line in .travis.yml from modulesync. [Here](https://github.com/voxpupuli/puppet-iis/blob/master/.sync.yml#L35) is an example of what that looks like.

If the forge puppet password is changed, an admin can run encrypt_travis.sh and the modules can bring in the new password on their own schedule.


Gem publishing is handled similarly, except there is not a unified user. Each gem owner is responsible for their own .travis.yml

## Releasing a new version of a module
*Please note that in order to perform a release you must be in the __Collaborators__ group on Github for the module in question.*

Run modulesync to ensure the dotfiles are up to date.

Create a 'release pr'. This pull request updates the changelog and bumps the version number to the target version, removing all release candidate identifiers, i.e. from `0.10.7-rc0` to `0.10.7`. Here's an example: [puppet-extlib's 0.10.7 release](https://github.com/voxpupuli/puppet-extlib/pull/43). In most cases it is sufficient to update CHANGELOG.md and metadata.json. We try to honor [semantic versioning](http://semver.org/) and decided that dropping ruby1.8 support is a major change and requires a major version bump for the module.

Get community feedback on the release pr, get it merged.

Checkout an updated copy of master (`git checkout master; git fetch origin; git pull origin master`)

If necessary, run `bundle install` before continuing.

Run the rake target `travis_release`. This will:

* create a new tag using the current version
* bump the current version to the next PATCH version and add `-rc0` to the end
* commit the change,
* and push it to origin.

`bundle exec rake travis_release`

Travis will then kick off a build against the new tag created and deploy that build to the forge.
