# RAID 60 — Engineering Notes

## 1. Engineering Objective

RAID 60 is designed to combine:

```text
RAID 6
    +
RAID 0
```

The engineering goal is to provide:

```text
Dual-parity protection
+
Multiple independent RAID 6 protection domains
+
Top-level striping
+
Higher aggregate parallelism
```

The fundamental engineering concept is:

> **RAID 60 is a set of independent RAID 6 groups combined by a RAID 0 layer.**

Therefore the protection mechanism exists inside each RAID 6 component, while
the top-level RAID 0 provides striping.

---

# 2. RAID 60 Architecture

A conceptual RAID 60 architecture is:

```text
                         RAID 60
                            |
                         RAID 0
                     /      |      \
                    /       |       \
                 RAID 6   RAID 6   RAID 6
```

For the laboratory configuration:

```text
                  md60
                 RAID 0
                /      \
              md61     md62
             RAID 6    RAID 6
            / | | \    / | | \
```

The hierarchy was:

```text
md60 → top-level RAID 0
md61 → RAID 6 component
md62 → RAID 6 component
```

The fresh laboratory began with:

```text
md61:
loop10
loop11
loop12
loop13

md62:
loop14
loop15
loop16
loop17
```

Replacement members were introduced during the recovery tests.

---

# 3. Component RAID 6 Groups

Each RAID 6 component is an independent protection domain.

Conceptually:

```text
Group A
D1 D2 D3 D4
   ↓
 RAID 6
```

and:

```text
Group B
D5 D6 D7 D8
   ↓
 RAID 6
```

The top-level RAID 0 combines the component arrays:

```text
md60
 |
 +---- md61
 |
 +---- md62
```

The engineering consequence is:

```text
md61 failure
≠
md62 failure
```

Each component must therefore be monitored independently.

---

# 4. Dual-Parity Protection

RAID 6 provides two independent parity relationships:

```text
P parity
+
Q parity
```

Conceptually:

```text
Data
+
P
+
Q
```

The purpose is to provide:

```text
Two-member fault tolerance
```

inside the same RAID 6 component.

This is the primary protection difference between RAID 60 and RAID 50.

---

# 5. P Parity

P parity is conceptually based on XOR relationships.

For example:

```text
P = D1 XOR D2 XOR D3
```

If one data member becomes unavailable, the missing information can be
reconstructed using the remaining data and parity.

This is a simplified engineering model.

Actual RAID 6 implementations operate on stripes and parity placement according
to the RAID implementation.

---

# 6. Q Parity

Q parity provides a second independent parity relationship.

Unlike simple XOR parity, Q parity uses a different coding relationship.

Conceptually:

```text
P → first parity relationship
Q → second independent parity relationship
```

This additional parity allows RAID 6 to reconstruct data when two members in the
same RAID 6 protection domain are unavailable.

The important engineering principle is:

```text
P + Q
→ two independent recovery relationships
→ two-member protection
```

---

# 7. RAID 60 Has No Global Parity

RAID 60 does not create one global P/Q parity domain across all physical
members.

Instead:

```text
md61
→ own data + P + Q

md62
→ own data + P + Q
```

Therefore:

```text
md61 parity
```

does not reconstruct:

```text
md62 data
```

and:

```text
md62 parity
```

does not reconstruct:

```text
md61 data
```

The component boundary is fundamental.

---

# 8. RAID 0 Layer Engineering

The top-level:

```text
md60
```

is RAID 0.

Its responsibility is:

```text
Striping
```

It distributes logical data across:

```text
md61
+
md62
```

Conceptually:

```text
Logical data
A B C D E F G H

md61 → A C E G
md62 → B D F H
```

The exact mapping depends on the RAID implementation and geometry.

The important engineering distinction is:

```text
md60
→ striping

md61 / md62
→ protection
```

---

# 9. RAID 60 Read Path

The logical read path is:

```text
Host
 ↓
RAID 60
 ↓
RAID 0
 ↓
Selected RAID 6 component
 ↓
Available RAID 6 member
 ↓
Data
```

During healthy operation, normal reads do not require parity reconstruction.

During degraded operation, if a requested member is unavailable, the RAID 6
component can reconstruct missing information from surviving data and parity.

---

# 10. RAID 60 Write Path

The logical write path is:

```text
Host
 ↓
RAID 60
 ↓
RAID 0
 ↓
Selected RAID 6 component
 ↓
Data update
+
P maintenance
+
Q maintenance
 ↓
Physical members
```

The top-level RAID 0 layer does not calculate parity.

The target RAID 6 component performs the data and parity work.

---

# 11. Small-Write Engineering

RAID 60 inherits the parity-maintenance behavior of RAID 6.

A partial write may involve:

```text
New data
+
P parity update
+
Q parity update
```

Therefore RAID 60 carries more parity-related write work than RAID 10.

The exact implementation of partial-write handling depends on the RAID engine
and workload.

The engineering distinction is:

```text
RAID 10
→ no parity

RAID 60
→ P + Q parity
```

---

# 12. RAID 60 Chunk Size

The RAID chunk size defines the granularity at which data is distributed by the
RAID layer.

In the laboratory:

```text
md61 → 512K chunk
md62 → 512K chunk
md60 → 512K chunk
```

The actual configuration should always be verified using:

```bash
sudo mdadm --detail /dev/md60
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
```

Do not infer RAID geometry from assumptions.

---

# 13. RAID 60 Capacity Engineering

For equal-sized members, the simplified usable-capacity model is:

```text
Number of RAID 6 groups
×
(Group members - 2)
×
Member size
```

For the laboratory:

```text
2 groups
4 members per group
1 GiB per member
```

Each component provides approximately:

```text
(4 - 2) × 1 GiB
=
2 GiB
```

Therefore the two groups provide approximately:

```text
2 GiB + 2 GiB
=
4 GiB
```

The actual reported device size was approximately:

```text
3.99 GiB
```

because of RAID metadata and storage geometry.

---

# 14. Capacity Efficiency

A RAID 6 component with equal members has approximately:

```text
(N - 2) / N
```

capacity efficiency before implementation overhead.

For four members:

```text
(4 - 2) / 4
=
50%
```

Therefore the laboratory configuration had approximately:

```text
50% raw-to-usable efficiency
```

before metadata and alignment effects.

This is the capacity cost of maintaining dual parity in each component.

---

# 15. Minimum Practical RAID 60 Configuration

A practical RAID 60 requires multiple RAID 6 groups.

For the laboratory configuration:

```text
2 RAID 6 groups
×
4 members
=
8 members
```

Conceptually:

```text
Group A → D1 D2 D3 D4
Group B → D5 D6 D7 D8
```

Each group independently maintains dual parity.

---

# 16. Single-Member Failure

Suppose:

```text
md61:
D1 → FAILED
D2 → HEALTHY
D3 → HEALTHY
D4 → HEALTHY
```

Then:

```text
md61 → degraded
```

but it remains operational.

If:

```text
md62 → healthy
```

then:

```text
md60 → operational
```

and the RAID 60 filesystem can remain accessible.

---

# 17. Two-Member Failure in One Component

Suppose:

```text
md61:
D1 → FAILED
D2 → FAILED
D3 → HEALTHY
D4 → HEALTHY
```

Then:

```text
md61 → two members unavailable
```

RAID 6 still has:

```text
P + Q
```

available through the surviving members and therefore can normally continue
operating.

Conceptually:

```text
md61 → [__UU]
```

The component remains operational.

If `md62` is also operational:

```text
md60 → operational
```

This is a critical RAID 60 advantage over RAID 50.

---

# 18. One Failure in Each RAID 6 Component

Consider:

```text
md61 → [_UUU]
md62 → [_UUU]
```

Each component has:

```text
1 failed
3 working
```

Both remain operational.

Therefore:

```text
md60 → operational
```

This validates distributed protection across the component groups.

---

# 19. Two Failures in Each RAID 6 Component

Consider:

```text
md61 → [__UU]
md62 → [__UU]
```

Each component has:

```text
2 failed
2 working
```

This is the maximum two-member failure boundary of each RAID 6 group.

For the laboratory topology:

```text
md61 → survives
md62 → survives
md60 → survives
```

Therefore:

```text
2 failures in md61
+
2 failures in md62
=
4 simultaneous failed members
```

while the RAID 60 remains operational.

This must be understood as a **distribution-dependent protection model**.

---

# 20. Why Four Failures Can Be Survivable

The statement:

```text
"RAID 60 tolerates four drive failures."
```

is incomplete.

For the two-group laboratory configuration:

```text
2 + 2
```

is within the protection boundary of both components.

However:

```text
3 + 1
```

is not survivable because the component containing three failures has exceeded
its two-member RAID 6 boundary.

Similarly:

```text
4 + 0
```

is not survivable.

Therefore:

> **RAID 60 fault tolerance depends on failure distribution across RAID 6 component groups.**

---

# 21. RAID 60 Failure Matrix

For the laboratory's two RAID 6 components:

| Failure Pattern | md61     | md62     | RAID 60     |
| --------------- | -------- | -------- | ----------- |
| 0 + 0           | Healthy  | Healthy  | Operational |
| 1 + 0           | Degraded | Healthy  | Operational |
| 0 + 1           | Healthy  | Degraded | Operational |
| 1 + 1           | Degraded | Degraded | Operational |
| 2 + 0           | Degraded | Healthy  | Operational |
| 0 + 2           | Healthy  | Degraded | Operational |
| 2 + 1           | Degraded | Degraded | Operational |
| 1 + 2           | Degraded | Degraded | Operational |
| 2 + 2           | Degraded | Degraded | Operational |
| 3 + 0           | Failed   | Healthy  | Unavailable |
| 0 + 3           | Healthy  | Failed   | Unavailable |
| 3 + 1           | Failed   | Degraded | Unavailable |
| 1 + 3           | Degraded | Failed   | Unavailable |

The key rule is:

```text
Every RAID 6 component must remain operational.
```

---

# 22. Three-Member Failure in One Component

Suppose:

```text
md61 → [___U]
md62 → [UUUU]
```

Three unavailable members exceed the normal RAID 6 two-member protection
boundary.

Therefore:

```text
md61 → unavailable
```

The top-level RAID 0 cannot operate without one of its component arrays.

Therefore:

```text
md60 → unavailable
RAID 60 → unavailable
```

The healthy `md62` cannot reconstruct `md61`.

---

# 23. Failure Distribution Engineering

The correct RAID 60 failure analysis is:

```text
Total failures
+
Failure distribution
```

For example:

```text
4 total failures
```

may mean:

```text
2 in md61
+
2 in md62
```

which is survivable in the laboratory topology.

But:

```text
3 in md61
+
1 in md62
```

is not survivable.

Therefore the engineer must ask:

> **How many members failed in each RAID 6 component?**

---

# 24. Rebuild Engineering

Suppose:

```text
md61 → [UU_U]
```

and a replacement is available.

The recovery sequence is:

```text
Failed member
      ↓
Remove failed member
      ↓
Add replacement
      ↓
RAID 6 reconstruction
      ↓
P/Q calculations
      ↓
Write reconstructed data
      ↓
md61 returns healthy
```

The rebuild occurs inside:

```text
md61
```

or:

```text
md62
```

depending on the affected component.

---

# 25. RAID 60 Reconstruction

For a degraded RAID 6 component:

```text
P + Q
```

provide the dual-parity information necessary to reconstruct missing data
within the component's protection boundary.

Conceptually:

```text
Surviving data
+
P
+
Q
      ↓
Missing information
      ↓
Replacement
```

The actual reconstruction mathematics is more complex than this conceptual
model.

The critical engineering point is:

```text
1 failed member
→ recoverable

2 failed members
→ recoverable

3 failed members
→ protection boundary exceeded
```

for a given RAID 6 component.

---

# 26. Rebuild Workload

During rebuild:

```text
Normal application I/O
        +
Reads from surviving members
        +
P/Q reconstruction
        +
Writes to replacement
```

This generates additional storage I/O.

Potential effects:

```text
Higher latency
Lower throughput
Increased disk activity
Longer recovery periods
```

Actual performance depends on hardware, workload, RAID implementation, and
rebuild policy.

---

# 27. Rebuild Window

After a member failure:

```text
Healthy
   ↓
Member failure
   ↓
Degraded RAID 6
   ↓
Replacement added
   ↓
Recovery
   ↓
Healthy RAID 6
```

During recovery, redundancy is reduced relative to the fully healthy state.

If the component already has:

```text
2 failed members
```

it is at its normal protection boundary.

An additional failure in that same component before recovery completes can
therefore cause the component to become unavailable.

---

# 28. Failure During Rebuild

Suppose:

```text
md61 → [_UUU]
```

and recovery is active.

If another member fails:

```text
md61 → [__UU]
```

the component can still remain operational because it is still within
RAID 6's two-member protection boundary.

If another member fails again:

```text
md61 → [___U]
```

the component has exceeded its protection boundary.

Therefore:

```text
md61 unavailable
      ↓
md60 loses a required component
      ↓
RAID 60 unavailable
```

This is why rebuild progress and failure count must be tracked together.

---

# 29. Four-Drive Failure During the Laboratory Test

The laboratory deliberately reached:

```text
md61 → [__UU]
md62 → [__UU]
```

This represented:

```text
2 failed in md61
+
2 failed in md62
=
4 failed members
```

At this point:

```text
md60 → active
```

and the filesystem remained accessible.

The test files were readable and their SHA-256 values matched the original
healthy-state baseline.

This is an important practical demonstration of RAID 60's distributed
failure-domain behavior.

---

# 30. Recovery of Four Failed Members

The four failed members were recovered sequentially.

The recovery model was:

```text
Failed member
      ↓
Remove
      ↓
Create fresh replacement
      ↓
Verify no RAID metadata
      ↓
Add replacement
      ↓
Monitor recovery
      ↓
Verify component
      ↓
Verify SHA-256
```

This process was repeated until:

```text
md61 → [UUUU]
md62 → [UUUU]
```

The top-level array was then healthy.

---

# 31. Replacement Member Engineering

Before adding a replacement member:

```bash
sudo mdadm --examine /dev/loopX
```

Expected clean result:

```text
mdadm: No md superblock detected on /dev/loopX.
```

Then add the replacement to the correct RAID 6 component:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loopX
```

or:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loopX
```

Never add a replacement without first identifying the affected component.

---

# 32. Rebuild Monitoring

Monitor:

```bash
cat /proc/mdstat
```

or:

```bash
watch -n 2 cat /proc/mdstat
```

Typical recovery output:

```text
recovery = XX.X%
```

A component should eventually return to:

```text
[UUUU]
```

For the laboratory:

```text
md61 → [UUUU]
md62 → [UUUU]
```

indicated fully restored component membership.

---

# 33. Rebuild Completion

A rebuild is not complete merely because the replacement was accepted.

Verify:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
```

Expected:

```text
State             : clean
Active Devices    : 4
Working Devices   : 4
Failed Devices    : 0
```

Then verify:

```bash
sudo mdadm --detail /dev/md60
```

Finally:

```bash
sha256sum /mnt/raid60/testfile_01.txt \
          /mnt/raid60/testfile_02.txt \
          /mnt/raid60/testfile_03.txt
```

---

# 34. Data-Integrity Engineering

A complete storage validation must distinguish:

```text
Data accessibility
```

from:

```text
Data integrity
```

Reading a file proves that the file can be accessed.

A checksum comparison provides evidence that its contents remain unchanged.

Healthy baseline:

```bash
sha256sum /mnt/raid60/testfile_01.txt \
          /mnt/raid60/testfile_02.txt \
          /mnt/raid60/testfile_03.txt
```

Repeat after:

```text
Failure
Degraded operation
Rebuild
Final recovery
```

The laboratory baseline was:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

---

# 35. Filesystem Layer

The RAID hierarchy should be separated from the filesystem layer:

```text
Physical member
      ↓
RAID 6 component
      ↓
RAID 0
      ↓
/dev/md60
      ↓
Filesystem
      ↓
Mount
      ↓
Application
```

If RAID metadata is healthy but the filesystem is inaccessible, investigate:

```text
Filesystem corruption
Mount failure
Permission issue
Application problem
```

Useful commands:

```bash
lsblk -f
```

```bash
mount | grep /mnt/raid60
```

```bash
df -h /mnt/raid60
```

---

# 36. RAID 60 Performance Model

RAID 60 gains parallelism from:

```text
RAID 0 striping
+
Multiple RAID 6 component groups
+
Multiple physical members per component
```

However, each RAID 6 component retains dual-parity maintenance.

Therefore the performance model contains:

```text
Parallel I/O
+
P/Q parity work
```

rather than the parity-free model of RAID 10.

The actual performance must be validated through workload-specific testing.

---

# 37. RAID 60 vs RAID 50

RAID 50:

```text
RAID 5 + RAID 0
```

RAID 60:

```text
RAID 6 + RAID 0
```

Component protection:

```text
RAID 50
→ 1 failed member per component

RAID 60
→ 2 failed members per component
```

Therefore, for the two-component laboratory topology:

```text
RAID 50:
1 + 1 → operational
2 + 0 → unavailable

RAID 60:
2 + 2 → operational
3 + 1 → unavailable
```

The difference comes from the component RAID level.

---

# 38. RAID 60 vs RAID 6

A single RAID 6 array:

```text
One protection domain
```

RAID 60:

```text
Multiple RAID 6 protection domains
+
RAID 0 striping
```

Therefore RAID 60 changes the failure-domain organization.

The important question becomes:

```text
Which RAID 6 component contains the failed member?
```

rather than treating all physical disks as one parity domain.

---

# 39. RAID 60 vs RAID 10

RAID 10 uses:

```text
Mirroring
+
Striping
```

RAID 60 uses:

```text
Dual parity
+
Striping
```

Therefore:

```text
RAID 10
→ mirror-copy rebuild

RAID 60
→ P/Q reconstruction
```

The failure models are also different:

```text
RAID 10
→ mirror-set placement

RAID 60
→ RAID 6 component placement
```

---

# 40. Physical Disk Identification

Never replace a production member based only on:

```text
/dev/sdb
/dev/sdc
```

Verify:

```text
Drive serial number
Drive model
Physical slot
Operating-system path
RAID membership
RAID component
```

The engineering mapping is:

```text
Physical drive
      ↓
RAID 6 component
      ↓
RAID 0 component
      ↓
RAID 60
```

---

# 41. `mdadm` Operational Validation

### Overall state

```bash
cat /proc/mdstat
```

### Top-level RAID 60

```bash
sudo mdadm --detail /dev/md60
```

### Component RAID 6 groups

```bash
sudo mdadm --detail /dev/md61
sudo mdadm --detail /dev/md62
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
mount | grep /mnt/raid60
```

### Capacity validation

```bash
df -h /mnt/raid60
```

### Continuous RAID monitoring

```bash
watch -n 2 cat /proc/mdstat
```

### Data integrity

```bash
sha256sum /mnt/raid60/testfile_01.txt \
          /mnt/raid60/testfile_02.txt \
          /mnt/raid60/testfile_03.txt
```

---

# 42. RAID 60 Failure-Domain Model

The core engineering hierarchy is:

```text
                         RAID 60
                            |
                         RAID 0
                     /           \
                    ↓             ↓
                 RAID 6         RAID 6
                  md61           md62
                 / | | \         / | | \
                D D D D         D D D D
```

When a member fails:

```text
Failed member
      ↓
Identify RAID 6 component
      ↓
Count failures in that component
      ↓
Is it within the two-member boundary?
```

If yes:

```text
Component remains operational
      ↓
md60 can remain operational
      ↓
RAID 60 can remain accessible
```

If no:

```text
RAID 6 component unavailable
      ↓
md60 loses required component
      ↓
RAID 60 unavailable
```

---

# 43. Engineering Responsibilities

A Storage Validation Engineer should validate:

```text
RAID 6 component creation
+
RAID 0 aggregation
+
Capacity
+
P/Q protection
+
Single-member failure
+
Two-member failure
+
Distributed failures
+
Maximum failure scenario
+
Member replacement
+
Rebuild behavior
+
Data accessibility
+
Data integrity
+
Final redundancy
```

Negative scenarios should include:

```text
Second failure in degraded component
Failure during rebuild
Third failure in one component
Replacement with stale RAID metadata
Filesystem issue after RAID degradation
Data-integrity verification after rebuild
```

---

# 44. RAID 60 Validation Model

A complete validation sequence is:

```text
Create RAID 6 groups
        ↓
Create RAID 0
        ↓
Create filesystem
        ↓
Mount filesystem
        ↓
Write known test data
        ↓
Record SHA-256 baseline
        ↓
Inject failures
        ↓
Validate component states
        ↓
Validate top-level RAID
        ↓
Validate filesystem
        ↓
Validate data integrity
        ↓
Replace failed members
        ↓
Monitor rebuild
        ↓
Verify component health
        ↓
Verify top-level health
        ↓
Verify SHA-256
```

---

# 45. Laboratory Evidence Model

The fresh RAID 60 laboratory demonstrated:

```text
8 × 1 GiB virtual members

md61 → RAID 6
md62 → RAID 6
md60 → RAID 0
```

The test covered:

```text
Single-member failure in md61
Two-member failure in md61
Single-member failure in md62
One failure in each component
Two failures in each component
Four simultaneous failed members
Sequential recovery
Data-access validation
SHA-256 integrity validation
Final RAID validation
Controlled cleanup
```

The maximum failure test reached:

```text
md61 → 2 failed
md62 → 2 failed
```

At that state:

```text
md60 → active
Filesystem → accessible
Files → readable
SHA-256 → unchanged
```

All four failed members were subsequently recovered sequentially.

---

# 46. Final Engineering Mental Model

Think about RAID 60 as:

```text
                       RAID 60
                          |
                       RAID 0
                     /       \
                    ↓         ↓
                 RAID 6     RAID 6
                  md61       md62
                 [UUUU]     [UUUU]
```

Normal operation:

```text
md61 → [UUUU]
md62 → [UUUU]
```

One distributed failure:

```text
md61 → [_UUU]
md62 → [UUUU]
```

Two failures in one component:

```text
md61 → [__UU]
md62 → [UUUU]
```

Maximum two-group failure:

```text
md61 → [__UU]
md62 → [__UU]
```

Critical failure boundary:

```text
md61 → [___U]
```

or:

```text
md62 → [___U]
```

Once a RAID 6 component exceeds two failed members:

```text
Component unavailable
      ↓
RAID 0 loses component
      ↓
RAID 60 unavailable
```

---

# 47. Key Engineering Takeaways

1. RAID 60 combines multiple RAID 6 groups with RAID 0 striping.
2. Each RAID 6 component provides P + Q dual-parity protection.
3. The top-level RAID 0 provides striping but no redundancy.
4. There is no global parity domain across all RAID 60 members.
5. Each RAID 6 component is an independent protection domain.
6. One failed member in a component is normally survivable.
7. Two failed members in a component are normally survivable.
8. Two failures in each of two components can therefore be survivable.
9. The laboratory demonstrated four simultaneous failed members using the 2+2 pattern.
10. Three failures in one RAID 6 component exceed its normal protection boundary.
11. A failed component causes the top-level RAID 0 to lose a required member.
12. Rebuilds occur inside the affected RAID 6 component.
13. RAID 60 recovery uses dual-parity reconstruction rather than mirror copying.
14. Rebuilds generate additional storage I/O.
15. Failure distribution is more important than total failure count alone.
16. Filesystem accessibility must be validated separately from RAID metadata.
17. SHA-256 checksums provide stronger data-integrity evidence than simple file reads.
18. Replacement members should be checked for stale RAID metadata before being added.
19. Physical disk identity should be confirmed before replacing real hardware.
20. RAID 60 offers stronger component-level failure protection than RAID 50 at the cost of additional parity work and capacity overhead.

---

# 48. Final Engineering Principle

> **Always troubleshoot RAID 60 from the RAID 6 component level upward.**

Use this reasoning:

```text
Failed member
      ↓
Identify RAID 6 component
      ↓
Count failures in that component
      ↓
Within two-member boundary?
      ↓
YES
 ↓
Component remains operational
 ↓
Check md60
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
RAID 6 component unavailable
 ↓
RAID 0 loses required component
 ↓
RAID 60 unavailable
```

For the two-component laboratory configuration:

```text
2 failures in md61
+
2 failures in md62
→ 4 simultaneous failed members
→ both RAID 6 components remain operational
→ RAID 60 can remain operational
```

while:

```text
3 failures in md61
+
any state in md62
→ md61 protection boundary exceeded
→ md60 loses md61
→ RAID 60 unavailable
```

The central engineering model is:

> **Physical member → RAID 6 protection domain → RAID 0 aggregation → filesystem → application data.**

