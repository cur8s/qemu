# TODO — cur8s.qemu

Open work only, in priority order.

## 1. Release hygiene
- [x] License: deliberately ambiguous across the cur8s family, by
  operator decision (2026-07-06) — no LICENSE file, on purpose. Note
  the standing tension: the docs invite vendoring while default
  copyright grants nothing; revisit if a consumer forces it.
- [ ] Enable immutable releases in the repository settings (GA
  2025-10-28) so published vX.Y.Z releases lock; the release workflow
  already ships the sha256 asset.

## 2. First consumers
- [ ] cur8s/ubuntu: refresh its vendored `scripts/qemu-vm.sh` from a
  tagged release once one exists (its copy currently predates the
  provenance header).
- [ ] The k3s repository's test rig: vendor the script + use the action
  (boot, install k3s from user-data or ssh, assert node Ready).
- [ ] The sandbox: local quick-VMs beside its DigitalOcean droplets.

## 3. Multi-VM networking (the multi-node k3s rig)
- [ ] Prototype rootless VM-to-VM networking. Research flags: `-netdev
  socket,mcast=` has a primary-source negative report on macOS hosts;
  QEMU 7.2's `dgram`/`stream` netdev backends are the cross-platform
  avenue; Lima's user-v2 (gvisor-tap-vsock) proves rootless fabrics are
  solvable. Lands as a minor version if additive verbs, major if it
  changes existing ones (RFC-002).

## 4. Watch list
- [ ] macadam (crc-org): the only other cloud-init-passthrough
  daemonless cross-platform CLI (33 stars, 2026). Re-evaluate in a
  year; if it matures, it could replace this product's script half.
- [ ] KVM on GitHub standard runners is policy-by-practice, not SLA
  (documented only as Android acceleration). If it regresses, CI falls
  back to larger runners.
