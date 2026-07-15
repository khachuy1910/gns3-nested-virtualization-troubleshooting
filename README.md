# gns3-nested-virtualization-troubleshooting
Troubleshooting and fix for the GNS3 VM VT-x/hv.capable error on VMware Workstation 25H2u1 (Windows VBS conflict)

## Environment
- **OS:** Windows 11 Pro (build 26200)
- **Hypervisor:** VMware Workstation Pro 25.0.1.25219725 (25H2u1)
- **Nested VM:** GNS3 VM 2.2.59
- **CPU:** Intel Core i5-13500H
- **Security features involved:** Windows VBS / Core Isolation (Memory Integrity)


## Problem
When powering on the GNS3 VM with "Virtualize Intel VT-x/EPT" enabled,
VMware failed with:

`Feature 'hv.capable' was 0, but must be at least 0x1.
Module 'FeatureCompatLate' power on failed.`

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

  Since VT-x was confirmed enabled in firmware, the next hypothesis was that a Windows-level feature was intercepting it — Hyper-V and Core Isolation were the two most likely candidates.

## Troubleshooting Steps

### 1. Disabled Windows Hyper-V features (did not resolve)
powershell:
```powershell
dism.exe /Online /Disable-Feature:VirtualMachinePlatform
dism.exe /Online /Disable-Feature:HypervisorPlatform
```
Rebooted — error persisted. This ruled out Hyper-V's Windows Features
as the direct blocker.

### 2. Checked hypervisor boot configuration
cmd:
```cmd
bcdedit /enum {current} /v
```
`hypervisorlaunchtype` was not explicitly set, meaning Windows was
using its default behavior — not the root cause.

### 3. Checked Virtualization-based Security (VBS) status
`Win + r` -> `msinfo32`:

Result: `Virtualization-based security: Running`

This identified VBS (Device Guard) — a separate mechanism from
Hyper-V — as the actual process holding exclusive access to VT-x.

## Root Cause
Windows VBS was granting exclusive access to VT-x/EPT to itself,
preventing VMware from passing the feature through to the nested
GNS3 VM. This persisted independently of the Hyper-V Windows
Features and `hypervisorlaunchtype` setting.

## Fix
Disabled VBS via registry:

powershell:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" -Name "EnableVirtualizationBasedSecurity" -Value 0
```
Rebooted.

## Verification
`Win + r` -> `msinfo32`:

Result: `Virtualization-based security: Not enabled`

GNS3 VM powered on successfully with nested virtualization working.
