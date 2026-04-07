# Windows Host Networking, WSL, and VLAN Validation Lab

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

## Evidence Set

Screenshots are stored in [`images/`](/home/dallas/projects/windows-hyperv-wsl-network-lab/images).

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

- [01-switch-vlan-overview.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/01-switch-vlan-overview.png)
- [02-switch-port-vlan-membership.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/02-switch-port-vlan-membership.png)

Note:

- `TODO`: document the intended role of each VLAN in a future revision if you want this repo to read more like a full segmentation lab instead of a troubleshooting case study.

### 2. Inspect the Windows host networking stack

The next step was to inspect the Windows host itself instead of assuming the problem was on the switch or Linux side.

Observed evidence:

- multiple virtual adapters were present, including Hyper-V paths
- the host had both physical and virtual interfaces active
- PowerShell output confirmed that the workstation was carrying multiple address contexts at once

Evidence:

- [03-windows-adapter-overview.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/03-windows-adapter-overview.png)
- [04-windows-ip-configuration.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/04-windows-ip-configuration.png)
- [05-windows-route-table.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/05-windows-route-table.png)

### 3. Bring up WSL and identify the first failure

The screenshots show the workflow moving from "no distro installed" to installing Ubuntu in WSL.

After Ubuntu was installed, package operations inside WSL failed on repository name resolution.

That matters because it showed:

- WSL was installed successfully
- the next issue was network/DNS behavior inside WSL, not WSL installation itself

Evidence:

- [06-wsl-not-installed.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/06-wsl-not-installed.png)
- [07-wsl-distro-list.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/07-wsl-distro-list.png)
- [08-wsl-install-and-dns-failure.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/08-wsl-install-and-dns-failure.png)

### 4. Validate that Windows DNS and general connectivity still worked

The next checks moved back to the Windows host.

The evidence shows:

- Windows could resolve Ubuntu repository hostnames
- the workstation still had basic external connectivity

That narrowed the problem. The host was not completely offline. The failure was more specific to the WSL-side path or resolver behavior.

Evidence:

- [09-dns-check-archive-ubuntu.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/09-dns-check-archive-ubuntu.png)
- [10-dns-check-security-ubuntu-and-ping.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/10-dns-check-security-ubuntu-and-ping.png)

### 5. Test Linux-node access from the Windows workstation

The final screenshots in this set show a useful control test:

- one SSH path failed with a public-key error
- another SSH path succeeded

That is operationally useful because it proves Windows-to-Linux access was not universally broken. At least one path was healthy, which helps separate credential-specific issues from total network loss.

Evidence:

- [11-ssh-publickey-failure.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/11-ssh-publickey-failure.png)
- [12-ssh-pi-core-success.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/12-ssh-pi-core-success.png)

## What I Learned

- Hyper-V hosts can carry enough virtual networking state that "the network is broken" is usually too vague to be actionable.
- Windows host connectivity, SSH reachability, and WSL package resolution should be tested separately.
- A route table and adapter snapshot are often more useful early evidence than repeated browser retries.
- Switch-side VLAN review is valuable context, but host-side validation is what narrows the actual fault domain.

## Problems Encountered / Notes

- WSL package operations failed after installation because repository names were not resolving from inside WSL.
- One SSH path failed with `Permission denied (publickey)` while another worked, indicating a credential/path mismatch rather than a blanket connectivity outage.
- `TODO`: add the exact WSL resolver fix in a future revision if you want this repo to include remediation, not just diagnosis and validation.
- `TODO`: add a short diagram or table mapping the Windows host adapters to their intended roles.

## Redaction Review

The sensitive screenshots in this repo were redacted in place before publication.

Redactions were applied where needed for:

- [01-switch-vlan-overview.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/01-switch-vlan-overview.png)
  - exposes switch UI details and internal addressing context
- [02-switch-port-vlan-membership.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/02-switch-port-vlan-membership.png)
  - exposes device names and VLAN membership details
- [03-windows-adapter-overview.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/03-windows-adapter-overview.png)
  - exposes adapter names and Wi-Fi SSID
- [04-windows-ip-configuration.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/04-windows-ip-configuration.png)
  - exposes internal IP ranges
- [05-windows-route-table.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/05-windows-route-table.png)
  - exposes internal IPs and MAC addresses
- [09-dns-check-archive-ubuntu.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/09-dns-check-archive-ubuntu.png)
  - exposes internal resolver address
- [10-dns-check-security-ubuntu-and-ping.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/10-dns-check-security-ubuntu-and-ping.png)
  - exposes internal resolver address
- [11-ssh-publickey-failure.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/11-ssh-publickey-failure.png)
  - exposes internal host addressing
- [12-ssh-pi-core-success.png](/home/dallas/projects/windows-hyperv-wsl-network-lab/images/12-ssh-pi-core-success.png)
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
