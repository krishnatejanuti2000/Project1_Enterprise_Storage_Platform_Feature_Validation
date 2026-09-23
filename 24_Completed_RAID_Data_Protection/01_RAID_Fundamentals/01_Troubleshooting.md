# Troubleshooting – RAID Fundamentals

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

Topic: RAID Fundamentals

---

# Introduction

This document describes common RAID-related issues encountered during RAID planning, deployment, configuration, and administration. It focuses on identifying possible causes and recommended solutions before discussing RAID-level-specific troubleshooting.

---

# Issue 1 – Wrong RAID Level Selection

## Problem

The selected RAID level does not meet the application's business requirements.

## Possible Symptoms

- Poor application performance
- Insufficient data protection
- Lower than expected usable capacity

## Possible Causes

- Performance prioritized instead of redundancy
- Capacity requirements calculated incorrectly
- Business requirements not analyzed before deployment

## Resolution

- Understand workload requirements.
- Identify whether the priority is Performance, Data Protection, or Capacity.
- Select the appropriate RAID level based on business requirements.

---

# Issue 2 – Unexpected Usable Capacity

## Problem

The usable capacity is much smaller than the total installed disk capacity.

## Possible Cause

Some RAID levels reserve storage for redundancy.

Examples:

- RAID 1 reserves 50% of raw capacity.
- RAID 5 reserves the equivalent of one disk.
- RAID 6 reserves the equivalent of two disks.

## Resolution

Always calculate both:

- Raw Capacity
- Usable Capacity

before deploying storage.

---

# Issue 3 – RAID Controller Not Detected

## Problem

The operating system cannot detect the RAID controller or RAID volumes.

## Possible Causes

- Controller driver missing
- Controller firmware issue
- Hardware connection problem
- Controller disabled in BIOS/UEFI

## Resolution

- Verify RAID controller detection.
- Install the required drivers.
- Check controller firmware.
- Verify BIOS/UEFI configuration.

---

# Issue 4 – Software RAID Not Available

## Problem

Linux Software RAID commands fail.

Example:

```bash
mdadm: command not found
```

## Possible Cause

The `mdadm` package is not installed.

## Resolution

Ubuntu/Debian:

```bash
sudo apt install mdadm
```

RHEL/CentOS:

```bash
sudo dnf install mdadm
```

Verify installation:

```bash
mdadm --version
```

---

# Issue 5 – Incorrect RAID Planning

## Problem

The deployed RAID configuration does not satisfy customer expectations.

## Possible Causes

- Workload analysis not performed
- Incorrect RAID level selected
- Capacity planning errors
- Fault tolerance requirements ignored

## Resolution

Perform workload analysis before deployment and choose the RAID level that best matches the business requirements.

---

# Issue 6 – Assuming RAID is a Backup

## Problem

Data is lost even though RAID was configured.

## Cause

RAID improves availability and fault tolerance but does not replace backups.

Examples:

- Accidental file deletion
- File corruption
- Malware or ransomware
- Application-level corruption

These issues affect the RAID array just as they would affect a single disk.

## Resolution

Always maintain regular backups in addition to RAID.

---

# Common Validation Commands

Linux Storage Engineers frequently use the following commands while validating RAID configurations.

View block devices:

```bash
lsblk
```

Display RAID status:

```bash
cat /proc/mdstat
```

Display RAID details:

```bash
mdadm --detail
```

Examine RAID metadata:

```bash
mdadm --examine
```

Display filesystem usage:

```bash
df -h
```

---

# Best Practices

- Analyze workload requirements before selecting a RAID level.
- Calculate raw and usable capacity during storage planning.
- Never consider RAID a replacement for backups.
- Monitor storage health regularly.
- Validate RAID configuration after deployment.
- Maintain documentation for RAID design and implementation.

---

# Summary

Many RAID-related issues originate during the planning phase rather than during implementation. Proper workload analysis, capacity planning, RAID selection, and validation significantly reduce operational problems. Enterprise storage engineers should always validate RAID configurations and maintain backups to ensure business continuity.
