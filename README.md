# cur8s.qemu

A disposable Ubuntu VM on QEMU, in one vendorable file. Give
`qemu-vm.sh` a plain cloud-init `#cloud-config` and get a VM on
`localhost:2222` in a couple of minutes — the same user-data you would
hand a real cloud host, passed through verbatim. Daemonless, rootless,
host-arch (hvf on macOS Apple silicon, KVM on Linux and CI).
`destroy` leaves nothing behind but the cached image.

## In CI (GitHub Actions)

```yaml
jobs:
  vm-test:
    runs-on: ubuntu-latest   # x64: no KVM on arm64/macOS runners
    steps:
      - uses: actions/checkout@v4
      - uses: cur8s/qemu/action@v0
        id: vm
      - run: |
          ssh -p ${{ steps.vm.outputs.ssh-port }} \
            -i ${{ steps.vm.outputs.ssh-identity }} \
            -o UserKnownHostsFile=/tmp/kh -o StrictHostKeyChecking=accept-new \
            ${{ steps.vm.outputs.ssh-user }}@127.0.0.1 'uname -a'
```

The action installs QEMU, opens `/dev/kvm`, caches the image, boots,
and waits for cloud-init; pass `user-data:` to use your own
`#cloud-config` (with `ssh-user:` and `ssh-identity:` matching it).
Prefer owning the lines? Copy `examples/boot-vm.yml`.

## On a workstation

```sh
# macOS: brew install qemu    Linux: apt install qemu-system-x86 qemu-utils genisoimage
QVM_USER_DATA=./user-data.yaml ./qemu-vm.sh create
./qemu-vm.sh boot
./qemu-vm.sh wait ubuntu ./key      # rides out a first-boot reboot
./qemu-vm.sh ssh  ubuntu ./key
./qemu-vm.sh destroy
```

Full command and `QVM_*` configuration reference: the script's own
header, and `docs/guides/user-guide.md`.

## Getting the file

- **Vendor it** (sibling repos): copy `qemu-vm.sh`, keep its provenance
  header; refresh deliberately with the pinned one-liner in the header.
- **Release download** (everyone else): each `vX.Y.Z` release carries
  `qemu-vm.sh` and its sha256; pin versions, verify checksums.
- **The action** (`cur8s/qemu/action@v0`): `v0` follows the latest
  compatible release until a deliberate 1.0; pin the full SHA if you need immutability.

## The contract

The RFCs in `docs/rfcs/` are normative: RFC-001 (cloud-init user-data
is the interface; daemonless; host-arch; the lifecycle verbs), RFC-002
(distribution channels and SemVer meaning). The research that shaped
them is recorded in `docs/notes/qemu-vm-research.md`. Contributor loop: `mise run test`
(local smoke), `mise run lint`; CI dogfoods the action on every push.
