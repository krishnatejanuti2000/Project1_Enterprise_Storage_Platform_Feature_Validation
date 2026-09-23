# RAID 50 — Troubleshooting Guide

## 1. Troubleshooting Objective

RAID 50 troubleshooting must focus on:

* RAID health
* Top-level RAID 0 health
* Component RAID 5 health
* Failed members
* Failure-domain placement
* Degraded state
* Rebuild status
* Data accessibility
* Data integrity
* Final redundancy restoration

The most important RAID 50 troubleshooting question is:

> **Which RAID 5 component contains the failed member, and how many members have failed inside that component?**

RAID 50 must always be analyzed in layers.

```text
Physical Members
      ↓
RAID 5 Components
      ↓
RAID 0 Layer
      ↓
RAID 50
```

---

# 2. First Troubleshooting Step

Always check the overall RAID state first.

```bash
cat /proc/mdstat
```

Then inspect the top-level array:

```bash
sudo mdadm --detail /dev/md50
```

Then inspect both component RAID 5 arrays:

```bash
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Important fields include:

```text
RAID Level
Raid Devices
Total Devices
Layout
Chunk Size
State
Active Devices
Working Devices
Failed Devices
Spare Devices
```

For RAID 50, inspecting only:

```bash
sudo mdadm --detail /dev/md50
```

is not sufficient.

The actual failed physical member may exist inside:

```text
md51
```

or:

```text
md52
```

---

# 3. Healthy RAID 50

A healthy two-component RAID 50 should conceptually show:

```text
md50 → RAID 0 → active

md51 → RAID 5 → [UUU]
md52 → RAID 5 → [UUU]
```

Conceptually:

```text
                 md50
                RAID 0
               /      \
             md51     md52
            RAID 5    RAID 5
             [UUU]     [UUU]
```

Expected condition:

```text
md50 → operational
md51 → clean
md52 → clean
Failed Devices → 0
```

---

# 4. Single-Member Failure

Example:

```text
md51:
D1 → FAILED
D2 → HEALTHY
D3 → HEALTHY
```

Expected state:

```text
md51 → [_UU]
```

The RAID 5 component is degraded but still operational.

If:

```text
md52 → [UUU]
```

then the top-level RAID 0 remains operational.

Conceptually:

```text
                 md50
                RAID 0
               /      \
            md51      md52
            [_UU]     [UUU]
```

The failure path is:

```text
Member failure
      ↓
md51 degraded
      ↓
md51 still operational
      ↓
md50 remains operational
      ↓
Filesystem can remain accessible
```

---

# 5. Single-Member Failure in the Other Component

The same logic applies to `md52`.

Example:

```text
md51 → [UUU]
md52 → [_UU]
```

Both top-level components remain operational.

Therefore:

```text
md50 → operational
```

The key point is that a degraded component is not necessarily a failed
component.

---

# 6. Two-Member Failure in Different Components

Example:

```text
md51 → [_UU]
md52 → [_UU]
```

Each RAID 5 component has one failed member.

Therefore:

```text
md51 → degraded but operational
md52 → degraded but operational
md50 → operational
```

Conceptually:

```text
                  md50
                 RAID 0
                /      \
              md51     md52
              [_UU]    [_UU]
```

This is a survivable distributed failure pattern.

The failures are spread across separate RAID 5 protection domains.

---

# 7. Two-Member Failure in the Same Component

Example:

```text
md51 → [__U]
md52 → [UUU]
```

The RAID 5 component has exceeded its normal single-member fault tolerance.

Therefore:

```text
md51 → unavailable
```

The top-level RAID 0 requires its component arrays to remain available.

Therefore:

```text
md51 unavailable
      ↓
md50 loses one required component
      ↓
RAID 50 unavailable
```

The healthy `md52` cannot reconstruct `md51`.

---

# 8. Why Failure Count Alone Is Not Enough

Incorrect troubleshooting:

```text
Two drives failed
→ RAID 50 should survive
```

Correct troubleshooting:

```text
Two drives failed
      ↓
Where did they fail?
      ↓
Which RAID 5 components are affected?
```

Compare:

```text
1 failure in md51
+
1 failure in md52
```

with:

```text
2 failures in md51
```

These are fundamentally different conditions.

Therefore:

> **RAID 50 failure analysis must consider failure distribution, not only failure count.**

---

# 9. RAID 50 Failure Decision Process

Use this process:

```text
RAID problem
    ↓
Check /proc/mdstat
    ↓
Check md50
    ↓
Check md51 / md52
    ↓
Identify failed member
    ↓
Map member to component RAID 5
    ↓
Count failures in component
    ↓
Determine component health
    ↓
Determine md50 health
    ↓
Check filesystem
    ↓
Check data integrity
    ↓
Replace failed member
    ↓
Monitor rebuild
    ↓
Verify final redundancy
```

---

# 10. Degraded-State Validation

After a member failure:

```bash
cat /proc/mdstat
```

Then inspect the affected component:

```bash
sudo mdadm --detail /dev/md51
```

or:

```bash
sudo mdadm --detail /dev/md52
```

Then inspect the top-level array:

```bash
sudo mdadm --detail /dev/md50
```

Important fields:

```text
State
Active Devices
Working Devices
Failed Devices
Spare Devices
Raid Devices
```

A normal degraded component may show:

```text
State             : clean, degraded
Active Devices    : 2
Working Devices   : 2
Failed Devices    : 1
```

for a three-member RAID 5.

---

# 11. Data Accessibility Validation

RAID metadata alone does not prove application-level availability.

Check the filesystem:

```bash
mount | grep /mnt/raid50
```

Then:

```bash
ls -lh /mnt/raid50
```

Read known test data:

```bash
cat /mnt/raid50/testfile_01.txt
```

Expected validation chain:

```text
RAID degraded
      ↓
Component still operational
      ↓
md50 still active
      ↓
Filesystem still mounted
      ↓
Test files readable
```

---

# 12. Data Integrity Verification

Do not rely only on:

```bash
cat testfile.txt
```

Use checksums.

Healthy-state baseline:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

Record the hashes before fault injection.

Then repeat after:

```text
Failure
Rebuild
Final recovery
```

The validation model is:

```text
Healthy baseline
      ↓
Record SHA-256
      ↓
Inject failure
      ↓
Validate degraded state
      ↓
Validate data access
      ↓
Validate SHA-256
      ↓
Rebuild
      ↓
Validate SHA-256 again
```

Matching hashes provide evidence that the test data remained unchanged.

---

# 13. Rebuild Troubleshooting

Suppose:

```text
md51 → [_UU]
```

A replacement member is available.

First identify the failed member:

```bash
sudo mdadm --detail /dev/md51
```

Remove it if required:

```bash
sudo mdadm --manage /dev/md51 --remove /dev/loopX
```

Verify the replacement:

```bash
sudo mdadm --examine /dev/loopY
```

Expected clean replacement result:

```text
mdadm: No md superblock detected on /dev/loopY.
```

Then add it to the affected component:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loopY
```

or:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loopY
```

---

# 14. RAID 50 Rebuild Process

The rebuild happens inside the affected RAID 5 component.

Conceptually:

```text
Failed member
      ↓
Remove failed member
      ↓
Add replacement
      ↓
RAID 5 reconstruction
      ↓
Recovery
      ↓
RAID 5 component returns healthy
```

For example:

```text
                md50
               RAID 0
              /      \
           md51      md52
           [_UU]     [UUU]
             |
             ↓
         Rebuild here
```

The top-level RAID 0 is not the layer that reconstructs the failed
physical member.

---

# 15. Rebuild Monitoring

Monitor:

```bash
cat /proc/mdstat
```

or:

```bash
watch -n 2 cat /proc/mdstat
```

During recovery, observe:

```text
recovery = XX.X%
```

The affected component should eventually return from:

```text
[_UU]
```

to:

```text
[UUU]
```

Do not stop validation merely because the replacement was successfully added.

---

# 16. Rebuild Completion

A rebuild is complete only after verification.

First:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md51
```

or:

```bash
sudo mdadm --detail /dev/md52
```

Expected:

```text
State             : clean
Active Devices    : 3
Working Devices   : 3
Failed Devices    : 0
```

Then verify the top-level array:

```bash
sudo mdadm --detail /dev/md50
```

Finally verify data:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

---

# 17. Failure During Rebuild

Consider:

```text
md51 → [_UU]
```

and recovery is active.

If another member of `md51` fails during rebuild:

```text
md51 → [__U]
```

the RAID 5 component has exceeded its protection boundary.

Therefore:

```text
md51 → unavailable
      ↓
md50 loses one required component
      ↓
RAID 50 unavailable
```

This is one of the most important RAID 50 operational risks.

---

# 18. Distributed Failure During Rebuild

Consider:

```text
md51 → [_UU]
md52 → [_UU]
```

Both component groups are degraded.

The array can remain operational while both RAID 5 components remain
available.

However:

```text
another failure in md51
```

changes:

```text
md51 → [__U]
```

while:

```text
md52 → [_UU]
```

still remains available.

The result is still determined by the failed component:

```text
md51 → unavailable
      ↓
md50 → unavailable
```

Therefore:

> **Failure location and rebuild state must always be evaluated together.**

---

# 19. Hot Spare Troubleshooting

Inspect:

```bash
sudo mdadm --detail /dev/md50
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Look for:

```text
Spare Devices
```

Possible conditions include:

```text
Hot spare available
No hot spare
Hot spare already active in recovery
```

A hot spare provides a rebuild target.

It does not provide another RAID protection layer.

---

# 20. Replacement Disk Validation

Before adding a replacement:

```bash
sudo mdadm --examine /dev/loopX
```

Expected clean device:

```text
mdadm: No md superblock detected on /dev/loopX.
```

Do not blindly add a replacement disk.

Confirm:

```text
Replacement identity
+
Correct RAID component
+
Correct member size
+
No stale RAID metadata
```

---

# 21. Physical Disk Identification

Never replace a real disk based only on:

```text
/dev/sdb
/dev/sdc
```

Device names can change.

Confirm:

```text
Drive serial number
Drive model
Physical slot
Operating-system device path
RAID membership
RAID component
```

The correct mapping is:

```text
Physical Drive
      ↓
RAID 5 Component
      ↓
RAID 0 Component
      ↓
RAID 50
```

This is especially important in enterprise storage environments.

---

# 22. Filesystem Troubleshooting

If RAID appears healthy but the filesystem is inaccessible, troubleshoot
the layers separately.

```text
Physical member
      ↓
RAID 5 component
      ↓
RAID 0
      ↓
/dev/md50
      ↓
Filesystem
      ↓
Mount
      ↓
Application
```

Useful commands:

```bash
lsblk -f
```

```bash
mount | grep /mnt/raid50
```

```bash
df -h /mnt/raid50
```

Do not automatically conclude that a filesystem problem is a RAID problem.

Possible filesystem-layer problems include:

```text
Filesystem corruption
Mount failure
Permission issue
Incorrect filesystem configuration
Application issue
```

---

# 23. RAID Metadata Troubleshooting

If the physical member appears present but the array is not assembling as
expected, inspect its metadata:

```bash
sudo mdadm --examine /dev/loopX
```

Then compare it with the component array:

```bash
sudo mdadm --detail /dev/md51
```

or:

```bash
sudo mdadm --detail /dev/md52
```

Check:

```text
RAID UUID
Role
Array membership
Events
State
```

The objective is to determine whether the member belongs to the expected
array and role.

---

# 24. Component-Level Troubleshooting

RAID 50 must be troubleshot from the component layer upward.

Use:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Then:

```bash
sudo mdadm --detail /dev/md50
```

Think:

```text
Which member?
      ↓
Which RAID 5?
      ↓
Is RAID 5 healthy?
      ↓
Is md50 healthy?
      ↓
Is filesystem healthy?
```

---

# 25. RAID 50 Failure Matrix

For a two-component RAID 50:

| Failure Pattern | md51                | md52                | RAID 50     |
| --------------- | ------------------- | ------------------- | ----------- |
| 0 + 0           | Healthy             | Healthy             | Operational |
| 1 + 0           | Degraded            | Healthy             | Operational |
| 0 + 1           | Healthy             | Degraded            | Operational |
| 1 + 1           | Degraded            | Degraded            | Operational |
| 2 + 0           | Protection exceeded | Healthy             | Unavailable |
| 0 + 2           | Healthy             | Protection exceeded | Unavailable |

The critical rule is:

```text
Every RAID 5 component must remain operational.
```

---

# 26. Why RAID 50 Is Different from RAID 6

RAID 6 provides:

```text
P + Q
```

and normally tolerates two failed members within one RAID 6 group.

RAID 50 uses:

```text
RAID 5 components
```

and therefore provides single-member protection per component.

Therefore:

```text
RAID 6:
2 failures in one group
→ normally tolerated

RAID 50:
2 failures in same RAID 5 group
→ protection exceeded
```

Do not apply the RAID 6 failure model directly to RAID 50.

---

# 27. Why RAID 50 Is Different from RAID 10

RAID 10 uses:

```text
Mirroring
+
Striping
```

RAID 50 uses:

```text
RAID 5 parity
+
Striping
```

Therefore their failure domains are different.

RAID 10:

```text
Mirror-group survival
```

RAID 50:

```text
RAID 5-component survival
```

For RAID 50, the correct question is:

> Which RAID 5 component contains the failed members?

---

# 28. Rebuild Workload

During RAID 50 rebuild:

```text
Normal application I/O
        +
Reads from surviving RAID 5 members
        +
Parity/data reconstruction
        +
Writes to replacement
```

This can result in:

```text
Higher latency
Lower throughput
Additional disk activity
Longer recovery windows
```

The actual impact depends on:

```text
Workload
Drive performance
RAID configuration
Controller or software implementation
Rebuild policy
```

---

# 29. RAID 50 Rebuild Risk

The critical operational risk is:

```text
Degraded RAID 5
       ↓
Rebuild starts
       ↓
Second member fails
       ↓
RAID 5 protection exceeded
       ↓
Component fails
       ↓
RAID 0 loses component
       ↓
RAID 50 unavailable
```

Therefore:

> **The rebuild window is also a period of reduced protection.**

---

# 30. Data Integrity Troubleshooting

Suppose:

```text
RAID state → clean
Filesystem → mounted
File → readable
```

That does not automatically prove content integrity.

Use:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

Compare against the recorded baseline.

The validation sequence is:

```text
RAID metadata
      +
Filesystem access
      +
Checksum verification
```

A complete storage validation requires all three.

---

# 31. Practical RAID 50 Troubleshooting Flow

Use this flow in an actual incident:

```text
RAID alarm / I/O issue
          ↓
cat /proc/mdstat
          ↓
Identify md50 state
          ↓
Inspect md51 + md52
          ↓
Identify failed physical member
          ↓
Map member to component
          ↓
Count failures in component
          ↓
Determine RAID 5 protection status
          ↓
Determine md50 status
          ↓
Check filesystem
          ↓
Check data accessibility
          ↓
Check data integrity
          ↓
Replace member
          ↓
Monitor recovery
          ↓
Verify clean state
          ↓
Verify application data
```

---

# 32. Common RAID 50 Troubleshooting Mistakes

## Mistake 1 — Counting only failed disks

Incorrect:

```text
Two disks failed → RAID 50 is safe.
```

Correct:

```text
Determine how many failures exist
inside each RAID 5 component.
```

---

## Mistake 2 — Treating RAID 50 as one RAID 5

Incorrect:

```text
RAID 50 is just a larger RAID 5.
```

Correct:

```text
RAID 50 consists of multiple RAID 5 groups
striped by RAID 0.
```

---

## Mistake 3 — Assuming one component can rebuild another

Incorrect:

```text
md51 failed → md52 reconstructs md51.
```

Correct:

```text
Each RAID 5 component has its own data and parity.
There is no cross-component parity.
```

---

## Mistake 4 — Rebuilding the wrong layer

Incorrect:

```text
Rebuild md50 when a physical member fails.
```

Correct:

```text
Identify whether the failed member belongs to md51 or md52.
Rebuild that component.
```

---

## Mistake 5 — Ignoring rebuild state

Incorrect:

```text
RAID is degraded but nothing needs monitoring.
```

Correct:

```text
Track recovery progress and watch for additional failures.
```

---

## Mistake 6 — Assuming rebuild completion from one command

Incorrect:

```text
Replacement added → rebuild finished.
```

Correct:

```text
Recovery complete
+
Component clean
+
Failed devices = 0
+
Top-level array healthy
+
Data integrity verified
```

---

## Mistake 7 — Replacing the wrong physical member

Always verify:

```text
Serial
Model
Physical slot
RAID membership
Component membership
```

before replacing real hardware.

---

# 33. Important Commands

### Overall RAID state

```bash
cat /proc/mdstat
```

### Top-level RAID 50

```bash
sudo mdadm --detail /dev/md50
```

### Component RAID 5 groups

```bash
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

### Member metadata

```bash
sudo mdadm --examine /dev/loopX
```

### Block devices

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

### Filesystem information

```bash
lsblk -f
```

### Mount validation

```bash
mount | grep /mnt/raid50
```

### Capacity validation

```bash
df -h /mnt/raid50
```

### Continuous RAID monitoring

```bash
watch -n 2 cat /proc/mdstat
```

### Data integrity

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

---

# 34. Laboratory Validation Model

The laboratory RAID 50 architecture was:

```text
                    md50
                   RAID 0
                  /      \
                md51      md52
               RAID 5    RAID 5
              /  |  \    /  |  \
          loop10 loop11 loop12 loop13 loop14 loop15
```

The test used:

```text
6 × 1 GiB virtual disk images
```

Filesystem:

```text
ext4
```

Mount point:

```text
/mnt/raid50
```

Known test files were created and SHA-256 values were recorded before
failure testing.

The fresh laboratory validation covered:

```text
Single-member failure in md51
Single-member failure in md52
One failure in each component
Member removal
Replacement member validation
Rebuild
Data-integrity verification
Distributed failure/recovery
Final cleanup
```

---

# 35. RAID 50 Troubleshooting Mental Model

Think about the array as:

```text
                       RAID 50
                          |
                       RAID 0
                     /       \
                    ↓         ↓
                 RAID 5     RAID 5
                  md51       md52
                 / | \       / | \
                D  D  D     D  D  D
```

When a disk fails:

```text
Failed disk
     ↓
Identify component
     ↓
Count failures in component
     ↓
Is component RAID 5 operational?
```

If yes:

```text
Component survives
      ↓
md50 can remain operational
```

If no:

```text
Component fails
      ↓
md50 loses required component
      ↓
RAID 50 becomes unavailable
```

---

# 36. Final Troubleshooting Principle

The most important RAID 50 troubleshooting rule is:

> **Always map every failed physical member to its RAID 5 component before deciding whether RAID 50 can survive.**

Use this reasoning:

```text
Failed member
      ↓
Identify RAID 5 component
      ↓
Count failures in that component
      ↓
Is RAID 5 still operational?
      ↓
YES
 ↓
Check md50
 ↓
Check filesystem
 ↓
Check data integrity
 ↓
Rebuild
 ↓
Verify redundancy

NO
 ↓
RAID 5 component unavailable
 ↓
RAID 0 loses required component
 ↓
RAID 50 unavailable
```

The central engineering rule is:

```text
One failed member in a RAID 5 component
→ degraded but normally operational

Two failed members in the same RAID 5 component
→ RAID 5 protection exceeded

Failed RAID 5 component
→ RAID 50 unavailable
```

> **RAID 50 troubleshooting is fundamentally component-oriented: physical member → RAID 5 protection domain → RAID 0 aggregation → filesystem → application data.**

