---
layout: post
title: Reviewing a module pull request
date: 2017-10-21
summary: Guidance for reviewing a pull request and criteria for merging.
---

There are a few things that can be checked if you review a pull request against
one of our modules:

* Does the email address used in the commits match the github email address? (This will let github display the contributor's avatar next to the commit)
* Is this a bugfix, modulesync, breaking change, enhancement, docs update? Label it with `bug`, `modulesync`, `backwards-incompatible`, `enhancement`, `docs`
* Are updates to the README.md needed but missing? Label it with `needs-docs`
* Has the updated file(s) documented all of its parameters or and examples in the puppet-strings header? This needs to be updated as well.
* Are there merge conflicts? You don't need to do anything. Our [Vox Pupuli Tasks GitHub App][vpt] will label this as `merge-conflicts` and notify the author
* Does it have failing tests? You don't need to do anything. Our [Vox Pupuli Tasks GitHub App][vpt] will label this as `tests-fail` and notify the author
* Were changes to master merged that are required in this PR (for example an updated GitHub Actions configuration)? Add the `needs-rebase` label
* Does it need additional tests? Add the `needs-tests` label
* Does it drop support for a specific Operating system or a major Puppet version? Add the `backwards-incompatible` label
* Are new parameters introduced? They must have datatypes
* Are facts used? They should only be accessed via `$facts[]` or [fact()](https://github.com/puppetlabs/puppetlabs-stdlib/blob/master/REFERENCE.md#fact) function from stdlib, but not topscope variables
* In the majority of cases, variables shouldn't be accessed via topscope: $::modulename::$param. Instead do: $modulename::$param
* Are datatypes from stdlib used? Ensure that lowest supported stdlib version is 4.18.0 (This is the first version that supports Puppet 5). Check if a newer version introduced the used datatype
* Are hiera yaml files added for data-in-modules? Ensure that the data is compatible with [hiera 5](https://puppet.com/docs/puppet/5.3/hiera_migrate.html#use-cases-for-upgrading-to-hiera-5). Static data that is equal across every supported operating system must stay in the init.pp, it shouldn't be moved to a common.yaml due to [puppet-strings issue #250](https://github.com/puppetlabs/puppet-strings/issues/250).
* Are there new parameters with datatype Hash or Array? If possible, they should default to empty `Hash` or `Array` instead of `undef`. You can also enforce the datastructure like `Array[String[1]]` (that's recommended if possible)
* Is the new datatype quite complex (like `Hash[String[1], Hash[String[1], Array[Any]]]`? Create a custom datatype for it.
* Are there new parameters with datatype Boolean? The default value is a tricky decision which needs careful reviewing. Sometimes a True/False is the better approach, sometimes `undef`
* Are there new parameters with literal datatypes (String, Integer, Float)? If they don't have a default value and aren't a required parameter, they should be marked `Optional` and default to `undef`. Their default value should not be `''` (empty string). A common pattern is to use `Optional[String[1]] $var = undef`. The parameter defaults to undef. It cannot be set to `''`, the minimal string length is 1 character
* Is this a bugfix? Write the Pull Request Title in a way that users can easily identify if they are impacted or not
* Does a new param map to an option in a config file of a service? The Parameter should accept the possible values that the service allows. For example 'on' and 'off'. Don't accept a boolean that will be converted to 'on' or 'off'
* Is a new template added? The preferred language is [epp](https://puppet.com/docs/puppet/latest/lang_template_epp.html), not [erb](https://puppet.com/docs/puppet/latest/lang_template_erb.html)
* Is a new class added? It should have unit tests using [rpsec-puppet-facts](https://github.com/mcanevet/rspec-puppet-facts#rspec-puppet-facts) that at least verify that the new class compiles. It also needs to have [puppet-strings docs](https://puppet.com/docs/puppet/6.17/puppet_strings.html).
* Files should always terminate with a newline if possible, with an exception being file or template fragments like those used with concat. This is the [POSIX standard][posix], and some tools don't handle the lack of a terminating newline properly
* If you can supply one or multiple values for an attribute it's common practice to enforce the datatype for one value and an array of that datatype. An example for string is `Variant[String[1],Array[String[1]]]`. This can be used in the Puppet code as `[$var].flatten()`
* In the past, the parameter section of a class or defined resource was always aligned at the `=` char. Sometimes there was an additional alignment at the `$` from the parameter name. Reordering this when a new parameter is introduced or when a datatype is changed, causes a lot of noise. Also it makes rebases, reverts and cherry-picks quite hard. New parameters should only have a single whitespace between the datatype and the variable, and also only a single whitespace for the default value. Example: `String[1] $foo = 'foo',`.
* Is a class considered private? Then it should contain [assert_private][as]
* A module should have as few public interfaces as possible. It should be aimed for the init.pp being the only public class. This is not a rule but a general guideline. Depending on the module, it is not always possible or feasible to configure everything through a single class.
* Is another module added as a dependency? Add it to the `.fixtures.yml` file as a git repository (as a `https://` link, not `ssh` or `git://`). Spec tests always run against master branches to detect breaking changes as early as possible. Acceptance tests use the last release (installed by [install_module_dependencies][imd] which parses it from the `metadata.json`)
* Only hard dependencies must be added to the metadata.json. Don't add soft dependencies! More explanation is [in the official Puppet styleguide][styleguide]
* Ensure that the version range of any dependency doesn't include an unreleased major version (do not allow version 6.X of a dependency if the current version is 5.X)
* An increase of an upper version boundary (of a module or Puppet itself) is only an enhancement if code adjustments were needed. Don't add the `enhancement` label if the only change is within the `metadata.json`. Ensure that `.fixtures.yml` doesn't pin a specific version.
* Sometimes you review a PR where somebody else requested changes. If the contributor clearly fixed it, you can still approve or merge it and ignore the `somebody requested changes` message. If you are not sure that it is really fixed, only approve it and do not merge it.
* When code deals with systemd units, dropin files, kernel modules to load or services limits (or other configuration options systemd can do), the [puppet/systemd][systemd] module should be used

### Approving and Merging

* You can merge your own PR if it was approved by a collaborator with merge permissions and CI is green. Don't merge if either one of those conditions are not true
  * Modulesync PRs are an exception (a PR based on changes that the msync tool did, NOT PRs on [modulesync_config][ms_docs]). We agreed some time ago that it's ok to merge your own modulesync PR if CI is green, without separate approval. This is okay because changes to [modulesync_config][ms_docs] were [reviewed and tested][ms_guid]
  * If your PR is non-trivial or perhaps has only been approved by a work colleague etc, please consider allowing reasonable extra time for other 3rd parties to leave their reviews before merging.
    There is no prescribed minimum review period, or definition of 'reasonable time'. Vox Pupuli trusts collaborators to use their own judgement here.
* It's okay to approve code regardless if CI is still running or not. The code won't be merged if CI fails after the PR got approved
* You are highly encouraged to review and approve code (or comment on it), even if you do not have merge permissions. This makes further reviews way easier

A green checkmark indicates that the review was done by someone with merge permissions:

<img alt="8bit vox" src="{{ site.url }}{{ site.baseurl }}/static/images/approved_pr_by_collaborator.png"/>


If you want to merge:

GitHub provides [multiple merge methods](methods):

<img alt="8bit vox" src="{{ site.url }}{{ site.baseurl }}/static/images/merge_methods.png"/>

Our default behaviour is `Create a merge commit`. Please keep this default.
GitHub will automatically sign the merge commit. You can merge it on the CLI if
you want to sign the commit with your own GPG key:

```sh
# clone the Vox Pupuli repo where you want to merge a Pull Request
git clone git@github.com:voxpupuli/puppet-myawesomemodule.git
# create a temporary branch
git checkout -b temp
# pull the changes from the pullrequest
git pull git@github.com:contributor/puppet-myawesomemodule.git featurebranch_from_the_pr
# switch back to the master branch
git checkout master
# merge the temp branch, that now contains the content from the PR
git merge --no-ff temp
# push the merge commit to the Vox Pupuli master branch
git push origin master
# delete the now obsolete temp branch
git branch -d temp
```

GitHub has some [docs available](gpg) to help you configuring GPG for git.
It's also good practice to automatically sign every commit. You can enable that
with:

```sh
git config --global commit.gpgsign true
```

[ms_docs]: https://github.com/voxpupuli/modulesync_config#modulesync-configs
[ms_guide]: https://github.com/voxpupuli/modulesync_config#contribution
[vpt]: https://github.com/voxpupuli/vox-pupuli-tasks#vox-pupuli-tasks---the-webapp-for-community-management
[posix]: http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html#tag_03_206
[as]: https://github.com/puppetlabs/puppetlabs-stdlib/blob/master/REFERENCE.md#assert_private
[imd]: https://github.com/puppetlabs/beaker-module_install_helper#install_module_dependencies
[styleguide]: https://puppet.com/docs/puppet/5.5/style_guide.html#dependencies
[methods]: https://help.github.com/en/articles/about-merge-methods-on-github
[gpg]: https://help.github.com/en/articles/generating-a-new-gpg-key
[systemd]:https://github.com/voxpupuli/puppet-systemd
