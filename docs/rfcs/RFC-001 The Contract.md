# RFC-001: The Contract

Status: Accepted

`qemu-vm.sh` boots a disposable Ubuntu cloud VM on QEMU from a plain
cloud-init user-data file. This RFC states the invariants that make the
tool worth existing. A change that breaks any of them produces a
different tool and requires a revision here first.

## User-data is the interface

The tool consumes a plain `#cloud-config` file and delivers it to the
guest verbatim, as a NoCloud seed. It never generates, templates,
merges, or interprets user-data. Provisioning belongs to cloud-init;
the tool's job ends at delivery.

The consequence is the load-bearing differentiator: the same file that
boots the local VM must remain usable, unchanged, to provision a real
cloud host. No established tool preserves this property — Lima
generates and owns its user-data from an internal template, and
Multipass passes it through only from behind a privileged always-on
daemon. Any feature that would give this tool an opinion about the
content of user-data is out.

## Daemonless and rootless

One script, one QEMU process per VM, nothing else: no background
service, no bridges, no state outside the VM directory and the image
cache, and no root beyond installing QEMU itself. Networking is QEMU
user-mode with a single SSH port forwarded on localhost. The tool must
remain runnable, unmodified, by an unprivileged user on a fresh
machine and on a CI runner.

## Host-architecture guests

The guest architecture follows the host, under hardware acceleration:
hvf on macOS, KVM on Linux. The tool never emulates a foreign
architecture — a TCG-emulated guest is a different product with
different performance promises, and offering it would blur what "a VM
in a couple of minutes" means.

## Lifecycle verbs are the surface

The command surface is a fixed verb set: `fetch-image`, `build-vm`,
`start-vm`, `wait-until-ready`, `ssh`, `status`, `show-boot-log`,
`destroy-vm`, `help`.
Configuration is `QVM_*` environment variables, every one with a
working default; setting `QVM_USER_DATA` alone must be enough to get a
working VM. `destroy` leaves nothing behind but the cached image.

## Images

Guests boot upstream Ubuntu cloud images, fetched from the publisher
and verified against the publisher's SHA256SUMS before first use. Each
VM runs on a copy-on-write overlay; the cached base image is never
written to, so every VM starts from a pristine image without a fresh
download.

## Curation over flexibility

The tool carries the cur8s posture: the absence of an option is
intentional, not an oversight. QEMU's flag surface is enormous, and the
value here is one curated path through it — this is not a libvirt
replacement, an image builder, or a VM fleet manager. A need outside
the contract belongs in a locally patched vendored copy or in a
different tool, not in a new flag.

## Scope

This RFC defines the invariants of `qemu-vm.sh`: its interface, its
process model, its image handling, and its posture.

It does not define how the tool reaches consumers or what its version
numbers promise (RFC-002: Distribution and Versioning), nor the content
of any user-data file — that contract belongs to cloud-init.

## Revisions

Initial version.
