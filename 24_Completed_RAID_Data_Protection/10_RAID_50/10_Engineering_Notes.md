# RAID 50 — Engineering Notes

## 1. Engineering Objective

RAID 50 is designed to combine:

```text
RAID 5
    +
RAID 0
```

The engineering goal is to provide:

```text
Distributed parity protection
+
Striped I/O across multiple RAID 5 groups
+
Higher aggregate storage performance
```

Unlike RAID 0, RAID 50 provides redundancy through the underlying RAID 5
components.

Unlike a single RAID 5 array, RAID 50 distributes the overall workload across
multiple independent RAID 5 groups.

The critical engineering concept is:

> **RAID 50 is a collection of independent RAID 5 protection domains combined
> by a RAID 0 striping layer.**

---

# 2. RAID 50 Architecture

A conceptual RAID 50 architecture is:

```text
                         RAID 50
                            |
                         RAID 0
                      /          \
                     ↓            ↓
                  RAID 5        RAID 5
                   md51          md52
                 /  |  \        /  |  \
                D1  D2  D3      D4 D5 D6
```

The component arrays provide redundancy:

```text
md51 → RAID 5
md52 → RAID 5
```

The top-level array provides striping:

```text
md50 → RAID 0
```

Therefore:

```text
RAID 5
   ↓
Protection

RAID 0
   ↓
Striping

RAID 50
   ↓
Protected striped storage
```

---

# 3. Component RAID Groups

Each RAID 5 group is an independent protection domain.

For example:

```text
Component Group 1
D1 D2 D3
```

forms:

```text
md51
```

and:

```text
Component Group 2
D4 D5 D6
```

forms:

```text
md52
```

The top-level RAID 0 stripes across these component arrays.

Conceptually:

```text
md50
 |
 +---- md51
 |
 +---- md52
```

The engineering implication is very important:

```text
md51 failure
≠
md52 failure
```

Each component must therefore be monitored independently.

---

# 4. RAID 50 Redundancy Mechanism

The redundancy mechanism exists inside each RAID 5 group.

RAID 5 uses:

```text
Data
+
Distributed parity
```

For conceptual parity:

```text
P = D1 XOR D2 XOR D3
```

If one member becomes unavailable, the missing information can be
reconstructed from the remaining data and parity.

RAID 50 does not introduce an additional parity layer above the RAID 5 groups.

Therefore:

```text
md51 parity → protects md51
md52 parity → protects md52
```

There is no cross-group parity.

---

# 5. RAID 50 Has No Global Parity Domain

This is one of the most important engineering distinctions.

RAID 50 is not equivalent to:

```text
One large RAID 5
```

Instead:

```text
RAID 5 Group 1
        +
RAID 5 Group 2
        +
RAID 5 Group 3
        ↓
RAID 0
```

Therefore:

```text
md51 parity
```

cannot reconstruct:

```text
md52 data
```

and:

```text
md52 parity
```

cannot reconstruct:

```text
md51 data
```

Every RAID 5 component must maintain its own protection.

---

# 6. RAID 0 Layer Engineering

The top-level RAID 0 layer:

```text
md50
```

is responsible for striping logical data across the component arrays.

Conceptually:

```text
Logical data:

A B C D E F G H
```

may be distributed as:

```text
md51 → A C E G
md52 → B D F H
```

The exact mapping is implementation-dependent.

The key engineering principle is:

```text
RAID 0
→ distributes logical data across component arrays
```

It does not provide fault tolerance.

---

# 7. RAID 50 Read Path

A logical read enters the top-level RAID layer.

Conceptually:

```text
Host
 ↓
RAID 50
 ↓
RAID 0 layer
 ↓
Selected RAID 5 component
 ↓
Available RAID 5 member
 ↓
Requested data
```

The RAID 5 component services the request.

During healthy operation, reading existing data does not inherently require
parity calculation.

The parity information becomes important when data must be reconstructed
because a member is unavailable.

---

# 8. RAID 50 Write Path

A logical write enters the RAID 0 layer.

The top-level layer determines the target RAID 5 component.

Conceptually:

```text
Host
 ↓
RAID 50
 ↓
RAID 0
 ↓
md51 or md52
 ↓
RAID 5 data update
+
Parity update
 ↓
Physical members
```

Therefore RAID 50 inherits the parity-maintenance characteristics of RAID 5.

There is no parity calculation at the RAID 0 layer.

---

# 9. Small Write Engineering

A partial write to a RAID 5 component may involve parity maintenance.

Conceptually:

```text
New data
    ↓
Affected RAID 5 stripe
    ↓
Update data
    ↓
Update parity
```

The exact implementation may use different RAID 5 write strategies, but
the important engineering point is:

```text
RAID 50
→ still carries RAID 5 parity overhead
```

The top-level RAID 0 striping does not remove the parity-maintenance work.

---

# 10. Chunk Size

The RAID chunk size determines the granularity at which data is distributed
within the RAID implementation.

In the laboratory:

```text
md51 → 512K chunk
md52 → 512K chunk
md50 → 512K chunk
```

The exact relationship between chunk sizes at multiple RAID layers should be
verified from:

```bash
sudo mdadm --detail /dev/md50
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Do not infer RAID geometry only from assumptions.

---

# 11. RAID 50 Capacity Engineering

For equal-sized members, the approximate usable capacity is:

```text
Number of RAID 5 groups
×
(Group members - 1)
×
Member size
```

For two three-member RAID 5 groups:

```text
Group 1:
(3 - 1) × S = 2S

Group 2:
(3 - 1) × S = 2S
```

Therefore:

```text
Total ≈ 4S
```

For:

```text
6 × 1 GiB
```

the approximate RAID 50 usable capacity is:

```text
4 GiB
```

The actual device size is slightly lower because of RAID metadata and
implementation geometry.

---

# 12. Capacity Efficiency

For a RAID 5 group:

```text
Usable capacity
≈
(N - 1) × member size
```

For RAID 50, this calculation is performed independently for each component.

Example:

```text
2 groups
3 members per group
```

Each group contributes:

```text
2 member-equivalents
```

Thus:

```text
6 raw member-equivalents
-
2 parity member-equivalents
=
4 usable member-equivalents
```

This gives approximately:

```text
66.7% usable capacity
```

for this six-member example, ignoring metadata and alignment overhead.

---

# 13. Minimum Practical Member Count

A practical RAID 50 configuration requires multiple RAID 5 groups.

Each RAID 5 group requires at least:

```text
3 members
```

A two-group RAID 50 therefore requires:

```text
6 members
```

Conceptually:

```text
Group 1 → D1 D2 D3
Group 2 → D4 D5 D6
```

More component groups can be added in larger designs.

---

# 14. Single-Member Failure Engineering

Suppose:

```text
md51:
D1 → FAILED
D2 → HEALTHY
D3 → HEALTHY
```

Then:

```text
md51 → degraded
```

but:

```text
md51 → still operational
```

If:

```text
md52 → healthy
```

then:

```text
md50 → operational
```

Therefore:

```text
Single member failure
      ↓
One RAID 5 group degraded
      ↓
Component remains operational
      ↓
Top-level RAID 0 remains operational
```

---

# 15. One Failure in Each Component

Suppose:

```text
md51 → [_UU]
md52 → [U_U]
```

Now each RAID 5 component has one failed member.

Each component remains operational.

Therefore:

```text
md50 → operational
```

This demonstrates that RAID 50 can survive multiple physical failures when
those failures are distributed across different RAID 5 protection domains.

The important engineering factor is:

```text
One failure per component
```

rather than:

```text
Total failures only
```

---

# 16. Two-Member Failure in One Component

Suppose:

```text
md51 → [__U]
md52 → [UUU]
```

The RAID 5 protection boundary of `md51` has been exceeded.

The consequence is:

```text
md51
 ↓
Unavailable
 ↓
md50 loses one RAID 0 component
 ↓
RAID 50 unavailable
```

The healthy `md52` cannot reconstruct the failed `md51` component.

This is the fundamental RAID 50 failure boundary.

---

# 17. Failure Distribution Model

The engineering failure model can be expressed as:

```text
Failure count
+
Failure distribution
```

Example 1:

```text
md51 → 1 failed
md52 → 1 failed
```

Result:

```text
RAID 50 → operational
```

Example 2:

```text
md51 → 2 failed
md52 → 0 failed
```

Result:

```text
md51 → unavailable
RAID 50 → unavailable
```

Therefore:

> **RAID 50 does not have a single global "number of disk failures tolerated"
> value. The component RAID 5 distribution determines the outcome.**

---

# 18. Failure-Domain Isolation

Every physical member belongs to a component RAID 5 group.

Troubleshooting therefore follows:

```text
Physical member
      ↓
RAID 5 component
      ↓
Component health
      ↓
RAID 0 health
      ↓
RAID 50 health
```

The component boundary is the important failure-domain boundary.

For example:

```text
loop10
   ↓
md51
   ↓
RAID 5 Group A
```

while:

```text
loop13
   ↓
md52
   ↓
RAID 5 Group B
```

---

# 19. Rebuild Engineering

When a RAID 5 member fails:

```text
Failed member
      ↓
Replacement member
      ↓
RAID 5 reconstruction
      ↓
Write reconstructed data
      ↓
Component RAID 5 restored
```

The rebuild occurs within:

```text
md51
```

or:

```text
md52
```

depending on where the failure occurred.

The top-level:

```text
md50
```

does not independently rebuild the physical drive.

---

# 20. RAID 50 Reconstruction

For a degraded RAID 5 component, the missing information may be reconstructed
using the remaining members and parity.

Conceptually:

```text
Known data
+
Parity
      ↓
Missing information
```

For a simplified XOR relationship:

```text
P = D1 XOR D2 XOR D3
```

if:

```text
D1
```

is unavailable:

```text
D1 = P XOR D2 XOR D3
```

This is a simplified conceptual model.

Actual RAID 5 implementations may use stripe layouts and parity placement
mechanisms that determine how data is reconstructed across the device.

---

# 21. Rebuild Workload

During rebuild:

```text
Normal application I/O
        +
Reads from surviving members
        +
Parity/data reconstruction
        +
Writes to replacement
```

Therefore rebuild generates additional I/O.

Potential effects include:

```text
Higher latency
Lower application throughput
Increased member activity
Longer recovery windows
```

The actual behavior depends on the workload and implementation.

---

# 22. Rebuild Window

After member failure:

```text
Healthy
   ↓
Failed member
   ↓
Degraded RAID 5
   ↓
Replacement added
   ↓
Recovery
   ↓
Healthy RAID 5
```

During the rebuild window:

```text
The affected RAID 5 component has reduced redundancy.
```

This makes additional failures in the same component especially important.

---

# 23. Failure During Rebuild

Suppose:

```text
md51 → [_UU]
```

and a replacement member is rebuilding.

If another member of `md51` fails before recovery completes:

```text
md51 → [__U]
```

The RAID 5 protection boundary has been exceeded.

Therefore:

```text
md51 → unavailable
md50 → unavailable
RAID 50 → unavailable
```

This is one of the most important operational risks of RAID 50.

---

# 24. Rebuild Monitoring

Use:

```bash
cat /proc/mdstat
```

to monitor the rebuild.

For continuous monitoring:

```bash
watch -n 2 cat /proc/mdstat
```

Look for:

```text
recovery = XX.X%
```

and eventually:

```text
[UUU]
```

for the affected RAID 5 component.

Then validate with:

```bash
sudo mdadm --detail /dev/md51
```

or:

```bash
sudo mdadm --detail /dev/md52
```

---

# 25. Replacement Member Engineering

Before adding a replacement, verify that the device does not contain stale
RAID metadata.

Use:

```bash
sudo mdadm --examine /dev/loopX
```

Expected clean output:

```text
mdadm: No md superblock detected on /dev/loopX.
```

Then add the replacement to the correct component:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loopX
```

or:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loopX
```

Always verify the target RAID component before adding the replacement.

---

# 26. RAID 50 Rebuild Verification

A rebuild should be considered complete only when all of the following are
true:

```text
Recovery complete
+
Component RAID 5 clean
+
Failed devices = 0
+
Expected active members present
+
Top-level RAID 0 healthy
+
Filesystem accessible
+
Data integrity verified
```

Use:

```bash
cat /proc/mdstat
```

then:

```bash
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
sudo mdadm --detail /dev/md50
```

Finally:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

---

# 27. Filesystem Layer

RAID health and filesystem health are separate engineering layers.

The complete path is:

```text
Physical member
      ↓
RAID 5 component
      ↓
RAID 0 layer
      ↓
/dev/md50
      ↓
Filesystem
      ↓
Mount
      ↓
Application
```

A RAID problem can therefore be distinguished from:

```text
Filesystem corruption
Mount failure
Permission issue
Application issue
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

---

# 28. RAID 50 Data-Integrity Engineering

Data availability and data integrity are separate validation dimensions.

A file being readable proves:

```text
Access possible
```

It does not by itself prove:

```text
Content unchanged
```

Therefore use checksums.

Healthy baseline:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

Repeat the same command:

```text
After failure
After rebuild
After final recovery
```

Matching hashes provide evidence that the test data remained unchanged.

---

# 29. RAID 50 Validation Model

A complete validation flow is:

```text
Create component RAID 5 groups
          ↓
Create top-level RAID 0
          ↓
Create filesystem
          ↓
Mount filesystem
          ↓
Write known test data
          ↓
Record checksum baseline
          ↓
Inject failure
          ↓
Validate degraded state
          ↓
Validate data access
          ↓
Validate checksum
          ↓
Replace member
          ↓
Monitor rebuild
          ↓
Validate clean state
          ↓
Validate checksum
```

This tests both:

```text
Functional behavior
+
Data integrity
```

---

# 30. RAID 50 Operational Geometry

The top-level RAID 50 geometry can be thought of as:

```text
              md50
             RAID 0
            /      \
           /        \
        md51        md52
       RAID 5      RAID 5
       / |  \      / |  \
      D1 D2 D3    D4 D5 D6
```

The two layers have different responsibilities:

```text
md50
→ Striping

md51
→ Data protection + parity

md52
→ Data protection + parity
```

The distinction is fundamental when analyzing performance or failures.

---

# 31. RAID 50 Performance Model

RAID 50 combines:

```text
Parallelism from RAID 0
+
Parallelism within multiple RAID 5 groups
```

This can increase aggregate I/O capability compared with a single RAID 5
configuration, depending on workload and implementation.

However, RAID 5 parity operations still exist within the component arrays.

Therefore the performance model is:

```text
More parallel component groups
+
Parity-related work
```

rather than the parity-free model of RAID 10.

---

# 32. RAID 50 vs Single Large RAID 5

A single large RAID 5:

```text
One protection domain
```

RAID 50:

```text
Multiple protection domains
```

Conceptually:

```text
Large RAID 5:

D1 D2 D3 D4 D5 D6


RAID 50:

D1 D2 D3   D4 D5 D6
   RAID 5    RAID 5
       \      /
        RAID 0
```

RAID 50 therefore changes the failure-domain organization.

It does not simply create a larger RAID 5.

---

# 33. RAID 50 vs RAID 6

RAID 6 provides:

```text
Two-member fault tolerance
```

within one RAID 6 group.

RAID 50 instead provides:

```text
Single-member protection
```

within each RAID 5 component.

Therefore:

```text
RAID 6:
2 failures in one group
→ normally tolerated

RAID 50:
2 failures in one RAID 5 component
→ protection exceeded
```

This is a major engineering distinction.

---

# 34. RAID 50 vs RAID 10

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

Conceptually:

```text
RAID 10
→ Copy data
→ Stripe copies

RAID 50
→ Protect with parity
→ Stripe protected groups
```

Their rebuild mechanisms are therefore different.

---

# 35. RAID 50 vs RAID 60

RAID 50:

```text
RAID 5
+
RAID 0
```

RAID 60:

```text
RAID 6
+
RAID 0
```

The most important difference is the protection level of each component.

```text
RAID 50:
1 failed member per RAID 5 component

RAID 60:
2 failed members per RAID 6 component
```

Therefore RAID 60 can survive failure patterns that would exceed a RAID 5
component's protection boundary.

---

# 36. Physical Disk Identification

Never identify a physical disk only by:

```text
/dev/sdb
/dev/sdc
```

Device names can change.

For physical hardware, verify:

```text
Serial number
Drive model
Physical slot
Operating-system device path
RAID membership
RAID component
```

The engineering mapping should be:

```text
Physical drive
      ↓
RAID 5 member
      ↓
RAID 5 component
      ↓
RAID 0 member
      ↓
RAID 50
```

---

# 37. `mdadm` Operational Validation

Important commands:

### Overall state

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

### Continuous monitoring

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

# 38. RAID 50 Engineering Failure Model

Use this mental model:

```text
                         RAID 50
                            |
                         RAID 0
                     /             \
                    ↓               ↓
                 RAID 5           RAID 5
                  md51              md52
                /  |  \           /  |  \
               D1 D2 D3         D4 D5 D6
```

When a drive fails:

```text
Drive failure
      ↓
Identify component
      ↓
Evaluate component RAID 5
      ↓
Is it still operational?
```

If yes:

```text
Component survives
      ↓
RAID 50 can continue
```

If no:

```text
Component fails
      ↓
RAID 0 loses one component
      ↓
RAID 50 unavailable
```

---

# 39. Engineering Responsibilities

A Storage Validation Engineer should validate:

```text
RAID 5 component creation
+
RAID 0 aggregation
+
Capacity accounting
+
Member replacement
+
Degraded operation
+
Rebuild behavior
+
Failure distribution
+
Data accessibility
+
Data integrity
+
Final redundancy restoration
```

A Storage Test Engineer should also verify negative scenarios such as:

```text
Second failure in the same component
Failure during rebuild
Replacement with stale RAID metadata
Filesystem accessibility after degradation
Data consistency after rebuild
```

---

# 40. Key Engineering Takeaways

1. RAID 50 combines multiple RAID 5 groups with RAID 0 striping.
2. RAID 5 provides the redundancy; RAID 0 provides the top-level striping.
3. Each RAID 5 component is an independent protection domain.
4. There is no global parity relationship across RAID 5 components.
5. A single failed member in a component normally leaves that component operational.
6. One failed member in each of multiple components can be survivable.
7. Two failed members in the same RAID 5 component exceed that component's normal protection.
8. A failed component causes the top-level RAID 0 to lose a required member.
9. Failure distribution is more important than total failure count alone.
10. Rebuilds occur inside the affected RAID 5 component.
11. RAID 50 rebuilds involve parity/data reconstruction rather than mirror copying.
12. Rebuild activity generates additional storage I/O.
13. Failure during rebuild must be evaluated within the affected RAID 5 component.
14. Replacement members must be validated before being added.
15. `cat /proc/mdstat` provides the primary real-time RAID status view.
16. `mdadm --detail` should be used to inspect component-level membership and state.
17. Filesystem availability should be validated separately from RAID metadata.
18. SHA-256 checksums provide stronger data-integrity evidence than simple file reads.
19. Physical disk identity should be confirmed before replacing real hardware.
20. RAID 50's effective failure tolerance depends on how failures are distributed across its component RAID 5 groups.

---

# 41. Final Engineering Principle

> **Always troubleshoot RAID 50 from the inside out: physical member → RAID 5 component → RAID 0 layer → filesystem.**

The fundamental mental model is:

```text
Failed member
      ↓
Which RAID 5 group?
      ↓
How many failures in that group?
      ↓
Is that RAID 5 still operational?
      ↓
Is md50 still operational?
      ↓
Is the filesystem accessible?
      ↓
Is the data intact?
      ↓
Can the failed member be rebuilt?
      ↓
Is redundancy restored?
```

The central rule is:

```text
One failure in a RAID 5 component
→ degraded but normally operational

Two failures in the same RAID 5 component
→ RAID 5 protection exceeded

Failed RAID 5 component
→ RAID 0 loses a required component

Therefore:
→ RAID 50 becomes unavailable
```

This component-level failure-domain model is the foundation for engineering,
testing, troubleshooting, and validating RAID 50.

