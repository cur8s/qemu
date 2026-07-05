# RFC-002: Distribution and Versioning

Status: Accepted

`qemu-vm.sh` is one file, and its distribution follows the idioms the
single-file problem already has. Three channels are supported, each
with its own integrity guarantee. Everything else — git submodules,
subtree, third-party vendoring tools — was evaluated and rejected; the
research record lives with consumer #1
(`cur8s/ubuntu`, `scripts/qemu-vm-research.md`).

## Vendored copies

Sibling repositories consume the script by copying it into their tree,
the way `config.guess`, `gradlew`, and `shunit2` are consumed. Every
vendored copy carries a provenance header in the `config.guess` idiom:
the version it was copied at, the canonical upstream URL, a
do-not-edit-here warning, and the exact command that refreshes it.
The header is what keeps the copy honest — the file is never mistaken
for local code, and never forked by accident.

Updates are deliberate re-copies at a pinned release, executed with
the refresh command from the header, never hand-edits. A local patch
that matters belongs upstream.

## Immutable releases

Public consumers fetch from GitHub Releases. Every release is
published immutable — tag and assets locked at publish time — and
ships the script together with a sha256 asset, so a consumer can pin
an exact artifact and verify it independently.

Pinning to an immutable release (or a full commit SHA) is doctrine,
not preference. Two incidents dictate it: the Codecov Bash Uploader
compromise (2021), where a hosted install script was silently modified
and caught only by a customer comparing checksums; and
tj-actions/changed-files (CVE-2025-30066, 2025), where version tags
were retroactively moved to a malicious commit and only SHA-pinned
consumers were unaffected. A mutable ref is a convenience pointer,
never an integrity anchor.

## The composite action

CI consumers use the composite action: `uses: cur8s/qemu/action@vN`.
The runner downloads the repository tarball at the referenced ref, so
the action always runs with the exact script it was released with.

Versioning follows the established actions norm: every release is a
semver tag (`vX.Y.Z`) published as an immutable release, and a plain
moving major tag (`v1`) is maintained as the convenience pointer,
force-moved to each release within the major. The major tag is mutable
by design; consumers who need immutability pin the full tag or the
commit SHA. Nothing — no channel, no example, no README snippet —
ever references `main`.

## What SemVer means here

The versioned contract is RFC-001's surface: the lifecycle verbs, the
`QVM_*` variables, and the on-disk layout.

- **Patch** — fixes within the contract: verbs, variables, and layout
  behave as before.
- **Minor** — additive and compatible: a new verb, a new `QVM_*`
  variable, or a new action input, each defaulting to the old
  behavior.
- **Major** — any change to an existing verb's behavior or arguments,
  to the semantics or default of an existing `QVM_*` variable, or to
  the on-disk layout (the contents of `QVM_DIR` or the image cache).

## Scope

This RFC defines how `qemu-vm.sh` reaches consumers and what its
version numbers promise.

It does not define the tool's contract itself (RFC-001: The Contract),
the mechanics of cutting a release, or any consumer's own pinning
policy.

## Revisions

Initial version.
