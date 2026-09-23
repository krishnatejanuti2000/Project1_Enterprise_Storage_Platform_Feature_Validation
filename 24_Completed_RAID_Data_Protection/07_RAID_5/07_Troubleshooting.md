# RAID 5 — Troubleshooting Guide

## 1. Troubleshooting Philosophy

When troubleshooting RAID 5, do not immediately assume that a RAID problem means a disk failure.

A storage engineer should determine:

```text
Symptom
   ↓
RAID State
   ↓
Member State
   ↓
I/O / Performance Behavior
   ↓
Logs / Errors
   ↓
Root Cause
   ↓
Corrective Action
```

The objective is to identify **what failed, where it failed, why it failed, and what state the RAID array is currently in**.

---

# 2. First-Level RAID 5 Health Check

Start with:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

Then inspect the underlying devices:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

These commands provide the initial RAID state.

---

# 3. RAID 5 Health States

Important states include:

```text
Healthy
Degraded
Recovering
Rebuilding
Failed
```

Conceptually:

```text
Healthy
   ↓
Member Failure
   ↓
Degraded
   ↓
Replacement
   ↓
Rebuild
   ↓
Healthy
```

---

# 4. Problem: RAID 5 Is Degraded

## Symptom

`/proc/mdstat` may show something similar to:

```text
[UUU_]
```

or:

```text
[U_UU]
```

The exact position depends on which member is unavailable.

## Investigation

Run:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

Look for:

```text
Active Devices
Working Devices
Failed Devices
Spare Devices
State
```

## Diagnosis

If one member is missing or failed:

```text
RAID 5
   ↓
One member unavailable
   ↓
Degraded operation
```

## Corrective Action

Identify the failed member.

Then replace it appropriately and start rebuild.

---

# 5. Problem: Member Has Failed

## Investigation

Run:

```bash
sudo mdadm --detail /dev/md0
```

Look for a member marked as:

```text
failed
```

Also inspect:

```bash
cat /proc/mdstat
```

Then inspect the underlying disk:

```bash
lsblk
```

If hardware health information is available:

```bash
sudo smartctl -a /dev/sdX
```

## Possible Root Causes

A failed member can be caused by:

* Physical disk failure
* I/O errors
* Controller problems
* Connection problems
* Power problems
* Device timeout
* Media errors
* Hardware instability

Do not assume the disk itself is always the root cause.

---

# 6. Problem: RAID Member Is Missing

A device may disappear completely from the operating system.

Check:

```bash
lsblk
```

Then:

```bash
dmesg | tail -n 100
```

If the device is not visible at the OS level, investigate the storage path before attempting RAID management commands.

Possible causes include:

```text
Disk
 ↓
Cable / Connection
 ↓
Controller
 ↓
Backplane
 ↓
Power
 ↓
Operating System
```

The RAID layer may only be reporting the consequence of a lower-level problem.

---

# 7. Problem: RAID Is Degraded but Disk Appears Healthy

This situation requires careful investigation.

A disk may still be visible to the operating system while RAID has marked it failed.

Check:

```bash
sudo mdadm --detail /dev/md0
```

Then inspect:

```bash
sudo smartctl -a /dev/sdX
```

Also check kernel messages:

```bash
dmesg | grep -iE 'error|fail|reset|timeout|I/O'
```

Possible causes:

* Temporary I/O errors
* Command timeouts
* Device resets
* Communication problems
* Controller issues
* Transient hardware errors
* Actual media problems

A healthy SMART result does not automatically prove that the entire storage path is healthy.

---

# 8. Problem: RAID 5 Rebuild Is Slow

## Symptom

The array is rebuilding, but progress is slow.

Check:

```bash
cat /proc/mdstat
```

For continuous monitoring:

```bash
watch -n 2 cat /proc/mdstat
```

## Investigation

Consider:

* Current application workload
* Disk performance
* Rebuild workload
* I/O contention
* Member capacity
* Disk health
* Rebuild configuration
* System load

During rebuild:

```text
Application I/O
       +
Rebuild I/O
       +
Parity / Reconstruction Work
```

Therefore rebuild activity can compete with normal application I/O.

---

# 9. Problem: RAID 5 Performance Is Slow During Normal Operation

First determine whether the workload is:

```text
Read-heavy
Write-heavy
Sequential
Random
Small-block
Large-block
```

Then inspect RAID state:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

If the array is healthy but writes are slow, investigate the workload pattern.

Small random writes can create additional parity-related work.

---

# 10. Problem: Small Random Writes Are Slow

A small partial write may require:

```text
Read Old Data
Read Old Parity
Calculate New Parity
Write New Data
Write New Parity
```

This is the RMW path.

Conceptually:

```text
Small Write
    ↓
Parity Must Be Updated
    ↓
Additional Reads
    ↓
Parity Calculation
    ↓
Additional Writes
```

Therefore small random writes are an important RAID 5 performance consideration.

---

# 11. Problem: Full-Stripe Writes Perform Better Than Small Writes

This can be expected behavior.

For a full-stripe write:

```text
New A
New B
New C
New D
```

The RAID layer can calculate:

```text
New P = New A XOR New B XOR New C XOR New D
```

The old data and old parity do not need to be read merely to calculate the new parity.

Therefore:

```text
Full-Stripe Write
      ↓
Complete New Data
      ↓
Calculate New Parity
      ↓
Write
```

This avoids the old-data/old-parity read path associated with RMW.

---

# 12. Problem: RAID 5 Rebuild Is Consuming High I/O

This can be expected during rebuild.

The RAID layer needs to:

```text
Read surviving members
      ↓
Reconstruct missing information
      ↓
Write replacement member
```

Therefore rebuild generates substantial I/O.

Check:

```bash
cat /proc/mdstat
```

and:

```bash
sudo mdadm --detail /dev/md0
```

If the application workload is also heavy, overall system performance may decrease.

---

# 13. Problem: RAID 5 Is in Degraded Mode and Application Reads Are Slow

Determine whether the requested data is located on the failed member.

If it is:

```text
Requested Data
      ↓
Failed Member
      ↓
Reconstruction Required
      ↓
Read Surviving Members
      ↓
XOR
      ↓
Return Data
```

This introduces additional work compared with a healthy direct read.

Therefore degraded RAID 5 reads can have different performance characteristics.

---

# 14. Problem: RAID 5 Has a URE During Rebuild

URE means:

```text
Unrecoverable Read Error
```

During rebuild, surviving members must be read.

If a required block cannot be read:

```text
Required Block
      ↓
URE
      ↓
Required Reconstruction Input Missing
```

The affected stripe may not be reconstructable.

## Investigation

Check kernel messages:

```bash
dmesg | grep -iE 'error|I/O|uncorrect|fail'
```

Check disk health:

```bash
sudo smartctl -a /dev/sdX
```

Check RAID state:

```bash
sudo mdadm --detail /dev/md0
```

---

# 15. Problem: Second Member Fails During Rebuild

This is a critical RAID 5 scenario.

Example:

```text
Member 1
   ↓
Fails
   ↓
Rebuild starts
   ↓
Member 2
   ↓
Fails
```

RAID 5 has only one disk-equivalent of redundancy.

Therefore the second failure can exceed the RAID's available redundancy.

The priority becomes:

```text
Protect remaining data
+
Stop additional damage
+
Determine recoverability
```

Do not blindly perform destructive RAID operations.

---

# 16. Problem: Rebuild Does Not Start

First check:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

Check whether a suitable replacement member has actually been added.

If necessary:

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

Then monitor:

```bash
watch -n 2 cat /proc/mdstat
```

If rebuild still does not start, investigate:

* Replacement device state
* RAID member size
* Device availability
* RAID metadata
* Array state
* Kernel messages

---

# 17. Problem: Replacement Disk Is Rejected

Before adding the disk, verify:

```bash
lsblk
```

Check its size.

The replacement must provide sufficient capacity for the RAID member being replaced.

Also check whether it contains old RAID metadata or another partitioning layout.

Inspect:

```bash
sudo mdadm --examine /dev/sdX
```

If the device is intended for reuse, clean it only after confirming that it contains no required data.

---

# 18. Problem: Wrong Disk Identified as Failed

This is a serious operational risk.

Never rely only on:

```text
/dev/sdX
```

names without verification.

Device names can change depending on discovery order.

Before performing:

```bash
--fail
--remove
--add
```

verify the device using:

```bash
lsblk
```

and:

```bash
sudo mdadm --detail /dev/md0
```

Also correlate:

* Device name
* Size
* Serial number
* RAID slot
* Physical location

The objective is:

> **Never remove or fail the wrong disk.**

---

# 19. Problem: RAID Array Shows Unexpected Member State

Use:

```bash
sudo mdadm --detail /dev/md0
```

Pay attention to:

```text
Active Devices
Working Devices
Failed Devices
Spare Devices
```

Example interpretation:

```text
Active Devices = 3
Failed Devices = 1
```

may indicate a degraded array.

A device marked as a spare or rebuilding device has a different role from an active member.

Do not interpret the RAID state from a single field.

---

# 20. Problem: RAID Recovery/Rebuild Appears Stuck

Monitor:

```bash
watch -n 2 cat /proc/mdstat
```

Compare progress over time.

If progress does not change, investigate:

```bash
dmesg | tail -n 100
```

and:

```bash
sudo mdadm --detail /dev/md0
```

Then check the member devices for errors.

Possible causes:

* Failing disk
* I/O errors
* Controller problems
* Severe I/O contention
* Device timeout
* Storage path instability

---

# 21. Problem: RAID Device Is Healthy but Filesystem Is Not Accessible

Remember that RAID and filesystem are separate layers.

Storage stack:

```text
Physical Disks
      ↓
RAID
      ↓
/dev/md0
      ↓
Filesystem
      ↓
Mount Point
```

Check RAID first:

```bash
sudo mdadm --detail /dev/md0
```

Then check filesystem:

```bash
lsblk -f
```

Then check mounts:

```bash
mount
```

A healthy RAID array does not automatically guarantee a healthy filesystem.

---

# 22. Problem: RAID Device Exists but Filesystem Is Missing

Check:

```bash
lsblk -f
```

If `/dev/md0` exists but has no expected filesystem, investigate before creating a new filesystem.

Do **not** immediately run:

```bash
mkfs
```

because formatting an existing RAID device can destroy the filesystem and its data.

---

# 23. Problem: RAID Metadata Confusion

Inspect RAID metadata:

```bash
sudo mdadm --examine /dev/sdX
```

This can help identify:

* RAID level
* Array identity
* Member role
* Metadata information
* RAID membership

Do not modify or erase RAID metadata until the array's state and recovery requirements are understood.

---

# 24. Problem: Disk Has Old RAID Metadata

A replacement or reused disk may contain old RAID metadata.

First inspect:

```bash
sudo mdadm --examine /dev/sdX
```

If the disk is confirmed to be safe to reuse, old metadata can be removed with:

```bash
sudo mdadm --zero-superblock /dev/sdX
```

This is a destructive metadata operation.

Only perform it after confirming that the disk does not belong to a required RAID array.

---

# 25. Problem: RAID 5 Capacity Is Lower Than Expected

Check member sizes:

```bash
lsblk
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

For equal-sized members:

```text
Usable ≈ (N - 1) × Member Size
```

For unequal disks, the smallest effective member size can constrain the usable capacity in a simple configuration.

Therefore:

```text
Large Disk
+
Smaller Disk
        ↓
Unused capacity may exist
```

---

# 26. Problem: One Large Disk Has Unused Capacity

This can be normal.

Example:

```text
4 TB
4 TB
6 TB
6 TB
```

The simple RAID configuration may use approximately:

```text
4 TB per member
```

The additional 2 TB on each larger disk may remain unused.

This is a capacity-planning issue rather than necessarily a RAID fault.

---

# 27. Problem: RAID 5 Has High Latency During Rebuild

Possible reasons include:

```text
Rebuild I/O
+
Application I/O
+
Disk seek / media limitations
+
Controller limitations
+
Storage path contention
```

Investigate:

```bash
cat /proc/mdstat
```

and system I/O behavior.

Do not immediately assume that slow rebuild means a failed RAID implementation.

---

# 28. Problem: RAID 5 Performance Changes After Degradation

This can be expected.

Healthy:

```text
Requested Data
      ↓
Direct Member Read
```

Degraded:

```text
Requested Data
      ↓
Failed Member
      ↓
Reconstruction
      ↓
Multiple Member Reads
      ↓
XOR
      ↓
Data
```

Therefore degraded operation can introduce additional latency and I/O.

---

# 29. Problem: Rebuild Causes Application Performance Degradation

This can be expected because rebuild competes for storage resources.

Investigate:

```text
Application workload
+
Rebuild rate
+
Disk utilization
+
I/O latency
```

The engineering goal is to balance:

```text
Recovery speed
        ↔
Application performance
```

---

# 30. RAID 5 Troubleshooting Decision Flow

```text
RAID 5 Problem
      |
      v
Check /proc/mdstat
      |
      v
Check mdadm --detail
      |
      +----------------------+
      |                      |
   Healthy                 Degraded
      |                      |
      v                      v
Performance?            Identify member
      |                      |
      v                      v
Workload analysis       Check device health
                             |
                             v
                       Replace / recover
                             |
                             v
                           Rebuild
                             |
                             v
                     Verify healthy state
```

---

# 31. Troubleshooting Commands — Quick Reference

## RAID status

```bash
cat /proc/mdstat
```

## Detailed RAID status

```bash
sudo mdadm --detail /dev/md0
```

## Block devices

```bash
lsblk
```

## Detailed block devices

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## RAID metadata

```bash
sudo mdadm --examine /dev/sdX
```

## Disk health

```bash
sudo smartctl -a /dev/sdX
```

## Kernel/storage errors

```bash
dmesg | grep -iE 'error|fail|reset|timeout|I/O'
```

## Monitor rebuild

```bash
watch -n 2 cat /proc/mdstat
```

## Fail member

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdX
```

## Remove member

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdX
```

## Add replacement

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

---

# 32. Troubleshooting Safety Rules

## Rule 1 — Never guess the disk

Always verify the device before:

```text
fail
remove
add
zero-superblock
```

---

## Rule 2 — Never format blindly

Do not run:

```bash
mkfs
```

until you have confirmed that the RAID device does not contain required data.

---

## Rule 3 — Check RAID state before changing anything

Start with:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

---

## Rule 4 — Separate RAID problems from filesystem problems

A RAID device can be healthy while the filesystem has problems.

---

## Rule 5 — Separate RAID problems from hardware-path problems

A RAID member failure can be caused by:

```text
Disk
Controller
Backplane
Cable
Power
Transport
```

Do not automatically blame the disk.

---

## Rule 6 — Protect the remaining members

During degraded operation:

```text
Redundancy is reduced
```

Avoid unnecessary destructive operations.

---

## Rule 7 — Verify recovery

After replacement/rebuild:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

Do not assume that adding a disk means recovery is complete.

---

# 33. Root Cause Analysis Template

When documenting a RAID 5 failure:

```text
Problem:
    What was observed?

Impact:
    What was affected?

RAID State:
    Healthy / Degraded / Recovering / Rebuilding / Failed

Failed Member:
    Which member?

Evidence:
    /proc/mdstat
    mdadm --detail
    dmesg
    SMART

Symptoms:
    What errors or performance changes occurred?

Root Cause:
    Why did the failure occur?

Corrective Action:
    What was done?

Recovery:
    Did rebuild complete?

Final State:
    Is redundancy restored?

Prevention:
    What can prevent recurrence?
```

---

# 34. Example Root Cause Analysis

## Problem

RAID 5 entered degraded state.

## Investigation

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

The RAID output showed one member unavailable.

Then:

```bash
sudo smartctl -a /dev/sdX
```

and:

```bash
dmesg | grep -iE 'error|fail|timeout|I/O'
```

were used to investigate the underlying device.

## Diagnosis

The member was determined to be unavailable and required replacement.

## Corrective Action

```text
Failed member
     ↓
Removed
     ↓
Replacement added
     ↓
Rebuild started
```

## Validation

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

were used to verify the final RAID state.

---

# 35. Key Troubleshooting Principles

1. Start with RAID state.
2. Identify the affected member.
3. Determine whether the member is actually failed or only unavailable.
4. Check the underlying storage path.
5. Check kernel errors.
6. Check disk health when appropriate.
7. Understand whether the array is healthy, degraded, recovering, or rebuilding.
8. Do not confuse reconstruction with rebuild.
9. Understand that degraded reads may require reconstruction.
10. Understand that small writes can have parity-related overhead.
11. Monitor rebuild progress rather than assuming it is complete.
12. Treat UREs seriously during rebuild.
13. Protect the remaining members during degraded operation.
14. Verify replacement-member suitability.
15. Never use destructive commands without confirming the target device.
16. Verify final RAID health after recovery.
17. Separate RAID-layer problems from filesystem-layer problems.
18. Separate RAID-layer problems from hardware-path problems.
19. Document evidence before making destructive changes.
20. Always determine root cause instead of treating the visible RAID symptom as the root cause.



