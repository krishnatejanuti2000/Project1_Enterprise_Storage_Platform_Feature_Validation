# RAID 50 — Interview Guide

## 1. What is RAID 50?

RAID 50 combines:

```text
RAID 5
+
RAID 0
```

Multiple RAID 5 groups are created first, and those protected groups are
then striped using RAID 0.

Conceptually:

```text
                 RAID 50
                    |
                 RAID 0
                /      \
             RAID 5    RAID 5
              md51      md52
```

RAID 5 provides the redundancy.

RAID 0 provides the top-level striping.

---

# 2. Why is RAID 50 called RAID 5+0?

Because the architecture combines:

```text
RAID 5 → Distributed parity
RAID 0 → Striping
```

The RAID 5 groups provide protection independently, while RAID 0 stripes
logical data across the component groups.

---

# 3. What is the main advantage of RAID 50?

The main advantages are:

```text
Parity-based redundancy
+
Multiple RAID 5 groups
+
Top-level striping
+
Higher aggregate parallelism
```

RAID 50 can provide more parallelism than a single RAID 5 configuration
while retaining parity-based protection inside each component.

---

# 4. Does RAID 50 use parity?

Yes.

Parity is provided by every underlying RAID 5 group.

Conceptually:

```text
RAID 5 Group 1 → parity
RAID 5 Group 2 → parity
RAID 5 Group 3 → parity
```

There is no additional parity calculation at the top-level RAID 0 layer.

---

# 5. What is the redundancy mechanism in RAID 50?

RAID 50 uses:

```text
Distributed parity
```

inside each RAID 5 component.

Conceptually:

```text
Data
+
Parity
```

protects the component RAID 5 group.

The RAID 0 layer simply stripes across those protected groups.

---

# 6. What is the minimum number of RAID 50 members?

A practical two-component RAID 50 requires:

```text
2 RAID 5 groups
```

and each RAID 5 group requires at least:

```text
3 members
```

Therefore a conventional two-group RAID 50 requires:

```text
6 members
```

Conceptually:

```text
Group 1 → D1 D2 D3
Group 2 → D4 D5 D6
```

---

# 7. What is the usable capacity of RAID 50?

For equal-sized members:

```text
Usable Capacity
≈
Number of RAID 5 groups
×
(Group members - 1)
×
Member size
```

Example:

```text
2 groups
3 members per group
1 TB members
```

Each group provides approximately:

```text
(3 - 1) × 1 TB
=
2 TB
```

Total:

```text
2 × 2 TB
=
4 TB
```

Actual usable capacity is slightly affected by RAID metadata, alignment, and
implementation details.

---

# 8. How does RAID 50 store data?

Conceptually:

```text
                 RAID 50
                    |
                 RAID 0
                /      \
               ↓        ↓
             md51      md52
             RAID 5    RAID 5
```

Logical data is striped across the RAID 5 component arrays.

Each component independently maintains its parity.

Therefore the architecture is:

```text
Physical members
      ↓
RAID 5 protection
      ↓
RAID 0 striping
      ↓
Logical RAID 50
```

---

# 9. Does RAID 50 have one global parity set?

No.

This is one of the most important RAID 50 concepts.

Each RAID 5 group has its own parity:

```text
md51 → own parity
md52 → own parity
```

There is no cross-group parity.

Therefore:

```text
md51 parity
```

cannot reconstruct:

```text
md52 data
```

and vice versa.

---

# 10. How does RAID 50 handle a single member failure?

Suppose:

```text
md51:
D1 → FAILED
D2 → HEALTHY
D3 → HEALTHY
```

Then:

```text
md51 → [_UU]
```

The RAID 5 group becomes degraded but remains operational.

If:

```text
md52 → [UUU]
```

then:

```text
md50 → operational
```

Therefore:

```text
Failed member
      ↓
RAID 5 becomes degraded
      ↓
RAID 5 remains operational
      ↓
RAID 0 remains active
      ↓
RAID 50 remains accessible
```

---

# 11. Can RAID 50 tolerate two drive failures?

The answer is:

> It depends on where the failures occur.

For example:

```text
md51 → 1 failed
md52 → 1 failed
```

Both RAID 5 groups can remain operational.

Therefore:

```text
RAID 50 → operational
```

But:

```text
md51 → 2 failed
md52 → 0 failed
```

causes `md51` to exceed normal RAID 5 protection.

Therefore:

```text
md51 → unavailable
      ↓
md50 loses one required component
      ↓
RAID 50 unavailable
```

---

# 12. What is the most important RAID 50 failure rule?

> Every RAID 5 component must remain operational.

A single failed member in a component is normally survivable.

Two failures in the same three-member RAID 5 component exceed its normal
single-member fault tolerance.

Therefore the failure distribution matters.

---

# 13. Why is failure count alone not enough?

Suppose an engineer says:

```text
"Two disks failed."
```

That statement is incomplete.

You must ask:

```text
Which RAID 5 components contain those failed members?
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

The first pattern can remain operational.

The second can cause the affected RAID 5 component to fail.

Therefore:

> **Always map failed members to their component RAID 5 group.**

---

# 14. What happens if one drive fails in each RAID 5 group?

Example:

```text
md51 → [_UU]
md52 → [_UU]
```

Each group has:

```text
1 failed
2 healthy
```

Both RAID 5 groups remain operational.

Therefore:

```text
md50 → operational
RAID 50 → accessible
```

This is a distributed failure pattern.

---

# 15. What happens if two drives fail in the same RAID 5 group?

Example:

```text
md51 → [__U]
md52 → [UUU]
```

The RAID 5 protection boundary of `md51` is exceeded.

Therefore:

```text
md51 → unavailable
```

Since `md50` is RAID 0:

```text
md50 requires md51
```

The loss of one RAID 0 component means:

```text
RAID 50 → unavailable
```

The healthy `md52` cannot reconstruct `md51`.

---

# 16. What happens if three drives fail across three different RAID 5 groups?

For a larger RAID 50:

```text
md51 → [_UU]
md52 → [U_U]
md53 → [UU_]
```

Each component has one failed member.

Therefore all component groups may remain operational.

So:

```text
RAID 50 → can remain operational
```

This example demonstrates why total failure count is not the complete
decision criterion.

---

# 17. How is RAID 50 different from RAID 6?

RAID 6 provides:

```text
P + Q
```

and normally tolerates:

```text
2 failed members
```

within the same RAID 6 group.

RAID 50 provides:

```text
RAID 5 protection
```

within each component.

Therefore:

```text
RAID 6:
2 failures in one group
→ normally tolerated

RAID 50:
2 failures in one RAID 5 component
→ protection exceeded
```

---

# 18. How is RAID 50 different from RAID 10?

RAID 10 uses:

```text
Mirroring + Striping
```

RAID 50 uses:

```text
RAID 5 parity + Striping
```

Therefore:

| Feature                 | RAID 10          | RAID 50                    |
| ----------------------- | ---------------- | -------------------------- |
| Redundancy              | Mirroring        | Parity                     |
| Parity                  | No               | Yes                        |
| Striping                | Yes              | Yes                        |
| Rebuild                 | Mirror copy      | Parity reconstruction      |
| Multiple-failure rule   | Mirror placement | RAID 5 component placement |
| Small-write parity work | No               | Yes                        |

---

# 19. How does RAID 50 perform a read?

Conceptually:

```text
Host
 ↓
RAID 50
 ↓
RAID 0
 ↓
Target RAID 5 component
 ↓
RAID 5 members
 ↓
Data
```

For normal healthy reads, parity calculation is not required simply to read
existing data.

If a member is degraded, the RAID 5 layer may reconstruct missing data as
needed.

---

# 20. How does RAID 50 perform a write?

Conceptually:

```text
Host
 ↓
RAID 50
 ↓
RAID 0
 ↓
Selected RAID 5 group
 ↓
Data update
+
Parity maintenance
 ↓
Members
```

The RAID 0 layer determines the component destination.

The RAID 5 component handles the parity-related work.

---

# 21. Does RAID 50 have a parity write penalty?

RAID 50 inherits parity-related write work from RAID 5.

Therefore a partial write may involve:

```text
New data
+
Parity maintenance
```

This differs from RAID 10, which does not use parity.

---

# 22. How does RAID 50 rebuild a failed member?

Suppose:

```text
md51 → [_UU]
```

and a replacement member is available.

The recovery sequence is:

```text
Failed member
      ↓
Remove failed member
      ↓
Add replacement
      ↓
Read surviving members
      ↓
Reconstruct missing information
      ↓
Write replacement
      ↓
md51 → [UUU]
```

The rebuild occurs inside the affected RAID 5 component.

---

# 23. Where does the rebuild happen?

The physical replacement member is rebuilt into:

```text
md51
```

or:

```text
md52
```

depending on which component contained the failure.

The top-level:

```text
md50
```

does not independently reconstruct the physical disk.

---

# 24. What happens during RAID 50 rebuild?

During rebuild there can be:

```text
Normal application I/O
+
Reads from surviving members
+
Reconstruction work
+
Writes to replacement
```

Therefore rebuild can produce:

```text
Higher latency
Lower throughput
Additional disk activity
```

The impact depends on workload, storage hardware, and RAID implementation.

---

# 25. What command should be used to monitor rebuild?

Use:

```bash
cat /proc/mdstat
```

or:

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

---

# 26. How do you confirm rebuild completion?

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

Then verify:

```bash
sudo mdadm --detail /dev/md50
```

Finally verify data integrity.

---

# 27. How do you verify data integrity after rebuild?

Use SHA-256:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

Compare the values against the healthy-state baseline.

A successful rebuild should preserve the expected test-data hashes.

---

# 28. Why is a checksum better than simply reading the file?

This:

```bash
cat testfile.txt
```

proves that the file is readable.

It does not provide a compact comparison against the original contents.

A checksum allows:

```text
Healthy baseline
      ↓
Failure state
      ↓
Rebuild state
      ↓
Compare hash
```

Matching hashes provide evidence that the content remained unchanged.

---

# 29. What should you check when a RAID member fails?

Use:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md50
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Then determine:

```text
Which member failed?
Which RAID 5 component contains it?
How many failures exist in that component?
Is that component still operational?
Is md50 still operational?
Is the filesystem accessible?
```

---

# 30. What is the RAID 50 troubleshooting flow?

```text
RAID problem
     ↓
cat /proc/mdstat
     ↓
Check md50
     ↓
Check md51 / md52
     ↓
Identify failed member
     ↓
Map member to component
     ↓
Count failures in component
     ↓
Validate component health
     ↓
Validate md50
     ↓
Validate filesystem
     ↓
Validate data
     ↓
Replace member
     ↓
Monitor rebuild
     ↓
Verify clean state
```

---

# 31. What happens if another member fails during rebuild?

Suppose:

```text
md51 → [_UU]
```

and the replacement is rebuilding.

If another member in `md51` fails:

```text
md51 → [__U]
```

The RAID 5 component has exceeded its normal protection boundary.

Therefore:

```text
md51 → unavailable
      ↓
md50 loses required RAID 0 component
      ↓
RAID 50 unavailable
```

This is an important operational risk.

---

# 32. What happens if a second failure occurs in another component while one component is rebuilding?

Suppose:

```text
md51 → [_UU]   rebuilding
md52 → [_UU]
```

Both components are degraded.

The array may remain operational because both RAID 5 groups remain available.

However, another failure in either component could exceed that component's RAID 5
protection.

Therefore engineers must track:

```text
Failure count
+
Failure location
+
Rebuild state
```

---

# 33. What is a hot spare in RAID 50?

A hot spare is a standby device that can become a rebuild target.

Conceptually:

```text
Failed member
      ↓
Hot spare
      ↓
Component RAID 5 rebuild
      ↓
Redundancy restored
```

A hot spare is a recovery mechanism.

It does not create an additional RAID protection layer.

---

# 34. How do you validate a replacement disk?

Before adding it:

```bash
sudo mdadm --examine /dev/loopX
```

A clean replacement should report:

```text
mdadm: No md superblock detected on /dev/loopX.
```

Then add it to the correct component:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loopX
```

or:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loopX
```

The target component must be verified before adding the replacement.

---

# 35. Why is physical disk identification important?

Device names such as:

```text
/dev/sdb
/dev/sdc
```

can change.

In a real storage environment, verify:

```text
Drive serial number
Drive model
Physical slot
Operating-system device
RAID membership
RAID component
```

This prevents replacement of the wrong disk.

---

# 36. What is the failure-domain model of RAID 50?

The important hierarchy is:

```text
Physical disk
      ↓
RAID 5 component
      ↓
RAID 0 component
      ↓
RAID 50
```

When a disk fails, first determine:

```text
Which RAID 5 component?
```

Then determine:

```text
How many members have failed in that component?
```

---

# 37. What is the RAID 50 failure matrix?

For a two-component configuration:

| Failure Pattern | md51                | md52                | RAID 50     |
| --------------- | ------------------- | ------------------- | ----------- |
| 0 + 0           | Healthy             | Healthy             | Operational |
| 1 + 0           | Degraded            | Healthy             | Operational |
| 0 + 1           | Healthy             | Degraded            | Operational |
| 1 + 1           | Degraded            | Degraded            | Operational |
| 2 + 0           | Protection exceeded | Healthy             | Unavailable |
| 0 + 2           | Healthy             | Protection exceeded | Unavailable |

The key is:

```text
Every component RAID 5 must remain operational.
```

---

# 38. Interview Scenario — One Drive Fails

### Question

One member of a RAID 50 component fails. What happens?

### Answer

The affected RAID 5 group enters degraded mode.

Because RAID 5 normally tolerates one failed member, that component can continue
operating.

If every other component remains healthy:

```text
RAID 50 → remains operational
```

---

# 39. Interview Scenario — Two Drives Fail in Different Components

### Question

Two members fail, one from each RAID 5 group. Is RAID 50 necessarily lost?

### Answer

No.

For example:

```text
md51 → [_UU]
md52 → [_UU]
```

Each component has one failed member and can still operate.

Therefore the top-level RAID 50 can remain accessible.

---

# 40. Interview Scenario — Two Drives Fail in the Same Component

### Question

What happens if two members of the same RAID 5 component fail?

### Answer

That RAID 5 component exceeds its normal single-member fault tolerance.

For example:

```text
md51 → [__U]
```

The component becomes unavailable.

Because RAID 0 cannot operate without one of its required components:

```text
md50 → unavailable
RAID 50 → unavailable
```

---

# 41. Interview Scenario — Why Can Another RAID 5 Group Not Help?

### Question

If `md51` fails, why can't healthy `md52` reconstruct it?

### Answer

Because the RAID 5 groups are independent protection domains.

`md51` has its own:

```text
Data
+
Parity
```

and `md52` has a separate:

```text
Data
+
Parity
```

There is no cross-group parity relationship.

---

# 42. Interview Scenario — RAID 50 vs RAID 6

### Question

Why can RAID 6 survive two failures in one group while RAID 50 may not?

### Answer

RAID 6 uses dual parity:

```text
P + Q
```

so two failed members can normally be reconstructed.

RAID 50 uses RAID 5 components, and each RAID 5 component normally tolerates
only one failed member.

Therefore:

```text
RAID 6:
2 failures in one group → normally tolerated

RAID 50:
2 failures in same RAID 5 group → component protection exceeded
```

---

# 43. Interview Scenario — Rebuild

### Question

How does RAID 50 rebuild a failed member?

### Answer

The RAID 5 component containing the failed member uses surviving members and
parity to reconstruct the missing information.

Conceptually:

```text
Surviving data
+
Parity
      ↓
Missing information
      ↓
Replacement member
```

The recovery happens inside the affected RAID 5 component.

---

# 44. Interview Scenario — Rebuild Risk

### Question

Why is another failure during RAID 50 rebuild dangerous?

### Answer

Because the affected RAID 5 component is already degraded.

If another member of the same component fails before the rebuild completes:

```text
[_UU]
  ↓
[__U]
```

The RAID 5 component exceeds its protection boundary.

Therefore:

```text
Component failure
      ↓
RAID 0 loses component
      ↓
RAID 50 becomes unavailable
```

---

# 45. Interview Scenario — Data Integrity

### Question

How would you prove that data survived a RAID 50 failure and rebuild?

### Answer

Create a checksum baseline before failure:

```bash
sha256sum /mnt/raid50/testfile_01.txt \
          /mnt/raid50/testfile_02.txt \
          /mnt/raid50/testfile_03.txt
```

Then repeat the same command after degradation and after rebuild.

Matching SHA-256 values provide evidence that the test-file contents remained
unchanged.

---

# 46. Interview Scenario — Troubleshooting

### Question

What is the first thing you do when a RAID 50 member fails?

### Answer

First inspect:

```bash
cat /proc/mdstat
```

Then inspect:

```bash
sudo mdadm --detail /dev/md50
sudo mdadm --detail /dev/md51
sudo mdadm --detail /dev/md52
```

Then determine:

```text
Which physical member failed?
Which RAID 5 component contains it?
How many failures exist in that component?
Is the component still operational?
Is md50 still operational?
```

---

# 47. Interview Scenario — Filesystem

### Question

The RAID appears healthy but the filesystem is inaccessible. Is RAID necessarily
the problem?

### Answer

No.

The storage stack must be investigated layer by layer:

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

Useful commands include:

```bash
lsblk -f
mount | grep /mnt/raid50
df -h /mnt/raid50
```

The filesystem layer should be investigated separately from RAID metadata.

---

# 48. Interview Scenario — Practical Lab

### Question

How would you validate RAID 50 in a Linux lab?

### Answer

A practical sequence is:

```text
Create RAID 5 component groups
        ↓
Create top-level RAID 0
        ↓
Create filesystem
        ↓
Mount filesystem
        ↓
Write known test data
        ↓
Record SHA-256 baseline
        ↓
Inject member failure
        ↓
Verify degraded state
        ↓
Verify data accessibility
        ↓
Replace member
        ↓
Monitor rebuild
        ↓
Verify clean state
        ↓
Verify SHA-256 again
```

This validates both:

```text
RAID behavior
+
Data integrity
```

---

# 49. Strong Interview Answer — Explain RAID 50

> RAID 50 combines multiple RAID 5 groups with RAID 0 striping. Each RAID 5
> group provides distributed parity and acts as an independent protection
> domain, while the top-level RAID 0 stripes logical data across the protected
> groups. A single failed member in a RAID 5 component is normally survivable,
> but two failures in the same RAID 5 component exceed that component's
> protection boundary and cause the RAID 0 layer to lose a required component.
> Two failures distributed across different RAID 5 groups can therefore be
> survivable, depending on the configuration. Rebuild is performed inside
> the affected RAID 5 component using surviving data and parity, and the
> final recovery should be verified using RAID metadata, filesystem access,
> and data-integrity checks.

---

# 50. Quick Interview Revision

```text
RAID 50
→ RAID 5 + RAID 0

Redundancy
→ RAID 5 distributed parity

Top-level layer
→ RAID 0 striping

Global parity
→ None

Minimum practical two-group design
→ 6 members

Capacity
→ Sum of component RAID 5 usable capacities

1 failure in one component
→ Normally survives

1 failure in each of two components
→ Can survive

2 failures in same RAID 5 component
→ Protection exceeded

Rebuild
→ RAID 5 reconstruction inside affected component

Important command
→ cat /proc/mdstat

Detailed validation
→ mdadm --detail

Data integrity
→ SHA-256

Most important rule
→ Evaluate failures per RAID 5 component
```

# 51. Interview Checklist

Before answering a RAID 50 failure question, think through:

```text
□ What is the RAID hierarchy?
□ How many RAID 5 components exist?
□ Which physical member failed?
□ Which component contains it?
□ How many failures exist in that component?
□ Is that RAID 5 component still operational?
□ Is the top-level RAID 0 still active?
□ Is the filesystem accessible?
□ Is the data intact?
□ Is a replacement available?
□ Has rebuild completed?
□ Are all expected members healthy?
```

---

# 52. Final Interview Principle

The most important RAID 50 interview rule is:

> **Never judge RAID 50 survivability from the total number of failed drives alone. Map every failed member to its RAID 5 component and evaluate each protection domain independently.**

Use this reasoning:

```text
Failed member
      ↓
Identify RAID 5 component
      ↓
Count failures inside component
      ↓
Component operational?
      ↓
YES
 ↓
Check md50
 ↓
Check filesystem
 ↓
Check data integrity

NO
 ↓
Component RAID 5 unavailable
 ↓
RAID 0 loses required component
 ↓
RAID 50 unavailable
```

This component-level failure-domain model is the core RAID 50 concept that a
Storage QA, Storage Validation, or Storage Automation Engineer should be able
to explain and troubleshoot.

