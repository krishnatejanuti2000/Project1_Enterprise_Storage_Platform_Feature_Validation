# RAID 6 — Troubleshooting Guide

## 1. Purpose

This document provides a structured troubleshooting approach for
RAID 6 failures, degraded states, rebuild problems, parity issues,
and data-access problems.

The objective is to determine:

```text
What failed?
        ↓
How many members failed?
        ↓
How much RAID redundancy remains?
        ↓
Can the array reconstruct the missing information?
        ↓
Can the failed members be rebuilt?
        ↓
Is the array healthy again?
```

---

# 2. RAID 6 Protection Model

RAID 6 normally tolerates:

```text
2 member failures
```

Therefore:

```text
0 failures → Healthy

1 failure → Degraded but protected

2 failures → Degraded and operating at its protection limit

3 failures → Beyond normal RAID 6 fault tolerance
```

A third member failure before recovery/rebuild completes can exceed
the normal protection capability.

---

# 3. First Troubleshooting Step

Always determine the RAID state first.

Check:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

Important information:

```text
RAID Level
Active Devices
Working Devices
Failed Devices
Spare Devices
State
Rebuild/Recovery status
Member states
```

---

# 4. RAID State Interpretation

## Healthy

Example concept:

```text
State : clean
Active Devices : all
Failed Devices : 0
```

Interpretation:

```text
RAID 6 is healthy
Full redundancy available
```

---

## Degraded

Example:

```text
Active Devices < RAID Devices
Failed Devices > 0
```

Interpretation:

```text
One or more members are unavailable
RAID is still operational
Redundancy has been reduced
```

---

## Rebuilding / Recovering

Typical indicators include:

```text
/proc/mdstat
```

showing recovery or rebuild activity.

Interpretation:

```text
Replacement/recovery is in progress
```

Do not declare the array healthy until the rebuild is complete and
the RAID returns to a healthy state.

---

## Failed

If the number of failed members exceeds RAID 6 protection:

```text
RAID 6
+
more than two failed members
        ↓
Protection exceeded
```

The array may become unavailable or unrecoverable through normal
RAID operation.

---

# 5. Troubleshooting Decision Flow

```text
RAID problem detected
        ↓
Check /proc/mdstat
        ↓
Check mdadm --detail
        ↓
Determine number of failed members
        ↓
Check whether data is accessible
        ↓
Check hot spare/replacement status
        ↓
Check rebuild/recovery status
        ↓
Check physical member health
        ↓
Determine recovery path
```

---

# 6. One Data Member Failure

Example:

```text
A    B    C❌    D    P    Q
```

RAID 6 remains within its protection capability.

For a missing data block, P is normally sufficient:

```text
C = A XOR B XOR D XOR P
```

Troubleshooting priorities:

```text
1. Confirm the failed member
2. Confirm RAID remains active/degraded
3. Check data accessibility
4. Check hot spare availability
5. Check rebuild/recovery status
6. Monitor for additional member errors
```

---

# 7. Two Data Members Failure

Example:

```text
A    B    C❌    D❌    P    Q
```

RAID 6 is operating at its normal two-member fault-tolerance
limit.

Recovery requires both parity relationships:

```text
P
+
Q
```

Conceptually:

```text
P → first equation
Q → second equation
        ↓
Recover two missing members
```

Troubleshooting priorities:

```text
1. Do not assume the array is safe from additional failure
2. Confirm exactly two members are unavailable
3. Verify P and Q are available
4. Check data accessibility
5. Identify replacement members
6. Begin/monitor rebuild as appropriate
7. Monitor surviving disks closely
```

---

# 8. Third Member Failure

Example:

```text
A    B    C❌    D❌    P❌    Q
```

or any other three-member failure combination.

This exceeds normal RAID 6 protection.

```text
2-member tolerance
        ↓
Third member failure
        ↓
Protection exceeded
```

At this point:

```text
Do not assume normal RAID reconstruction will succeed.
```

Immediate priorities:

```text
1. Stop unnecessary configuration changes
2. Preserve the current state
3. Record RAID metadata
4. Check backups
5. Escalate according to recovery procedures
```

---

# 9. P Parity Failure

Example:

```text
A    B    C    D    P❌    Q
```

Data remains available because the data members are intact.

Regenerate:

```text
P = A XOR B XOR C XOR D
```

Troubleshooting path:

```text
P missing
   ↓
Confirm all data members are healthy
   ↓
Recalculate/rebuild P
   ↓
Verify RAID returns to healthy state
```

---

# 10. Q Parity Failure

Example:

```text
A    B    C    D    P    Q❌
```

Data remains available.

Q must be regenerated using the RAID 6 Reed–Solomon calculation.

Conceptually:

```text
A B C D
   ↓
Reed–Solomon
   ↓
Galois Field arithmetic
   ↓
Q
```

Troubleshooting path:

```text
Q missing
   ↓
Confirm data members are healthy
   ↓
Regenerate Q
   ↓
Verify RAID state
```

---

# 11. One Data + P Failure

Example:

```text
A    B    C❌    D    P❌    Q
```

Use Q to recover the missing data member.

Conceptually:

```text
Q
↓
Recover C
↓
A B C D available
↓
XOR data
↓
Regenerate P
```

---

# 12. One Data + Q Failure

Example:

```text
A    B    C    D❌    P    Q❌
```

Use P to recover the missing data:

```text
D = A XOR B XOR C XOR P
```

Then regenerate Q:

```text
A B C D
   ↓
Reed–Solomon
   ↓
Q
```

---

# 13. Reconstruction vs Rebuild Troubleshooting

These must not be confused.

## Reconstruction

```text
Missing information
        ↓
Calculate missing information
```

The result can be used to satisfy a host read even before a
replacement member exists.

## Rebuild

```text
Replacement member available
        ↓
Reconstruct missing blocks
        ↓
Write blocks
        ↓
Replacement member restored
```

---

# 14. Host Read During Degraded State

If an application requests data from a missing member:

```text
Application
    ↓
RAID layer
    ↓
Read surviving members
    ↓
Reconstruction
    ↓
Recovered data
    ↓
Application
```

The reconstructed result can be returned to the host without first
having a replacement disk.

---

# 15. Rebuild Troubleshooting

During rebuild, monitor:

```text
/proc/mdstat
```

and:

```bash
sudo mdadm --detail /dev/md0
```

Also monitor:

```text
Rebuild progress
Member errors
Additional failures
Performance impact
Controller/kernel messages
```

Do not declare success until:

```text
Rebuild complete
+
No failed members
+
RAID healthy
```

---

# 16. Rebuild Performance Impact

During rebuild:

```text
Normal application I/O
        +
Rebuild reads
        +
Parity calculations
        +
Replacement writes
```

Therefore:

```text
Rebuild
    ↓
Additional storage workload
    ↓
Potential application performance impact
```

Watch for:

```text
High I/O latency
Long response times
Additional disk errors
Controller warnings
```

---

# 17. Rebuild Window Risk

The rebuild window is the period during which full redundancy has not
yet been restored.

For RAID 6:

```text
Failed members
      ↓
Replacement
      ↓
Rebuild
      ↓
Full redundancy restored
```

Until rebuild completes:

```text
Recovery is incomplete
```

A further failure during recovery must be evaluated against the
remaining RAID protection.

---

# 18. Hot Spare Troubleshooting

Check whether a hot spare exists:

```text
Spare Devices
```

in:

```bash
sudo mdadm --detail /dev/md0
```

Possible states:

```text
Hot spare available
Hot spare already consumed
No hot spare available
Manual replacement required
```

Remember:

```text
Hot spare
→ recovery resource

Hot spare
≠ additional RAID fault tolerance
```

---

# 19. Physical Disk Investigation

When a RAID member fails, investigate the physical disk.

Check:

```text
Disk identity
Disk size
Serial number
Slot
SMART information
Kernel errors
I/O errors
Predictive failure indicators
```

Do not focus only on the failed member.

Also inspect the surviving members for warning signs.

---

# 20. Linux Commands for RAID Investigation

### RAID status

```bash
cat /proc/mdstat
```

### RAID detailed state

```bash
sudo mdadm --detail /dev/md0
```

### Block-device relationship

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

Replace `/dev/sdX` with the appropriate RAID member.

---

# 21. Common Troubleshooting Mistakes

## Mistake 1 — Treating RAID 6 as unlimited fault tolerance

Incorrect:

```text
RAID 6 can survive any number of disk failures
```

Correct:

```text
RAID 6 normally tolerates two member failures.
```

---

## Mistake 2 — Confusing reconstruction with rebuild

Incorrect:

```text
Reconstructing data means the failed disk has been restored.
```

Correct:

```text
Reconstruction → calculate missing information
Rebuild → restore the failed member onto a replacement
```

---

## Mistake 3 — Assuming the hot spare adds another parity level

Incorrect:

```text
RAID 6 + hot spare = three-disk fault tolerance
```

Correct:

```text
Hot spare = standby recovery resource
```

---

## Mistake 4 — Assuming Q is another copy of P

Incorrect:

```text
Q = another XOR copy of P
```

Correct:

```text
P → XOR
Q → Reed–Solomon / Galois Field calculation
```

---

## Mistake 5 — Replacing the wrong physical disk

Before replacement:

```text
Verify RAID member identity
+
Verify physical slot
+
Verify serial number
```

Never replace a disk based only on assumptions about `/dev/sdX`
naming.

---

# 22. Troubleshooting Evidence Collection

When investigating a RAID 6 problem, collect:

```text
RAID status
RAID detail
Physical disk inventory
Member metadata
Failure state
Rebuild state
Kernel/storage errors
Disk health information
Application impact
```

Important evidence commands:

```bash
cat /proc/mdstat

sudo mdadm --detail /dev/md0

lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS

lsblk -f

sudo mdadm --examine /dev/sdX
```

---

# 23. Rebuild Validation

Do not declare recovery successful simply because the rebuild
started.

Validate:

```text
✓ Rebuild reaches 100%
✓ RAID state is healthy
✓ Failed Devices = 0
✓ Active Devices = expected count
✓ Working Devices = expected count
✓ No unexpected member errors
✓ Data remains accessible
✓ Test data matches expected content
```

---

# 24. Troubleshooting Decision Table

| Situation               | Normal Recovery Approach                         |
| ----------------------- | ------------------------------------------------ |
| One data member failed  | P-based reconstruction / rebuild                 |
| Two data members failed | P + Q recovery                                   |
| P failed                | Recalculate P from data                          |
| Q failed                | Recalculate Q from data                          |
| Data + P failed         | Q → recover data → regenerate P                  |
| Data + Q failed         | P → recover data → regenerate Q                  |
| Third member failed     | Protection exceeded; escalate/recovery procedure |
| Hot spare available     | Use as rebuild target according to configuration |
| No hot spare            | Manual replacement may be required               |
| Rebuild active          | Monitor progress and surviving-member health     |
| Rebuild complete        | Validate healthy state and data integrity        |

---

# 25. Troubleshooting Mental Model

```text
             RAID 6 PROBLEM
                    ↓
          Check RAID state
                    ↓
        Count failed members
                    ↓
      ┌─────────────┴─────────────┐
      │                           │
  0–2 failures              >2 failures
      │                           │
  Determine type             Protection
      │                       exceeded
      ↓
Data / P / Q combination
      ↓
Determine surviving parity
      ↓
Reconstruct
      ↓
Replacement
      ↓
Rebuild
      ↓
Validate
      ↓
Healthy
```

# 26. Final Troubleshooting Principle

The most important RAID 6 troubleshooting rule is:

> **First determine exactly what failed and how many members failed before attempting any recovery action.**

Then determine:

```text
What information survives?
        ↓
Which parity relationships remain?
        ↓
Can the missing information be reconstructed?
        ↓
Is a replacement member available?
        ↓
Can the failed member be rebuilt?
        ↓
Did redundancy return to the expected healthy state?
```

