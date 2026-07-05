# User Guide

How to boot disposable Ubuntu VMs with `qemu-vm.sh`: on your
workstation, in CI, or vendored into your own repository. The RFCs in
`docs/rfcs/` own the what and the why; this guide owns the how.

## 1. What this is

A single POSIX shell script that boots a disposable Ubuntu cloud VM on
QEMU from a plain cloud-init `#cloud-config` file — the same file,
unchanged, that would provision a real cloud host. One QEMU process
per VM, no daemon, no root, no bridges; `destroy` leaves nothing
behind but the cached image (RFC-001: The Contract).

## 2. Requirements

- **macOS**: `brew install qemu`. Nothing else — the seed ISO is built
  with the system's `hdiutil`.
- **Linux**: `qemu-system-x86` or `qemu-system-arm` (matching your
  host), `qemu-utils`, `genisoimage`, and access to `/dev/kvm`.

The guest architecture follows the host — arm64 hosts boot arm64
guests, x86_64 hosts boot amd64 guests — always hardware-accelerated
(hvf on macOS, KVM on Linux). There is no foreign-architecture
emulation.

## 3. Quick start on a workstation

Write a minimal `#cloud-config` — one user, one SSH key, password
authentication off:

```sh
ssh-keygen -t ed25519 -N '' -f ./key
cat > user-data.yaml <<EOF
#cloud-config
users:
  - name: ubuntu
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - $(cat ./key.pub)
ssh_pwauth: false
EOF
```

Then create, boot, wait, and connect:

```sh
QVM_USER_DATA=./user-data.yaml ./qemu-vm.sh create
./qemu-vm.sh boot
./qemu-vm.sh wait ubuntu ./key
./qemu-vm.sh ssh ubuntu ./key
```

`create` fetches and checksum-verifies the Ubuntu cloud image on first
use (cached thereafter), builds a copy-on-write overlay disk, and
packs your user-data verbatim into a NoCloud seed. `boot` starts QEMU
daemonized with SSH forwarded to `127.0.0.1:2222`; `wait` blocks until
cloud-init reports done — and rides out a first-boot reboot if your
user-data requests one (`power_state`), re-verifying on the far side.
`ssh` takes an optional command after the identity file; without one
it opens a shell.

While it runs: `./qemu-vm.sh status` for the facts,
`./qemu-vm.sh console` to follow the serial log. When you are done:

```sh
./qemu-vm.sh destroy
```

The VM and its state directory are gone; the cached image stays, so
the next `create` is fast.

## 4. Configuration reference

Everything is an environment variable, and every value has a working
default — `QVM_USER_DATA` is the only one `create` requires.

| Variable | Meaning | Default |
| --- | --- | --- |
| `QVM_DIR` | VM state directory | `./.qvm/vm` |
| `QVM_CACHE_DIR` | image cache directory | `./.qvm/cache` |
| `QVM_NAME` | instance-id and hostname | `qemu-vm` |
| `QVM_USER_DATA` | cloud-init user-data file | none; required by `create` |
| `QVM_IMAGE_URL` | cloud image | Ubuntu 24.04 noble, current, host architecture |
| `QVM_SHA256SUMS_URL` | checksum file | `SHA256SUMS` beside the image URL |
| `QVM_SSH_PORT` | forwarded SSH port | `2222` |
| `QVM_CPUS` | virtual CPUs | `4` |
| `QVM_MEMORY` | guest memory | `4G` |
| `QVM_DISK_SIZE` | overlay disk size | `20G` |
| `QVM_WAIT_TIMEOUT_SECONDS` | `wait` deadline | `1200` |

To run more than one VM at once, give each its own `QVM_DIR`,
`QVM_NAME`, and `QVM_SSH_PORT`.

## 5. CI usage

The composite action boots the VM and hands you its coordinates as
step outputs:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Boot a VM
        id: vm
        uses: cur8s/qemu/action@v1
        with:
          user-data: ./ci/user-data.yaml
      - name: Run against the VM
        run: >
          ssh -i ${{ steps.vm.outputs.ssh-identity }}
          -p ${{ steps.vm.outputs.ssh-port }}
          -o StrictHostKeyChecking=accept-new
          ${{ steps.vm.outputs.ssh-user }}@127.0.0.1
          cloud-init status
```

`ubuntu-latest` on x64 is required: GitHub's arm64 and macOS hosted
runners have no KVM, so hardware-accelerated VMs only run on the x64
Ubuntu runners. If you prefer to own the lines instead of binding to
the action, `examples/boot-vm.yml` is the same job written out in
full, ready to copy and adapt.

## 6. Vendoring into your repository

Copy the one file in, keep its provenance header intact — the header
records the version, the upstream URL, and the refresh command, and is
what stops the copy from drifting into local code. To refresh, re-copy
at a pinned release:

```sh
gh api -H "Accept: application/vnd.github.raw" \
  "repos/cur8s/qemu/contents/qemu-vm.sh?ref=vX.Y.Z" > scripts/qemu-vm.sh
```

Updates are deliberate: pick the release, run the one-liner, review
the diff, commit. Never hand-edit the vendored copy — a patch worth
keeping belongs upstream (RFC-002: Distribution and Versioning).

## 7. What it is not

Not a daemon — nothing runs when your VM isn't running. Not a
foreign-architecture emulator — the guest always matches the host.
And not a config-management DSL — the tool passes your user-data
through untouched and takes no view of its contents: your user-data
is your provisioning, and it works on a real cloud exactly as it does
here.
