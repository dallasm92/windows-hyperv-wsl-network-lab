# Lab Summary

- Reviewed managed-switch VLAN configuration and port membership to confirm the Windows workstation was operating in a segmented lab network rather than a flat default layout.
- Validated Hyper-V host networking by capturing adapter state, IP configuration, and route-table evidence from Windows PowerShell.
- Installed Ubuntu in WSL and identified that the next failure was DNS/repository resolution inside WSL, not the WSL installation process itself.
- Used Windows-side `nslookup`, ping, and SSH tests to separate host connectivity from WSL-specific failure modes.
- Built a repeatable troubleshooting narrative that can be expanded later with the final resolver fix and sanitized screenshots.
