# Windows Host Networking, WSL, and VLAN Validation Lab

Last reviewed: April 13, 2026

Social preview asset:
- [assets/social-preview.png](assets/social-preview.png)

This repo documents a Windows-based homelab troubleshooting workflow that touched three related areas:

- managed switch VLAN review
- Hyper-V host adapter and route inspection
- WSL bring-up and DNS troubleshooting from a Windows workstation

It is written as a portfolio-ready case study, not as a claim that every issue shown here was fully resolved in the same screenshot set.

## Objective

Validate how a Windows workstation interacted with a small lab network that included:

- a managed switch with VLAN membership changes
- Hyper-V virtual adapters on the Windows host
- WSL networking on the same host
- SSH access to Linux lab nodes

The main goal was to separate host-level connectivity from WSL-specific failure modes instead of treating all network symptoms as one problem.

## Lab Relationship

This repo fits between endpoint support work and broader homelab operations. It is useful as evidence of Windows networking diagnosis on a workstation that also carries Hyper-V and WSL complexity, rather than as a pure switch-configuration or VLAN-build project.

## Environment

- Windows workstation: `MAIN-PC`
- Hyper-V virtual adapters present on the host
- Managed switch: Netgear smart switch UI shown in evidence
- Linux lab nodes referenced in evidence:
  - `asus-server`
  - `pi-core`
- WSL target shown in screenshots:
  - `Ubuntu-24.04`

## What This Lab Demonstrates

- Reviewing VLAN mode and per-port membership on a managed switch
- Inspecting Windows adapter state and route tables on a Hyper-V host
- Verifying that Windows host connectivity and WSL connectivity can fail differently
- Using DNS checks and SSH tests to isolate where the failure actually lives
- Keeping troubleshooting evidence organized enough to reuse in portfolio documentation

## Hiring Manager Quick View

| Review area | Evidence |
|---|---|
| Windows networking | Adapter inventory, route table, DNS checks, and connectivity validation |
| Virtualization context | Hyper-V virtual adapters reviewed alongside physical host networking |
| WSL troubleshooting | WSL install succeeded, then package-resolution failure was isolated from host DNS |
| SSH troubleshooting | One Linux SSH path failed with public-key error while another succeeded, narrowing the fault domain |
| Documentation quality | Redacted screenshot set, evidence map, and clear boundary between diagnosis and remediation |

## Evidence Set

Screenshots are stored in [`images/`](images/).

Current sequence:

1. `01-switch-vlan-overview.png` — switch VLAN page showing configured VLAN entries
2. `02-switch-port-vlan-membership.png` — switch port membership view showing VLAN assignments per port
3. `03-windows-adapter-overview.png` — Windows Network Connections view with physical and virtual adapters
4. `04-windows-ip-configuration.png` — PowerShell output of current interface IP configuration
5. `05-windows-route-table.png` — Windows route table review
6. `06-wsl-not-installed.png` — WSL state before a distro was installed
7. `07-wsl-distro-list.png` — available WSL distributions listed
8. `08-wsl-install-and-dns-failure.png` — Ubuntu installation followed by package-resolution failure inside WSL
9. `09-dns-check-archive-ubuntu.png` — Windows `nslookup` check for `archive.ubuntu.com`
10. `10-dns-check-security-ubuntu-and-ping.png` — Windows `nslookup` plus general reachability test
11. `11-ssh-publickey-failure.png` — SSH failure to one lab host using the current key path
12. `12-ssh-pi-core-success.png` — SSH success to another lab host from the same Windows system

## Steps Performed

### 1. Review managed-switch VLAN state

The first screenshots capture the switch-side view of the network:

- VLAN configuration was present in the switch UI
- port membership reflected a non-default VLAN layout rather than a flat all-default setup

Evidence:

- [01-switch-vlan-overview.png](images/01-switch-vlan-overview.png)
- [02-switch-port-vlan-membership.png](images/02-switch-port-vlan-membership.png)

Scope note:

- This repo is a troubleshooting case study, not a full VLAN segmentation design. The VLAN screenshots are included as supporting context for the Windows host investigation.

### 2. Inspect the Windows host networking stack

The next step was to inspect the Windows host itself instead of assuming the problem was on the switch or Linux side.

Observed evidence:

- multiple virtual adapters were present, including Hyper-V paths
- the host had both physical and virtual interfaces active
- PowerShell output confirmed that the workstation was carrying multiple address contexts at once

Evidence:

- [03-windows-adapter-overview.png](images/03-windows-adapter-overview.png)
- [04-windows-ip-configuration.png](images/04-windows-ip-configuration.png)
- [05-windows-route-table.png](images/05-windows-route-table.png)

### 3. Bring up WSL and identify the first failure

The screenshots show the workflow moving from "no distro installed" to installing Ubuntu in WSL.

After Ubuntu was installed, package operations inside WSL failed on repository name resolution.

That matters because it showed:

- WSL was installed successfully
- the next issue was network/DNS behavior inside WSL, not WSL installation itself

Evidence:

- [06-wsl-not-installed.png](images/06-wsl-not-installed.png)
- [07-wsl-distro-list.png](images/07-wsl-distro-list.png)
- [08-wsl-install-and-dns-failure.png](images/08-wsl-install-and-dns-failure.png)

### 4. Validate that Windows DNS and general connectivity still worked

The next checks moved back to the Windows host.

The evidence shows:

- Windows could resolve Ubuntu repository hostnames
- the workstation still had basic external connectivity

That narrowed the problem. The host was not completely offline. The failure was more specific to the WSL-side path or resolver behavior.

Evidence:

- [09-dns-check-archive-ubuntu.png](images/09-dns-check-archive-ubuntu.png)
- [10-dns-check-security-ubuntu-and-ping.png](images/10-dns-check-security-ubuntu-and-ping.png)

### 5. Test Linux-node access from the Windows workstation

The final screenshots in this set show a useful control test:

- one SSH path failed with a public-key error
- another SSH path succeeded

That is operationally useful because it proves Windows-to-Linux access was not universally broken. At least one path was healthy, which helps separate credential-specific issues from total network loss.

Evidence:

- [11-ssh-publickey-failure.png](images/11-ssh-publickey-failure.png)
- [12-ssh-pi-core-success.png](images/12-ssh-pi-core-success.png)

## What I Learned

- Windows host networking, Hyper-V virtual adapters, and WSL resolver behavior can fail independently even when they share the same workstation.
- Validating DNS from the Windows host before focusing on WSL narrows the problem much faster.
- Mixed SSH results are useful evidence. A single successful path proves the workstation is not experiencing a universal outbound failure.

## Problems Encountered / Notes

- This evidence set documents diagnosis and narrowing, not a complete end-to-end remediation for every symptom shown.
- The repo is intentionally written as a troubleshooting case study rather than a claim that every networking issue in the screenshots was resolved in the same session.
- A future lab can document the exact WSL resolver remediation if that fix is captured with clean evidence. This repo stays honest about the evidence it currently contains.

## Redaction Review

The sensitive screenshots in this repo were redacted in place before publication.

Redactions were applied where needed for:

- [01-switch-vlan-overview.png](images/01-switch-vlan-overview.png)
  - exposes switch UI details and internal addressing context
- [02-switch-port-vlan-membership.png](images/02-switch-port-vlan-membership.png)
  - exposes device names and VLAN membership details
- [03-windows-adapter-overview.png](images/03-windows-adapter-overview.png)
  - exposes adapter names and Wi-Fi SSID
- [04-windows-ip-configuration.png](images/04-windows-ip-configuration.png)
  - exposes internal IP ranges
- [05-windows-route-table.png](images/05-windows-route-table.png)
  - exposes internal IPs and MAC addresses
- [09-dns-check-archive-ubuntu.png](images/09-dns-check-archive-ubuntu.png)
  - exposes internal resolver address
- [10-dns-check-security-ubuntu-and-ping.png](images/10-dns-check-security-ubuntu-and-ping.png)
  - exposes internal resolver address
- [11-ssh-publickey-failure.png](images/11-ssh-publickey-failure.png)
  - exposes internal host addressing
- [12-ssh-pi-core-success.png](images/12-ssh-pi-core-success.png)
  - exposes internal hostnames

## Outcome

This repo now captures a real troubleshooting sequence rather than isolated screenshots with no narrative.

The documented outcome is:

- switch VLAN state was reviewed
- Windows adapter and route state was captured
- WSL installation succeeded
- the remaining failure was narrowed to WSL-side DNS/repository access rather than full host connectivity loss
- Windows-to-Linux access testing showed mixed results that helped isolate the problem further

That makes this a solid evidence-backed operations case study, and it is now structured for public GitHub use with the copied evidence set redacted in place.
