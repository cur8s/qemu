# RFC-000: The Role of RFCs

Status: Accepted

This repository defines `qemu-vm.sh`, a single vendorable script that
boots disposable Ubuntu cloud VMs on QEMU. These RFCs are its normative
contract: they prescribe what must remain true rather than documenting
how the script happens to be implemented today. The script, the
composite action, and the tooling around them may evolve freely
provided they keep satisfying the constraints the RFCs establish.

A change that violates an accepted RFC is an architectural change and
must land as an explicit revision to the affected RFC, or as a new RFC
that supersedes it. Pre-1.0, RFCs are edited in place: clarifications
are folded in directly, and the Revisions section records only changes
of meaning. RFC numbers are stable identifiers, never renumbered or
reused.

RFCs describe what the tool is and why it exists, never how to operate
it. The guides (`docs/guides/`) own the how. The audience is anyone
evolving or consuming this repository: contributors, users deciding
whether to adopt the tool, and AI coding agents working in the tree.

## Scope

This RFC defines the purpose, authority, and evolution model of the
cur8s.qemu RFCs.

It does not define the tool's contract (RFC-001: The Contract) or how
the tool is distributed and versioned (RFC-002: Distribution and
Versioning).

## Revisions

Initial version.
