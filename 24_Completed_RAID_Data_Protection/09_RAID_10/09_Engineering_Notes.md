# RAID 10 — Engineering Notes

## 1. Engineering Objective

RAID 10 is designed to combine:

```text
Mirroring
    +
Striping
```

The engineering goal is to provide:

```text
Data redundancy
+
High I/O parallelism
+
No parity-calculation overhead
```

Unlike RAID 5 and RAID 6, RAID 10 does not derive redundancy from
parity.

Instead, redundancy comes from maintaining multiple copies of the
data.

---

# 2. RAID 10 Architecture

The classic RAID 10 mental model is:

```text
Mirror Pair 1
    D1 ↔ D2

Mirror Pair 2
    D3 ↔ D4
```

Data is then striped across the mirror sets.

Conceptually:

```text
             RAID 10
                |
       ┌────────┴────────┐
       ↓                 ↓
   Mirror Set 1      Mirror Set 2
    D1 ↔ D2           D3 ↔ D4
```

This can be thought of as:

```text
RAID 1
   ↓
Create redundant copies

RAID 0
   ↓
Stripe across the redundant groups
```

The simple mirror-pair diagram is a teaching model. Linux software
RAID 10 supports more flexible copy-placement layouts.

---

# 3. Redundancy Mechanism

RAID 10 does not use:

```text
P parity
Q parity
XOR parity
Reed–Solomon
Galois Field arithmetic
```

Instead:

```text
Data
 ↓
Multiple copies
```

For example:

```text
A → Copy 1
A → Copy 2
```

Therefore redundancy is achieved through duplication.

---

# 4. Striping Mechanism

Striping distributes logical data across the available RAID 10
layout.

Conceptually:

```text
Logical data:

A B C D E F G H
```

may be distributed across mirror sets as:

```text
Mirror Set 1 → A C E G
Mirror Set 2 → B D F H
```

while each value has its redundant copy within the mirror layout.

The result is:

```text
Mirroring
    ↓
Redundancy

Striping
    ↓
Parallelism
```

---

# 5. Read Path

For a healthy mirror group:

```text
D1 ↔ D2
```

both members contain the same logical data.

A read can therefore be serviced from an available mirror member.

Conceptually:

```text
Host
 ↓
RAID 10 layer
 ↓
Available mirror copy
 ↓
Host
```

Multiple reads may be distributed across available members,
depending on workload and implementation.

No parity calculation is necessary.

---

# 6. Write Path

Suppose new data is:

```text
X
```

and its mirror group is:

```text
D1 ↔ D2
```

The RAID layer must maintain both copies:

```text
D1 → X
D2 → X
```

The logical write requirement is therefore:

```text
Write data
   ↓
Mirror copy 1
+
Mirror copy 2
```

There is no P or Q update.

---

# 7. Small Random Writes

RAID 5 and RAID 6 may require parity-maintenance operations for
partial updates.

RAID 10 does not have parity.

Conceptually:

```text
RAID 10:

New data
   ↓
Write to mirror copy 1
+
Write to mirror copy 2
```

Therefore RAID 10 avoids:

```text
Parity calculation
Parity update
P/Q maintenance
```

This is one reason RAID 10 is frequently considered for
performance-sensitive workloads.

---

# 8. Single-Member Failure

Suppose:

```text
D1 ↔ D2
```

and:

```text
D1 → FAILED
D2 → HEALTHY
```

The data that existed on D1 remains available through D2.

Therefore:

```text
D1 failure
   ↓
Surviving mirror
   ↓
Data remains accessible
```

The array enters a degraded state, but the mirror still contains
the required information.

---

# 9. Two-Member Failure

Two-member failure behavior depends on which members fail.

## Different mirror sets

```text
D1 → FAILED
D3 → FAILED
```

with:

```text
D2 → HEALTHY
D4 → HEALTHY
```

Each mirror set retains one surviving member:

```text
D1 ❌ ↔ D2 ✅
D3 ❌ ↔ D4 ✅
```

The array can therefore remain operational.

## Same mirror set

```text
D1 → FAILED
D2 → FAILED
```

Then:

```text
D1 ❌ ↔ D2 ❌
```

The entire mirror set is lost.

Because RAID 10 has no parity reconstruction mechanism, the missing
data cannot be mathematically reconstructed from the remaining
mirror set.

---

# 10. Fault-Tolerance Engineering Rule

The correct engineering rule is:

> Every mirror set must retain at least one surviving member.

Therefore:

```text
One member failure
    ↓
Normally survivable

Two failures in different mirror sets
    ↓
Can be survivable

Two failures in the same mirror set
    ↓
Mirror set lost
```

This is why RAID 10 does not have a simple fixed statement such as:

```text
"RAID 10 always tolerates two disk failures."
```

The failure pattern matters.

---

# 11. Rebuild Engineering

Suppose:

```text
D1 → FAILED
D2 → HEALTHY
```

and replacement:

```text
D1'
```

is available.

The surviving mirror already contains the failed member's data.

Therefore:

```text
D2
 ↓
Read surviving copy
 ↓
Write replacement
 ↓
D1'
```

This is fundamentally different from parity RAID.

There is no need to calculate:

```text
P
Q
XOR
Reed–Solomon
```

for the normal single-member mirror rebuild.

---

# 12. Rebuild Workload

Although the rebuild algorithm is conceptually simple, it still
generates significant storage I/O.

During rebuild:

```text
Normal application I/O
        +
Read surviving mirror
        +
Write replacement
```

Therefore rebuild can affect:

```text
Latency
Throughput
Application response time
```

The surviving mirror member may receive additional read workload,
while the replacement receives rebuild writes.

---

# 13. Rebuild Window

After a member failure:

```text
Healthy
   ↓
Member failure
   ↓
Degraded
   ↓
Replacement
   ↓
Rebuild
   ↓
Healthy
```

The rebuild window is the period during which the failed mirror copy
has not yet been restored.

A second failure during this period must be evaluated according to
mirror-pair placement.

---

# 14. Example of Rebuild Risk

Suppose:

```text
D1 ❌
D3 ❌
```

Then:

```text
D1 ❌ ↔ D2 ✅
D3 ❌ ↔ D4 ✅
```

Both mirror sets are degraded.

If D2 fails before the D1 mirror is restored:

```text
D1 ❌
D2 ❌
```

then:

```text
D1 ❌ ↔ D2 ❌
```

and that complete mirror set is lost.

Therefore:

```text
Failure count
+
Failure location
+
Rebuild state
```

must all be considered during RAID 10 troubleshooting.

---

# 15. Capacity Engineering

With equal-sized members:

```text
Usable Capacity ≈ (N / 2) × Member Size
```

The approximate usable capacity is therefore 50% of raw capacity.

Example:

```text
4 × 1 TB
```

Raw:

```text
4 TB
```

Usable:

```text
2 TB
```

The remaining capacity is consumed by the duplicate mirror copies.

---

# 16. Minimum Member Count

A conventional two-copy RAID 10 arrangement requires at least:

```text
4 members
```

because at least two redundant groups are required for the normal
striped-mirror architecture.

Conceptually:

```text
D1 ↔ D2
D3 ↔ D4
```

---

# 17. Performance Trade-off

RAID 10 exchanges capacity efficiency for performance and simple
redundancy.

Advantages:

```text
High I/O parallelism
No parity calculation
No Reed–Solomon calculation
Simple mirror-based rebuild
Good random I/O characteristics
```

Costs:

```text
Approximately 50% usable capacity
Two physical copies must be maintained
Rebuild still consumes I/O resources
```

---

# 18. RAID 10 vs RAID 6 — Engineering Trade-off

### RAID 10

```text
Redundancy:
    Mirroring

Parity:
    None

Capacity:
    ~50%

Write path:
    Mirror updates

Rebuild:
    Copy surviving mirror

Failure model:
    Depends on mirror placement
```

### RAID 6

```text
Redundancy:
    P + Q

Parity:
    Yes

Capacity:
    (N - 2) × member size

Write path:
    Data + parity maintenance

Rebuild:
    P/Q-based reconstruction

Failure model:
    Two-member fault tolerance
```

The choice depends on workload, capacity requirements, and desired
failure protection.

---

# 19. RAID 10 Layouts in Linux `mdadm`

Linux software RAID 10 supports multiple data-copy layouts.

Important layouts include:

```text
near
far
offset
```

The layout controls where the redundant copies are positioned.

For example:

```text
near=2
```

means:

```text
2 copies
+
near placement
```

The number represents the number of copies, while the layout name
describes their placement.

It does not mean:

```text
2 mirror pairs
```

---

# 20. `near` Layout

In the near layout, copies are placed relatively close together in
the logical layout.

Conceptually:

```text
A A
B B
C C
D D
```

The diagram is conceptual rather than a literal statement about
individual physical sectors.

---

# 21. `far` Layout

The far layout places the redundant copy farther away in the
logical/device address space.

The intention is to provide different physical placement behavior
that can benefit certain access patterns.

The redundancy model remains:

```text
Multiple copies
```

---

# 22. `offset` Layout

The offset layout positions the second copy using an offset within
the RAID10 layout.

Again:

```text
Multiple copies
+
different placement strategy
```

The RAID level remains RAID 10.

---

# 23. `mdadm` Operational Validation

Important commands include:

```bash
cat /proc/mdstat
```

and:

```bash
sudo mdadm --detail /dev/md10
```

Important fields to inspect include:

```text
Raid Level
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

The actual layout should always be verified rather than assumed.

---

# 24. Hot Spare Engineering

A hot spare is an available standby disk that can become a rebuild
target.

Conceptually:

```text
Active RAID members
        +
Hot Spare
```

After a failure:

```text
Failed member
     ↓
Hot spare
     ↓
Rebuild
```

The hot spare is a recovery mechanism.

It does not fundamentally change RAID 10's mirror-based redundancy
model.

---

# 25. Unequal Drive Sizes

For simple RAID 10 configurations, unequal member sizes can affect
usable capacity because the RAID implementation must establish a
consistent member geometry.

The exact usable capacity should therefore be verified from:

```bash
sudo mdadm --detail /dev/md10
```

rather than inferred only from raw disk sizes.

---

# 26. Troubleshooting Mental Model

When a RAID 10 member fails:

```text
Member failure
     ↓
Identify mirror relationship
     ↓
Check surviving copy
     ↓
Verify degraded state
     ↓
Check replacement / hot spare
     ↓
Rebuild mirror
     ↓
Verify healthy state
```

For multiple failures:

```text
Failure count
     +
Which mirror sets are affected?
     +
Is rebuild already in progress?
```

must all be evaluated.

---

# 27. Engineering Mental Model

```text
                       RAID 10
                          |
             ┌────────────┴────────────┐
             ↓                         ↓
         Mirroring                  Striping
             ↓                         ↓
         Redundancy                Parallel I/O
             \                         /
              \                       /
                    RAID 10
```

Failure model:

```text
Every mirror set
must have ≥ 1 surviving member.
```

Rebuild:

```text
Surviving mirror
      ↓
Read
      ↓
Replacement
      ↓
Write identical data
      ↓
Mirror restored
```

---

# 28. Key Engineering Takeaways

1. RAID 10 combines mirroring and striping.
2. Redundancy comes from duplicate copies rather than parity.
3. RAID 10 does not require XOR, Reed–Solomon, or Galois Field
   calculations for normal redundancy.
4. RAID 10 has approximately 50% usable capacity for equal-sized
   members.
5. A conventional two-copy RAID 10 arrangement requires at least
   four members.
6. One member failure is normally survivable.
7. Two failures in different mirror sets can be survivable.
8. Two failures in the same mirror set can destroy that mirror set.
9. RAID 10 rebuilds by copying an available surviving mirror.
10. Rebuild still creates additional storage I/O.
11. `mdadm` supports near, far, and offset RAID10 layouts.
12. `near=2` represents two data copies using the near layout.
13. The actual RAID10 layout should be verified using `mdadm`.
14. Hot spares can provide an automatic rebuild target.
15. Failure location is as important as failure count.
16. RAID 10 is especially attractive when performance is more
    important than capacity efficiency.
17. RAID 6 provides higher capacity efficiency and deterministic
    two-member fault tolerance, but has more parity-maintenance
    overhead.
    
