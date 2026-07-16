---
author: Bronwen Aker
updated: 2026-07-13
presentation_type: Workshop
venue: Antisyphon AI Summit
---

# VM Export Preflight Checklist

Steps to run on both HeartOfGold and Marvin before zipping them up for delivery to students. Draft list — review and trim what doesn't apply.

## Identity & Networking

- [ ] **Log out of Tailscale** (`tailscale logout` on each VM) before export. If a VM is exported while still authenticated, every student's clone inherits the same Tailscale node identity/key, causing conflicts when multiple students bring their clones online at once. Students should run `tailscale up` fresh after import, into their own tailnet.
- [ ] **Revoke any Tailscale auth keys** used during testing, and remove HeartOfGold/Marvin as devices from your personal Tailscale admin console so they don't linger as stale/authorized devices on your own tailnet.
- [ ] **Regenerate SSH host keys** on each VM (remove `/etc/ssh/ssh_host_*` and run `sudo ssh-keygen -A`, or let cloud-init/first-boot regenerate them) so every student clone doesn't share the same host key identity.
- [ ] **Clear/regenerate `/etc/machine-id`** (and `/var/lib/dbus/machine-id`) so cloned VMs don't collide on machine identity. This can affect DHCP leases and systemd/journald behavior if left shared.
- [ ] **Confirm network adapter mode** is NAT/bridged-DHCP, not anything hardcoded to your home network.

## Security & Credential Hygiene

- [ ] **Clear shell history** (`.bash_history`) on both VMs, especially important if a Tailscale auth key or any other secret was typed out during testing.
- [ ] **Confirm the `benjy`/`frankie` credentials are intentional to ship in plaintext** (consistent with how WWHF published `llm-lab`/`LLMs4evr`), rather than something you meant to rotate per cohort.
- [ ] **Scrub personal/identifying info** from both VMs: git config, saved Wi-Fi profiles, browser history/profiles, shell prompt customizations tied to you personally.

## Disk & Size

- [ ] **Verify the root filesystem actually uses the full allocated disk.** Ubuntu Server's autoinstall defaults to an LVM layout that can leave a large chunk of the virtual disk unallocated in the volume group (seen firsthand on HeartOfGold: a 30GB disk yielding only a 14GB root LV, half the disk invisible to the OS). Run `lsblk` and `sudo vgs` on each VM; if `VFree` is non-trivial, run `sudo lvextend -l +100%FREE /dev/<vg>/<lv>` followed by `sudo resize2fs /dev/<vg>/<lv>`, then confirm with `df -h /`.
- [ ] **Clear package and temp caches** (`apt clean`, `/tmp` contents, leftover installers) to shrink the image before zipping.
- [ ] **Consolidate/delete VMware snapshots** before export. A snapshot chain will bloat the exported file and can behave unpredictably after import.
- [ ] **Compact/shrink the virtual disks** to minimize the final zip size.

## Content & Functionality

- [ ] **Confirm final model(s) are already pulled on HeartOfGold** so students aren't downloading multi-GB models over conference Wi-Fi mid-workshop.
- [ ] **Confirm nginx is installed but NOT pre-configured on HeartOfGold.** Lesson 06 needs something for students to actually build, not a finished reverse proxy.
- [ ] **Verify sudo access** still works for both `benjy` and `frankie` after all the credential/user changes made during setup.
- [ ] **Do a clean end-to-end test**: import the exported zip fresh (not your working copies) on a separate machine and walk the entire lab sequence start to finish as a student would.
- [ ] **Confirm VMware Tools / open-vm-tools** is installed and working on both VMs (clipboard, display resizing, etc.).

## Naming

- [ ] **Finalize and rename the VMs.** HeartOfGold and Marvin are working names; confirm final names before export (per earlier note, this is being deferred until closer to shipping).

## Content Style

- [ ] **Scan all shipped course materials** (Lab Manual pages, READMEs, any in-VM docs) for em dashes and remove them per house style: no hyphens or dashes unless grammatically required.

