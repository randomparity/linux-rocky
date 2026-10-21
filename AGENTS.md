# Working with this kernel mirror

This repository mirrors RHEL-compatible Linux kernel releases from public Rocky
Linux SRPMs. The extractor lives in
[srpm-scanner](https://github.com/randomparity/srpm-scanner); its default `prep` mode
runs `rpmbuild -bp` and imports the prepared source tree, including downstream
patches. These are release snapshots, not upstream Linux development history.

## Find an exact release

`main` holds README.md, CHANGELOG.md, index.json, and repository guidance, rather
than the kernel source. `rhel9` and `rhel10` hold imported source snapshots;
branch tips move as releases are imported. Pin a tag for reproducible analysis.

Tags use `rhel<major>-<version-release>`, for example:
`rhel9-5.14.0-570.58.1.el9_6` or `rhel10-6.12.0-211.67.1.el10_2`.
Preserve the full release suffix: the upstream version alone is insufficient.

Run from the mirror root:

```sh
git tag --list 'rhel9-*' --sort=-version:refname
git tag --list 'rhel10-*' --sort=-version:refname
git tag --list '*570.58.1*'
git show main:CHANGELOG.md
git show main:index.json
```

The index maps tags to package NVRs, commits, build timestamps, and config paths.
Version sorting helps browse releases; it does not establish SRPM build order.
If a requested tag is missing locally, fetch with `git fetch origin --tags` and
check again. Do not substitute a nearby release without reporting it.

## Check out source in a sibling worktree

Keep the primary checkout available for documentation and imports. Put worktrees
under `~/src/linux-rocky-worktrees`, outside the repository. From the mirror root:

```sh
tag=rhel9-5.14.0-570.58.1.el9_6
wt="$HOME/src/linux-rocky-worktrees/$tag"
mkdir -p "$HOME/src/linux-rocky-worktrees"
git worktree add --detach "$wt" "$tag"
git -C "$wt" describe --tags --exact-match
git -C "$wt" rev-parse HEAD
git -C "$wt" status --short
cd "$wt"
```

Detached HEAD is appropriate for reading. For changes, create a dedicated branch
from the tag instead, using a new worktree path:

```sh
git worktree add -b work/kernel-investigation \
  "$HOME/src/linux-rocky-worktrees/kernel-investigation" "$tag"
```

Never modify imported tags or commit development changes to `main`, `rhel9`, or
`rhel10`. Multiple detached worktrees can examine the same tag; a branch can be
checked out in only one worktree at a time. To switch versions inside a clean
inspection worktree, use `git switch --detach <exact-tag>`. Read this guide before
leaving the documentation checkout: existing source tags do not contain it.

Inspect `git status --short` before cleanup, preserve any work, then remove an
unused worktree from the primary checkout with `git worktree remove <path>`.
Do not force removal of a dirty worktree.

## What the extracted tree contains

A local checkout of `rhel9-5.14.0-570.58.1.el9_6` was verified to contain:

- Kernel source directly at the root: `arch/`, `drivers/`, `fs/`, `include/`,
  `kernel/`, `mm/`, `net/`, `tools/`, `Documentation/`, `Kbuild`, and `Makefile`.
- Downstream files such as `Kconfig.redhat`, `Makefile.rhelver`, and `configs/`.
  `Makefile.rhelver` identifies RHEL 9.6 and release `570.58.1` in this snapshot.
- Imported `.config` and `.config.old`; their presence does not select the right
  architecture or configuration for your task.
- `metadata/configs/<arch>/<tag>/`, containing config files and `manifest.json`
  when config export was enabled. The verified x86_64 directory includes both
  a base config and a `*.resolved.config`. `latest/` is relative to the snapshot;
  use the explicit tag directory when pinning a release.

Layouts can vary by import. The importer's alternative `sources` mode stores raw
SRPM payloads in `packaging/SOURCES/` and `packaging/SPECS/`, rather than a prepared
kernel tree. Inspect the selected tag before assuming files exist.

For release comparisons, use `git diff <old-tag> <new-tag> -- <source-path>`.
Import commits represent whole SRPM snapshots; they do not provide individual
upstream or downstream patch history.

## Keep imports and investigations separate

Do not rerun extraction just to read an existing release. Import tooling and its
setup instructions belong in
[srpm-scanner](https://github.com/randomparity/srpm-scanner). The importer switches
branches and replaces checkout contents; coordinate imports with other work. Its
`MAIN_ENFORCE_MINIMAL=1` cleanup currently excludes AGENTS.md from the keep list,
so that option can delete this guide. Do not use it without addressing that.
Do not delete local `.work/` caches or ignored leftovers as part of an investigation.

There is no mirror-specific test or lint suite on `main`. For documentation edits,
check the diff and exercise the documented Git commands. Kernel builds require an
explicit target architecture, config, and suitable toolchain; inspect the tagged
`Documentation/` and build files first. Keep build output outside the source
worktree with the kernel's `O=<output-directory>` option and report the exact tag,
commit, architecture, config, and checks used. A successful checkout is not build
verification.
