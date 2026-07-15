# gns3-nested-virtualization-troubleshooting
Troubleshooting and fix for the GNS3 VM VT-x/hv.capable error on VMware Workstation 25H2u1 (Windows VBS conflict)

## Environment
- **OS:** Windows 11 Pro (build 26200)
- **Hypervisor:** VMware Workstation Pro 25.0.1.25219725 (25H2u1)
- **Nested VM:** GNS3 VM 2.2.59
- **CPU:** Intel Core i5-13500H
- **Security features involved:** Windows VBS / Core Isolation (Memory Integrity)

## Diagnosis
1. Check `Task Manager` -> `Performance` -> `CPU`:

`Virtualization: Enabled`

This confirmed that VT-x was enabled at BIOS level, so the issue was at software-level not a BIOS setting.

2. In `cmd` check:
      ```cmd
      systeminfo
      ```
      
  Or open System Information:

  `A hypervisor has been detected...`

  This indicated Window is running Hyper-V, which prevents VMware from getting direct access to VT-x
