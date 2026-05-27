# RHEL Kernel Source Mirror (Rocky SRPMs → Git)

Branches:
- `rhel9`, `rhel10`: prepared kernel source trees per SRPM.
- Tags: `rhel<major>-<version-release>`.

This `main` branch stays minimal:
- README.md (this file)
- CHANGELOG.md (one line per imported SRPM)
- index.json (machine-readable list of all tags with metadata)

If enabled, per-arch Kconfig snapshots live under:
`metadata/configs/<arch>/<TAG>/` and `metadata/configs/<arch>/latest/`.
