# Overview

aptly produces a fixed set of packages in the repository, so that package installation and upgrade becomes deterministic. At the same time aptly is able to perform controlled, fine-grained changes to repository content. aptly allows you to transition your package environment to a new version, or rollback to a previous version.

published representation of aptly generated snapshot or local repository, ready to be consumed by apt tools

The schema of aptly’s commands and transitions between entities:

![[Pasted image 20250307161927.png]]

We can start with creating [mirrors of remote repositories](https://www.aptly.info/doc/aptly/mirror/create). Also we can create [local package repositories](https://www.aptly.info/doc/aptly/repo/create) and [import package files](https://www.aptly.info/doc/aptly/repo/add). Local repos could be modified by [copying](https://www.aptly.info/doc/aptly/repo/copy) and [moving](https://www.aptly.info/doc/aptly/repo/move) packages between local repositories and [importing](https://www.aptly.info/doc/aptly/repo/import) them from mirrors. Snapshot could be [created](https://www.aptly.info/doc/aptly/snapshot/create) from remote repository (official Debian repositories, backports, 3rd party repos, etc.) or your local repository (custom built packages, your own software). Snapshots can be used to produce new snapshots by [filtering](https://www.aptly.info/doc/aptly/snapshot/filter/) other snapshots, [pulling](https://www.aptly.info/doc/aptly/snapshot/pull) packages with dependencies between snapshots and by [merging](https://www.aptly.info/doc/aptly/snapshot/merge) snapshots. Any snapshot can be [published](https://www.aptly.info/doc/aptly/publish/snapshot) to location (distribution name, prefix) and consumed by `apt` tools on your Debian systems. Local repositories could be [published directly](https://www.aptly.info/doc/aptly/publish/repo) bypassing snapshot step.

aptly can be used as [CLI tool](https://www.aptly.info/doc/commands) and [HTTP REST API service](https://www.aptly.info/doc/api), whatever works best in your environment.

Next section, [Why aptly?](https://www.aptly.info/doc/why/) describes why aptly is unique.