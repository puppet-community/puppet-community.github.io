---
layout: post
title: Software Support Lifecycle
date: 2021-06-11
summary: Vox Pupuli policy on supporting upstream software
---

## Software Support Lifecycle

As a part of Vox Pupuli's mission to provide a home for modules that still have a userbase and need development, we need to define a clear support strategy for what operating systems, dependencies, and upstream software we will support and for how long those will be supported.

As we operate entirely on the graces of our community members being willing to voluntarily contribute to our modules, we also need to be respectful of their time and resources and not add undue stress in order to support software that upstream creators no longer support.

To account for this, we no longer support any software or operating system after the date that the maintainers have marked said software or operating system as End of Life, hereafter referred to as "EOL". However, this does not mean we will always immediately remove code that allows for the module to run on EOL'd software or operating systems, but the community will no longer be expected to provide support to users for that software. The list of supported operating systems in `metadata.json` is always leading.

If a user still needs to use the EOL operating system, then they must either:

* Pin their module version to a version compatible with their operating system.
* Fork the module and maintain it internally.

### Support Lifecycle

#### Operating Systems

An Operating System is no longer considered supported once it reaches an EOL date determined by the upstream maintainer* of the operating system. To ensure our users have enough time to adjust, we will take the following steps to ensure as seamless of a transition as possible:

1. A deprecation PR will be created no later than 3 months before an Operating System's EOL date.
2. Removal of the EOL version from the metadata.json and tests and related code to be completed by the EOL date.
3. Issues relating to the EOL operating system will no longer be addressed after the EOL date.
4. Pull Requests specific to the EOL operating system will no longer be accepted.

*maintainers include, but are not limited to, the community, company, or organization that maintains ownership over a specific piece of software or project.

##### Support Table

|      OS Name       | Support Begins |         EOL         |   Notification PR   | End of Support | Removal of Code |
| :----------------: | :------------: | :-----------------: | :-----------------: | :------------: | :-------------: |
|   CentOS/RHEL 6    |   2011/07/10   |     2020/11/30      |         N/A         |   2020/11/30   |   2020/11/30    |
|   CentOS/RHEL 7    |   2014/07/07   |     2024/06/30      |     2024/03/31      |   2024/06/30   |   2024/06/30    |
|      CentOS 8      |   2019/09/24   |     2021/12/31      |     2021/09/30      |   2021/12/31   |   2021/12/31    |
|       RHEL 8       |   2019/05/01   |     2029/05/31      |     2029/02/28      |   2029/05/31   |   2029/05/31    |
|  CentOS Stream 8   |   2022/01/01   |     2024/05/31      |     2024/02/28      |   2024/05/31   |   2024/05/31    |
|    Ubuntu 16.04    |   2016/04/21   |     2021/04/30      |     2021/01/31      |   2021/04/30   |   2021/04/30    |
|    Ubuntu 18.04    |   2018/04/26   |     2023/04/30      |     2023/01/31      |   2023/04/30   |   2023/04/30    |
|    Ubuntu 20.04    |   2020/04/23   |     2025/04/30      |     2025/01/31      |   2025/04/30   |   2025/04/30    |
|      Debian 9      |   2017/06/17   |     2022/01/31      |     2021/10/31      |   2022/01/31   |   2022/01/31    |
|     Debian 10      |   2019/07/06   | 2024/01/31 (approx) |     2023/10/31      |   2024/01/31   |   2024/01/31    |
|     Fedora 31      |   2019/10/29   |     2020/11/24      |         N/A         |   2020/11/24   |   2020/11/24    |
|     Fedora 32      |   2020/04/21   | 2021/05/31 (approx) | 2021/02/28 (approx) |   2021/05/31   |   2021/05/31    |
| OpenSUSE Leap 15.1 |   2019/05/22   |     2021/01/31      |     2020/12/31      |   2021/01/31   |   2021/01/31    |
| OpenSUSE Leap 15.2 |   2020/07/02   |     2021/12/31      |     2020/09/30      |   2021/12/31   |   2021/12/31    |

NOTE: *\*All Dates subject to change based on upstream project*

Sources:

* OpenSUSE - [OpenSUSE Documentation](https://en.opensuse.org/Lifetime)
* All others - [EndOfLife.software](https://endoflife.software/operating-systems)

#### Puppet Versions

Puppet support will fall in-line with the Puppet Open Source project's lifecycle.

1. A Puppet version that has been labelled as EOL by the Puppet Open Source project, will no longer be supported by Vox Pupuli.
2. A deprecation PR will be created no later than 6 months before EOL.
3. All tests and data in metadata.json will be removed no later than the EOL date for that version. Code specifically for the EOL version will be removed as well.
4. Pull Requests and Issues relating to the EOL version of Puppet will no longer be accepted.
5. Issues are only valid if they affect supported versions.

#### Vox Pupuli Modules

Modules are considered supported only at their current latest release and roll forward, not back in case of any bugs or issues.

1. Modules are supported at their latest current version.
    1. Latest version is determined by the version number based on the following order:
        1. Major
        2. Minor
        3. Patch
        4. Examples:
            1. `5.1.1` is the latest version and supported.
            2. `5.1.0` is not supported.
            3. `4.9.9` is not supported.
    2. Exceptions:
        1. The maintainer may make exceptions to the above rule as long as it does not create undue expectations for the community.
2. For all other documentation on Archived Modules see [Deprecated and Archived Modules](https://voxpupuli.org/docs/deprecated_and_archived_modules/)
