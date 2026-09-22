# RAID 60 — Troubleshooting Guide

## 1. Troubleshooting Objective

The objective of RAID 60 troubleshooting is not simply to determine
whether a disk has failed.

The objective is to determine:

```text
Which member failed?
        ↓
Which RAID 6 component contains the failure?
        ↓
How many members are failed in that component?
        ↓
Is that RAID 6 group still within its protection limit?
        ↓
Is the top-level RAID 0 layer still operational?
        ↓
Is user data accessible?
        ↓
Can the failed member be rebuilt safely?
        ↓
Has full redundancy been restored?
```

RAID 60 must therefore be diagnosed layer by layer.

The three important layers in the Linux laboratory configuration are:

```text
md60
RAID 0
   ↓
md61 + md62
RAID 6 + RAID 6
   ↓
Physical members
```

The most important troubleshooting principle is:

> Never evaluate RAID 60 only by total failed-disk count. Evaluate the
> failure distribution across every component RAID 6 group.

---

# 2. First Troubleshooting Step

The first action is to determine the current RAID hierarchy and health.

Start with:

```bash
cat /proc/mdstat
```

Then inspect each RAID layer:

```bash
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
sudo mdadm --detail /dev/md60
```

The goal is to establish:

```text
RAID level
RAID devices
Active devices
Working devices
Failed devices
State
Rebuild status
Member identity
```

Do not immediately replace a disk before understanding the RAID
topology.

---

# 3. Healthy RAID 60

A healthy RAID 60 should show:

```text
md61 → RAID 6 → Healthy
md62 → RAID 6 → Healthy
md60 → RAID 0 → Healthy
```

Conceptually:

```text
                  RAID 60
                    |
                   md60
                    |
           ┌────────┴────────┐
           ↓                 ↓
         md61              md62
        RAID 6             RAID 6
       [UUUU]              [UUUU]
```

A healthy component array should have:

```text
Active Devices = expected
Working Devices = expected
Failed Devices = 0
```

For a four-member component RAID 6:

```text
[4/4] [UUUU]
```

indicates all four members are active.

The top-level array must also be inspected.

Do not assume that healthy child arrays automatically prove that every
layer is healthy.

---

# 4. Single-Member Failure

Suppose one member of `md61` fails:

```text
md61
D1 ❌
D2 ✅
D3 ✅
D4 ✅
```

The RAID 6 component becomes degraded.

The expected state is conceptually:

```text
[4/3] [_UUU]
```

The important checks are:

```text
Is md61 still operational?
Is md62 healthy?
Is md60 still active?
Is the filesystem accessible?
Is user data readable?
```

A single-member failure should not normally make RAID 60 unavailable
because RAID 6 provides protection against the loss of one member.

---

# 5. Two-Member Failure in the Same RAID 6 Group

This is a more serious degraded state.

Example:

```text
md61

D1 ❌
D2 ❌
D3 ✅
D4 ✅
```

Conceptually:

```text
[4/2] [__UU]
```

The affected RAID 6 component has two failed members.

This is still within RAID 6's normal dual-member fault tolerance.

Therefore:

```text
md61 → degraded but operational
md62 → healthy
md60 → active
```

The system should remain accessible, but the component has exhausted
its normal two-member failure protection.

Immediate recovery should become the priority.

---

# 6. One Failure in Each RAID 6 Group

Consider:

```text
md61:
D1 ❌
D2 ✅
D3 ✅
D4 ✅

md62:
D5 ❌
D6 ✅
D7 ✅
D8 ✅
```

Now both component arrays are degraded.

Conceptually:

```text
           md60
          RAID 0
        /         \
       /           \
    md61            md62
   [_UUU]          [_UUU]
```

This is a critical engineering state because:

```text
Failure in md61
+
Failure in md62
```

means redundancy has been reduced independently in both failure
domains.

The array can remain accessible because each RAID 6 component still has
enough surviving members.

However, the remaining protection must be restored promptly.

---

# 7. Failure Distribution Matters More Than Failure Count

RAID 60 failure analysis must always use:

```text
Failure count
+
Failure location
```

Consider four failed members.

### Scenario A — 2 + 2

```text
md61 → 2 failures
md62 → 2 failures
```

Both component RAID 6 arrays remain within dual-failure tolerance.

Therefore the top-level RAID 60 can remain operational.

### Scenario B — 3 + 1

```text
md61 → 3 failures
md62 → 1 failure
```

The first RAID 6 component has exceeded its normal fault tolerance.

Therefore the RAID 60 can become unavailable even though the total
number of failed drives is the same.

The troubleshooting question is therefore:

```text
How many failed members are in each RAID 6 group?
```

not simply:

```text
How many failed drives exist?
```

---

# 8. Why Failure Count Alone Is Insufficient

The following examples demonstrate the failure-domain model:

```text
1 + 0
→ Both groups operational

1 + 1
→ Both groups degraded

2 + 0
→ One group has exhausted dual-failure tolerance

2 + 1
→ One group remains protected, but is heavily degraded

2 + 2
→ Both groups are at their normal maximum degraded-member state

3 + 1
→ One group exceeds RAID 6 protection capability
```

Therefore:

```text
Same total failure count
        ≠
Same RAID 60 outcome
```

The distribution across component RAID 6 groups determines the result.

---

# 9. Degraded-State Validation

When a member fails, verify the state using:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md61
```

or:

```bash
sudo mdadm --detail /dev/md62
```

depending on which component is affected.

Verify:

```text
State
Active Devices
Working Devices
Failed Devices
Device Role
Device State
```

For example:

```text
State : clean, degraded
Active Devices : 3
Working Devices : 3
Failed Devices : 1
```

This confirms that the component RAID 6 is operating in degraded mode.

Then inspect the parent:

```bash
sudo mdadm --detail /dev/md60
```

The top-level state must also be verified.

---

# 10. Validate Data Accessibility

RAID health alone is not enough.

The next step is to verify the filesystem and actual user data.

For example:

```bash
mount | grep raid60
```

Then:

```bash
ls -lh /mnt/raid60
```

Read representative files:

```bash
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

Expected result during a survivable degraded state:

```text
Filesystem accessible
+
Files readable
+
Data contents unchanged
```

A degraded RAID that remains mounted and readable is still an
incident requiring recovery.

---

# 11. Data Integrity Verification

Availability does not automatically prove data integrity.

Before failure injection, record checksums:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

After failure:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

After rebuild:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

Compare:

```text
Healthy baseline
      =
After failure
      =
After rebuild
```

For the RAID60 laboratory, the test files retained their original
SHA-256 values through the tested failure and recovery scenarios,
including the 2+2 failure state.

This provides stronger evidence than simply checking whether the mount
point remains accessible.

---

# 12. Rebuild Troubleshooting

After identifying and safely removing a failed member, a replacement
member can be added.

Typical workflow:

```text
Failed member
      ↓
Verify failure
      ↓
Remove failed member
      ↓
Prepare replacement
      ↓
Add replacement
      ↓
Rebuild
      ↓
Monitor progress
      ↓
Verify healthy state
```

Example:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loopXX
```

Then:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loopYY
```

Monitor:

```bash
cat /proc/mdstat
```

The rebuild is complete only when the expected member count is
restored.

For a four-member RAID 6 component:

```text
[4/4] [UUUU]
```

should be observed after successful recovery.

---

# 13. Rebuild Verification

During rebuild, monitor:

```bash
watch cat /proc/mdstat
```

Look for:

```text
recovery
resync
rebuild progress
completion percentage
```

After completion:

```bash
sudo mdadm --detail /dev/md61
```

or:

```bash
sudo mdadm --detail /dev/md62
```

Confirm:

```text
State = clean
Active Devices = expected
Working Devices = expected
Failed Devices = 0
```

Then validate the parent:

```bash
sudo mdadm --detail /dev/md60
```

Finally verify data:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

---

# 14. Rebuild Window

The rebuild period is an important risk window.

Example:

```text
Before failure

md61
[UUUU]


After one failure

md61
[_UUU]


During rebuild

md61
[_UUU]
recovery = active


After rebuild

md61
[UUUU]
```

During this period:

```text
Protection is reduced
+
Storage I/O is active
+
Another hardware failure may occur
```

Therefore the engineer should:

```text
Monitor rebuild
+
Monitor remaining members
+
Monitor system errors
+
Avoid unnecessary configuration changes
```

A successful rebuild should be treated as a restoration of redundancy,
not merely a percentage reaching 100%.

---

# 15. Second Failure During Rebuild

This is a critical troubleshooting scenario.

Suppose:

```text
md61
D1 ❌
D2 → rebuilding
D3 ✅
D4 ✅
```

If another member fails:

```text
D3 ❌
```

the component now has two failed members.

RAID 6 can normally tolerate this condition.

However, the situation is substantially more degraded.

The engineer must immediately determine:

```text
Which component failed?
Current failed count?
Current rebuild state?
Remaining active members?
Any additional hardware warnings?
```

A further failure could exceed the component's normal protection
capability.

---

# 16. Rebuild Failure

A rebuild may fail or stop for reasons such as:

```text
Replacement member unavailable
Replacement size insufficient
Additional member failure
I/O errors
Media errors
Controller problems
Configuration mismatch
```

The troubleshooting sequence should be:

```text
Check /proc/mdstat
        ↓
Check mdadm --detail
        ↓
Identify failed/rebuild member
        ↓
Check kernel/storage errors
        ↓
Validate replacement member
        ↓
Determine whether another member is failing
```

Do not repeatedly remove and add devices without understanding why the
rebuild is failing.

---

# 17. Physical Disk Identification

The RAID layer identifies a member logically, but the engineer may need
to identify the actual physical device.

Use:

```bash
lsblk
```

and:

```bash
sudo mdadm --detail /dev/md61
```

For physical environments, correlate:

```text
RAID member
      ↓
Device path
      ↓
Serial number
      ↓
Enclosure slot
      ↓
Physical drive
```

Additional information may be available with:

```bash
udevadm info --query=all --name=/dev/sdX
```

or platform-specific storage-management utilities.

Never replace a physical disk based only on an assumed device name.

---

# 18. Remaining-Member Health

When one member fails, do not investigate only the failed disk.

Check the remaining members.

Look for:

```text
Media errors
Predictive failure
I/O errors
SMART warnings
Controller alerts
Repeated timeouts
```

This is especially important during RAID 60 rebuild because another
weak member can fail while the component is already degraded.

The practical mindset is:

```text
Failed member
+
Surviving members
+
Replacement member
```

all require evaluation.

---

# 19. Hot Spare Troubleshooting

If a hot spare is configured, verify:

```text
Hot Spare available?
        ↓
Correct size?
        ↓
Compatible?
        ↓
Automatically assigned?
        ↓
Rebuild started?
```

A hot spare should be treated as a recovery resource.

It does not create an additional RAID protection level.

Conceptually:

```text
Failed member
      ↓
Hot spare
      ↓
RAID 6 reconstruction
      ↓
Healthy component
```

After the rebuild, verify whether the spare pool still contains the
required standby capacity.

---

# 20. Filesystem Troubleshooting

A RAID 60 incident does not automatically mean that the filesystem is
corrupt.

Separate the layers:

```text
Physical device
      ↓
RAID 6 component
      ↓
RAID 0 layer
      ↓
Filesystem
      ↓
Files
```

If the RAID remains operational and files are readable:

```text
Do not immediately run filesystem repair tools.
```

First determine whether the problem is actually at the filesystem
layer.

Useful checks include:

```bash
mount | grep raid60
```

```bash
df -h /mnt/raid60
```

```bash
dmesg | tail -n 50
```

Filesystem repair should follow the appropriate recovery procedure
rather than being used as a first response to every RAID issue.

---

# 21. Kernel and System Log Validation

When RAID behavior is unexpected, inspect system logs.

For example:

```bash
dmesg | grep -Ei 'md|raid|I/O|error|fail|sd[a-z]'
```

Also inspect recent kernel messages:

```bash
dmesg | tail -n 100
```

The objective is to determine whether the RAID event is associated
with:

```text
I/O failure
Device timeout
Medium error
Reset
Controller issue
Disk failure
```

Logs should be correlated with the exact timestamp of the failure.

---

# 22. RAID 60 Top-Level Troubleshooting

The top-level RAID 0 layer must also be checked.

Inspect:

```bash
sudo mdadm --detail /dev/md60
```

Expected healthy architecture:

```text
md60
 |
 +-- md61
 |
 +-- md62
```

A healthy child:

```text
md61 → operational
```

does not replace the need to validate:

```text
md60 → operational
```

The top-level layer determines whether the logical RAID 60 device remains
available to the filesystem.

---

# 23. Component RAID 6 Failure

This is the most serious RAID 60 condition.

Example:

```text
md61 → unavailable
md62 → healthy
```

The outer RAID 0 layer cannot recreate the missing component.

RAID 0 has no redundancy mechanism.

Therefore:

```text
md61 ❌
md62 ✅
md60
 ↓
Top-level RAID 60 unavailable
```

The correct response is to determine why the component RAID 6 failed
and follow the platform's recovery procedure.

Do not assume the healthy second component can reconstruct the lost
component.

---

# 24. Maximum Tested 2 + 2 Failure Scenario

The RAID60 laboratory explicitly validated the maximum distributed
failure pattern for its two 4-member RAID 6 components:

```text
md61 → 2 failed members
md62 → 2 failed members
```

The resulting state was:

```text
md61 → [4/2] [__UU]
md62 → [4/2] [__UU]
```

At this point:

```text
Both RAID 6 components degraded
+
Top-level md60 active
+
Filesystem accessible
+
Test files readable
+
SHA-256 values unchanged
```

This is important evidence because the same total of four failed
members would not necessarily be survivable if distributed differently.

---

# 25. Recovery of a 2 + 2 Failure

The recovery should be controlled.

The tested recovery sequence was:

```text
2 failures in md61
        ↓
Replace/rebuild one member
        ↓
md61 partially restored
        ↓
Replace/rebuild second member
        ↓
md61 healthy

2 failures in md62
        ↓
Replace/rebuild one member
        ↓
md62 partially restored
        ↓
Replace/rebuild second member
        ↓
md62 healthy
```

The reason for sequential recovery is to keep the failure state clearly
observable and verify each rebuild before progressing.

After full recovery:

```text
md61 → [4/4] [UUUU]
md62 → [4/4] [UUUU]
md60 → clean
```

Then verify:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

---

# 26. Common Troubleshooting Mistakes

## Mistake 1 — Counting only total failed disks

Wrong:

```text
4 drives failed
→ RAID 60 must be dead
```

Correct:

```text
Determine failure distribution by RAID 6 group.
```

---

## Mistake 2 — Assuming RAID 0 provides redundancy

Wrong:

```text
Outer RAID 0 will recover the failed RAID 6 group.
```

Correct:

```text
RAID 0 provides striping only.
```

---

## Mistake 3 — Replacing the wrong physical disk

Always correlate:

```text
RAID member
+
device
+
slot
+
serial number
```

before physical replacement.

---

## Mistake 4 — Ignoring surviving-member health

During rebuild, a second weak member can create a much more serious
failure.

Always inspect the complete component group.

---

## Mistake 5 — Declaring success because the filesystem is mounted

Mounted filesystem:

```text
≠
Complete validation
```

You should also verify:

```text
RAID health
+
Rebuild completion
+
Failed device count
+
File access
+
Checksum integrity
```

---

## Mistake 6 — Ignoring the parent RAID layer

A healthy `md61` does not prove that `md60` is healthy.

Always validate:

```text
Component RAID 6
+
Top-level RAID 0
```

---

## Mistake 7 — Treating rebuild completion as the only success criterion

A rebuild reaching 100% is not enough.

Also verify:

```text
State = clean
Failed Devices = 0
Expected members present
Filesystem accessible
Data integrity verified
```

---

# 27. Important Commands

### RAID overview

```bash
cat /proc/mdstat
```

### Component RAID 6 details

```bash
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
```

### Top-level RAID 0 details

```bash
sudo mdadm --detail /dev/md60
```

### Block-device inventory

```bash
lsblk
```

### Mount validation

```bash
mount | grep raid60
```

### Capacity validation

```bash
df -h /mnt/raid60
```

### File validation

```bash
ls -lh /mnt/raid60
```

### Data readability

```bash
cat /mnt/raid60/testfile_01.txt
```

### Checksum validation

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

### Kernel messages

```bash
dmesg | tail -n 100
```

### Search storage/RAID errors

```bash
dmesg | grep -Ei 'md|raid|I/O|error|fail'
```

### Remove failed member

```bash
sudo mdadm --manage /dev/md61 --remove /dev/DEVICE
```

### Add replacement member

```bash
sudo mdadm --manage /dev/md61 --add /dev/DEVICE
```

---

# 28. RAID 60 Failure Decision Process

Use the following decision model:

```text
                RAID 60 Alert
                      |
                      ▼
             Check /proc/mdstat
                      |
                      ▼
             Identify RAID layer
                      |
          ┌───────────┴───────────┐
          ↓                       ↓
      md61 failed?            md62 failed?
          |                       |
          └───────────┬───────────┘
                      ↓
          Identify failed members
                      |
                      ▼
       Map members to RAID 6 group
                      |
                      ▼
       Count failures per component
                      |
          ┌───────────┴───────────┐
          ↓                       ↓
       ≤ 2/group              > 2/group
          |                       |
          ▼                       ▼
     Group survives        Component may fail
          |                       |
          ▼                       ▼
    Validate data          Escalate recovery
          |
          ▼
      Replace member
          |
          ▼
       Rebuild
          |
          ▼
   Verify [UUUU] / clean
          |
          ▼
    Validate md60
          |
          ▼
  Validate filesystem
          |
          ▼
  Validate data integrity
```

---

# 29. RAID 60 Troubleshooting Decision Table

| Failure Pattern | RAID 6 Group 1 | RAID 6 Group 2 | Expected Protection State          |
| --------------- | -------------: | -------------: | ---------------------------------- |
| 1 + 0           |       1 failed |       0 failed | Group 1 degraded, operational      |
| 0 + 1           |       0 failed |       1 failed | Group 2 degraded, operational      |
| 2 + 0           |       2 failed |       0 failed | Group 1 at dual-failure limit      |
| 0 + 2           |       0 failed |       2 failed | Group 2 at dual-failure limit      |
| 1 + 1           |       1 failed |       1 failed | Both groups degraded               |
| 2 + 1           |       2 failed |       1 failed | Both operational, heavily degraded |
| 1 + 2           |       1 failed |       2 failed | Both operational, heavily degraded |
| 2 + 2           |       2 failed |       2 failed | Maximum tested distributed failure |
| 3 + 0           |       3 failed |       0 failed | Group 1 exceeds normal tolerance   |
| 0 + 3           |       0 failed |       3 failed | Group 2 exceeds normal tolerance   |
| 3 + 1           |       3 failed |       1 failed | One group exceeds tolerance        |
| 1 + 3           |       1 failed |       3 failed | One group exceeds tolerance        |

The table describes the RAID 6 component failure model; actual platform
behavior should always be verified from RAID status and implementation
details.

---

# 30. Validation After Fix

After recovery, verify every layer.

### Component RAID 6

```bash
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
```

Confirm:

```text
State = clean
Active Devices = expected
Working Devices = expected
Failed Devices = 0
```

### Top-level RAID 0

```bash
sudo mdadm --detail /dev/md60
```

Confirm:

```text
Healthy
Expected component members
Failed Devices = 0
```

### Filesystem

```bash
mount | grep raid60
```

### Capacity

```bash
df -h /mnt/raid60
```

### Data

```bash
ls -lh /mnt/raid60
```

### Integrity

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

The final checksum values should match the healthy-state baseline.

---

# 31. Final Troubleshooting Workflow

The complete RAID 60 troubleshooting workflow is:

```text
RAID Alert
   ↓
Check /proc/mdstat
   ↓
Identify affected component
   ↓
Inspect md61 / md62
   ↓
Inspect md60
   ↓
Map failed members
   ↓
Count failures per RAID 6 group
   ↓
Determine remaining protection
   ↓
Validate filesystem
   ↓
Validate data
   ↓
Validate checksums
   ↓
Identify replacement
   ↓
Replace/add member
   ↓
Monitor rebuild
   ↓
Verify component returns healthy
   ↓
Verify top-level RAID 60
   ↓
Verify filesystem
   ↓
Verify data integrity
   ↓
Close incident only after redundancy is restored
```

---

# 32. Root Cause Analysis Model

When documenting a RAID 60 failure, separate:

```text
Symptom
   ↓
Technical cause
   ↓
Contributing factor
   ↓
Resolution
   ↓
Validation
```

Example:

### Symptom

```text
RAID 6 component entered degraded state.
```

### Technical Cause

```text
Physical storage member failed.
```

### Contributing Factors

Possible contributors include:

```text
Media degradation
Predictive failure
Controller issue
Power event
Link problem
Firmware issue
```

### Resolution

```text
Failed member removed
+
Replacement added
+
Rebuild completed
```

### Validation

```text
Component healthy
+
Top-level RAID healthy
+
Filesystem accessible
+
Checksums match
```

A good RCA should be based on actual evidence rather than assumption.

---

# 33. Final Troubleshooting Principle

RAID 60 troubleshooting must always be performed at three levels:

```text
                 RAID 60
                    |
                    ▼
              RAID 0 layer
                    |
          ┌─────────┴─────────┐
          ↓                   ↓
       RAID 6               RAID 6
       Group 1              Group 2
          |                   |
       Members             Members
```

The engineer must continuously ask:

```text
Which member failed?
        ↓
Which RAID 6 group?
        ↓
How many failures in that group?
        ↓
Is it within dual-failure tolerance?
        ↓
Is the top-level RAID 0 still operational?
        ↓
Is user data accessible?
        ↓
Is data integrity preserved?
        ↓
Can the member be rebuilt?
        ↓
Has full redundancy returned?
```

The central RAID 60 troubleshooting rule is:

> Failure count alone is never enough. Failure distribution across the
> component RAID 6 groups determines the actual protection state.

And the final success condition is:

```text
RAID components healthy
        +
Top-level RAID healthy
        +
Filesystem healthy
        +
Data accessible
        +
Data integrity verified
        +
Redundancy fully restored
```

Only then should the RAID 60 incident be considered completely
resolved.

