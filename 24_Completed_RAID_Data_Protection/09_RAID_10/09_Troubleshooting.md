# RAID 10 — Troubleshooting Guide

## 1. Troubleshooting Objective

RAID 10 troubleshooting must focus on:

- RAID health
- Failed members
- Mirror-group relationships
- Degraded state
- Rebuild status
- Data accessibility
- Final redundancy restoration

The most important RAID 10 troubleshooting question is:

> **Which mirror group is affected, and does that mirror group still have a surviving member?**

---

# 2. First Troubleshooting Step

Always check the RAID state first.

```bash
cat /proc/mdstat
````

Then:

```bash
sudo mdadm --detail /dev/md10
```

Important fields:

```text
RAID Level
Raid Devices
Total Devices
Layout
State
Active Devices
Working Devices
Failed Devices
Spare Devices
```

---

# 3. Healthy RAID 10

A healthy array should show all expected members active.

Conceptually:

```text
[UUUU]
```

For four members:

```text
Active Devices  : 4
Working Devices : 4
Failed Devices  : 0
```

Interpretation:

```text
All mirror groups have complete redundancy.
```

---

# 4. Single-Member Failure

Example:

```text
D1 → FAILED
D2 → HEALTHY
D3 → HEALTHY
D4 → HEALTHY
```

Mirror relationship:

```text
D1 ↔ D2
```

Because D2 survives:

```text
D1 failure
   ↓
D2 still contains the mirror copy
   ↓
Data remains available
```

Expected condition:

```text
DEGRADED
```

but the array can continue operating.

---

# 5. Two-Member Failure — Different Mirror Groups

Example:

```text
D1 → FAILED
D3 → FAILED
```

Survivors:

```text
D2 → HEALTHY
D4 → HEALTHY
```

Mirror groups:

```text
D1 ❌ ↔ D2 ✅
D3 ❌ ↔ D4 ✅
```

Every mirror group still has one surviving member.

Therefore:

```text
Array → operational
Data  → accessible
```

This is a survivable two-member failure pattern.

---

# 6. Two-Member Failure — Same Mirror Group

Example:

```text
D1 → FAILED
D2 → FAILED
```

Then:

```text
D1 ❌ ↔ D2 ❌
```

The complete mirror group is gone.

The remaining mirror group cannot recreate the missing data because
RAID 10 does not use parity.

Therefore:

```text
Mirror group lost
       ↓
Data stored only in that group unavailable
       ↓
RAID 10 protection exceeded for that data
```

This is the critical failure pattern to identify.

---

# 7. RAID 10 Failure Decision Process

Use this process:

```text
RAID problem
    ↓
Check /proc/mdstat
    ↓
Check mdadm --detail
    ↓
Count failed members
    ↓
Identify affected mirror group(s)
    ↓
Check whether each affected group has a surviving member
    ↓
Determine data availability
    ↓
Check replacement/hot spare
    ↓
Rebuild
    ↓
Verify healthy state
```

---

# 8. Why Failure Count Alone Is Not Enough

For RAID 6:

```text
Two failed members
→ normally tolerated
```

For RAID 10:

```text
Two failed members
→ depends on mirror-group placement
```

Therefore never troubleshoot RAID 10 using only:

```text
"How many disks failed?"
```

Also determine:

```text
"Which disks failed?"
```

---

# 9. Degraded-State Validation

After a member failure, verify:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md10
```

Check:

```text
State
Active Devices
Working Devices
Failed Devices
```

Then verify filesystem access.

Example:

```bash
mount | grep /mnt/raid10
```

and:

```bash
ls -l /mnt/raid10
```

---

# 10. Data Integrity Verification

Do not rely only on:

```bash
cat testfile.txt
```

Use checksums.

Example:

```bash
sha256sum /mnt/raid10/testfile_01.txt \
          /mnt/raid10/testfile_02.txt \
          /mnt/raid10/testfile_03.txt
```

Keep a healthy-state baseline before failure testing.

Then compare:

```text
Healthy baseline
      ↓
Failure
      ↓
Degraded verification
      ↓
Rebuild
      ↓
Final verification
```

Matching hashes provide evidence that the test data remained
unchanged.

---

# 11. Rebuild Troubleshooting

Suppose:

```text
D1 → FAILED
D2 → HEALTHY
```

and replacement D1' is available.

Add the replacement:

```bash
sudo mdadm --manage /dev/md10 --add /dev/sdX
```

Then monitor:

```bash
cat /proc/mdstat
```

or:

```bash
watch -n 2 cat /proc/mdstat
```

You should observe recovery/rebuild activity.

---

# 12. RAID 10 Rebuild Process

The rebuild is based on the surviving mirror.

Conceptually:

```text
Failed member
      ↓
Identify surviving mirror
      ↓
Read surviving copy
      ↓
Write replacement
      ↓
Mirror restored
```

Unlike RAID 5/6:

```text
No P parity reconstruction
No Q parity reconstruction
No Reed–Solomon calculation
```

---

# 13. Rebuild Completion

Do not consider the rebuild complete merely because it started.

Verify:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md10
```

Look for:

```text
State             : clean
Active Devices    : expected count
Working Devices   : expected count
Failed Devices    : 0
```

And verify that all members are:

```text
active sync
```

---

# 14. Rebuild Window

During rebuild:

```text
Normal application I/O
        +
Read surviving mirror
        +
Write replacement
```

Therefore rebuild consumes I/O resources.

Possible symptoms:

```text
Higher latency
Lower application throughput
Additional disk activity
Longer request times
```

---

# 15. Failure During Rebuild

Suppose:

```text
D1 → FAILED
D2 → HEALTHY
D3 → FAILED
D4 → HEALTHY
```

Both mirror groups are degraded.

Now D1 is being rebuilt.

If D2 fails before D1's mirror is restored:

```text
D1 ❌
D2 ❌
```

then the first mirror group is completely lost.

Therefore:

```text
Failure during rebuild
        ↓
Check mirror-group placement immediately
```

This is an important RAID 10 operational risk.

---

# 16. Hot Spare Troubleshooting

Check:

```bash
sudo mdadm --detail /dev/md10
```

Look for:

```text
Spare Devices
```

Possible conditions:

```text
Hot spare available
No hot spare
Hot spare already rebuilding
```

A hot spare can provide a rebuild target automatically, depending on
the configuration.

---

# 17. Physical Disk Identification

Never replace a disk based only on:

```text
/dev/sdb
/dev/sdc
```

Device names can change depending on discovery order.

For a real hardware environment, confirm:

```text
Serial number
Drive model
Physical slot
Device path
RAID member identity
```

Useful commands include:

```bash
lsblk
sudo mdadm --detail /dev/md10
sudo mdadm --examine /dev/sdX
```

---

# 18. Filesystem Troubleshooting

If RAID appears healthy but the filesystem is inaccessible, separate
the problem into layers.

```text
Physical disk
    ↓
RAID member
    ↓
/dev/md10
    ↓
Filesystem
    ↓
Mount
    ↓
Application
```

Check:

```bash
lsblk -f
```

Then:

```bash
mount | grep /mnt/raid10
```

If necessary, inspect filesystem-specific errors separately from
RAID errors.

Do not automatically conclude that a filesystem problem is a RAID
problem.

---

# 19. Common RAID 10 Troubleshooting Mistakes

## Mistake 1 — Counting only failed disks

Incorrect:

```text
Two disks failed → RAID 10 is safe
```

Correct:

```text
Identify which mirror groups contain the failures.
```

---

## Mistake 2 — Assuming parity can reconstruct lost data

Incorrect:

```text
RAID 10 can XOR the missing data.
```

Correct:

```text
RAID 10 has no parity.
It depends on surviving mirror copies.
```

---

## Mistake 3 — Confusing RAID 10 with RAID 6

Incorrect:

```text
Two failures are always tolerated.
```

Correct:

```text
Two failures are survivable only if
every affected mirror group retains a member.
```

---

## Mistake 4 — Assuming rebuild is finished because it started

Correct validation requires:

```text
Recovery complete
+
0 failed members
+
all expected active members
+
data-integrity verification
```

---

## Mistake 5 — Replacing the wrong physical disk

Always verify physical identity before replacement.

---

# 20. Important Commands

### RAID state

```bash
cat /proc/mdstat
```

### RAID detail

```bash
sudo mdadm --detail /dev/md10
```

### Block devices

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

### Filesystem information

```bash
lsblk -f
```

### RAID member metadata

```bash
sudo mdadm --examine /dev/sdX
```

### Continuous RAID monitoring

```bash
watch -n 2 cat /proc/mdstat
```

---

# 21. Troubleshooting Decision Table

| Condition                               | Interpretation                     |
| --------------------------------------- | ---------------------------------- |
| One member failed                       | Normally degraded but operational  |
| Two failures in different mirror groups | Can remain operational             |
| Two failures in same mirror group       | Complete mirror group lost         |
| Replacement added                       | Rebuild should begin               |
| Recovery active                         | Monitor until completion           |
| Rebuild complete                        | Verify 0 failed members            |
| Data accessible                         | Verify checksum                    |
| RAID healthy but filesystem unavailable | Investigate filesystem/mount layer |
| Hot spare available                     | Can be used as rebuild target      |

---

# 22. Final Troubleshooting Principle

The most important RAID 10 troubleshooting rule is:

> **Always map every failed member to its mirror group before deciding whether the array can survive.**

Use this reasoning:

```text
Failed member
      ↓
Identify mirror group
      ↓
Is mirror partner alive?
      ↓
YES → data can remain available

NO
 ↓
Entire mirror group lost
 ↓
RAID 10 cannot reconstruct it using parity
```

This is the central troubleshooting model for RAID 10.


