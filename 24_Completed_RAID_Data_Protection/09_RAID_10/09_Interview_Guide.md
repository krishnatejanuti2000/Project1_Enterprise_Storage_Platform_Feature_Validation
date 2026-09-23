# RAID 10 — Interview Guide

## 1. What is RAID 10?

RAID 10 combines:

```text
Mirroring
+
Striping
```

It provides redundancy through mirrored copies and performance
through striping.

It does not use parity.

---

# 2. Why is RAID 10 called RAID 1+0?

Because its conceptual design combines:

```text
RAID 1 → Mirroring
RAID 0 → Striping
```

The classic RAID 10 arrangement creates redundant copies and then
stripes across the mirrored groups.

It is different from RAID 0+1 because the nesting/order is different,
which produces different failure behavior.

---

# 3. What is the main advantage of RAID 10?

The main advantages are:

```text
High I/O performance
+
Mirrored redundancy
+
No parity calculation
+
Simple mirror-based rebuild
```

---

# 4. Does RAID 10 use parity?

No.

RAID 10 does not use:

```text
P parity
Q parity
XOR parity
Reed–Solomon
Galois Field arithmetic
```

Redundancy is provided through duplicate data copies.

---

# 5. What is the minimum number of RAID 10 members?

A conventional two-copy RAID 10 configuration requires:

```text
4 members
```

For example:

```text
D1 ↔ D2
D3 ↔ D4
```

---

# 6. What is the usable capacity of RAID 10?

For equal-sized members:

```text
Usable Capacity ≈ (N / 2) × Member Size
```

Therefore approximately 50% of raw capacity is usable.

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

---

# 7. How does RAID 10 store data?

Conceptually:

```text
D1 ↔ D2
D3 ↔ D4
```

Data is striped across the mirror groups, while each data chunk
has a redundant copy.

Example:

```text
D1    D2    D3    D4
 A     A     B     B
 C     C     D     D
 E     E     F     F
 G     G     H     H
```

---

# 8. How does RAID 10 handle a single member failure?

Suppose:

```text
D1 → failed
D2 → healthy
```

The data remains available through D2 because D2 contains the mirror
copy.

Therefore:

```text
Failed member
      ↓
Surviving mirror
      ↓
Data remains accessible
```

---

# 9. Can RAID 10 tolerate two drive failures?

The correct answer is:

> It depends on which members fail.

Two failures in different mirror groups can be tolerated:

```text
D1 ❌
D3 ❌

D2 ✅
D4 ✅
```

because every mirror group still has one surviving member.

But:

```text
D1 ❌
D2 ❌
```

loses an entire mirror group.

Therefore the array cannot provide the data stored only in that
mirror group.

---

# 10. What is the most important RAID 10 failure rule?

> Every mirror group must retain at least one surviving member.

Therefore:

```text
1 failure
→ normally survivable

2 failures in different mirror groups
→ can be survivable

2 failures in the same mirror group
→ mirror group lost
```

---

# 11. Why is RAID 10 different from RAID 6 regarding two failures?

RAID 6 uses:

```text
P + Q
```

to provide two independent parity relationships.

Therefore two-member failure is normally tolerated regardless of
which members fail.

RAID 10 uses:

```text
Mirroring
```

instead of parity.

Therefore two-member failure depends on mirror-group placement.

---

# 12. How does RAID 10 rebuild a failed member?

Suppose:

```text
D1 → failed
D2 → healthy
```

and replacement D1' is available.

The RAID layer reads the surviving mirror:

```text
D2
 ↓
Read data
 ↓
D1'
```

The missing mirror copy is recreated on the replacement.

No parity calculation is required.

---

# 13. How is RAID 10 rebuild different from RAID 6 rebuild?

### RAID 10

```text
Surviving mirror
      ↓
Copy data
      ↓
Replacement
```

### RAID 6

```text
Surviving members
      ↓
P + Q calculations
      ↓
Reconstruct missing data
      ↓
Replacement
```

RAID 10 rebuild is therefore conceptually a mirror-copy operation.

---

# 14. Does RAID 10 rebuild have performance impact?

Yes.

Even though no parity calculation is required, rebuild still
requires:

```text
Reads from surviving mirror
+
Writes to replacement
```

This adds I/O workload and can affect application performance.

---

# 15. What happens during RAID 10 degraded operation?

Suppose:

```text
D1 ❌
D2 ✅
D3 ✅
D4 ✅
```

The array remains operational, but the mirror containing D1 has lost
its redundancy.

The system continues using D2 for the affected data.

Until rebuild is complete, that mirror group has reduced protection.

---

# 16. What happens if both members of a mirror group fail?

Example:

```text
D1 ❌
D2 ❌
```

Then:

```text
D1 ❌ ↔ D2 ❌
```

The entire mirror group is lost.

Because RAID 10 has no parity, it cannot reconstruct the missing
data from the other mirror group.

---

# 17. What is a hot spare in RAID 10?

A hot spare is a standby drive that can be used as the rebuild target
after a member failure.

Conceptually:

```text
Member failure
      ↓
Hot spare
      ↓
Mirror-copy rebuild
      ↓
Redundancy restored
```

A hot spare is a recovery resource, not an additional RAID parity
level.

---

# 18. What are RAID 10 layouts in Linux `mdadm`?

Linux software RAID 10 supports layouts such as:

```text
near
far
offset
```

The layout determines how the multiple copies are positioned.

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

The number refers to the number of copies, not the number of mirror
groups.

The actual layout should be verified using:

```bash
sudo mdadm --detail /dev/md10
```

---

# 19. What is `near=2`?

`near=2` means:

```text
2 copies of each data chunk
```

using the near placement layout.

Conceptually:

```text
A A
B B
C C
D D
```

The exact physical placement is determined by the implementation.

---

# 20. What is `far` layout?

The far layout places the redundant copies farther apart in the
logical/device address space.

It changes copy placement while preserving the RAID 10 redundancy
model.

---

# 21. What is `offset` layout?

The offset layout places the second copy using an offset within the
RAID 10 layout.

Again, the purpose is to control copy placement rather than change
the basic redundancy model.

---

# 22. Does RAID 10 use a write penalty like RAID 5/6?

RAID 10 does not have the same parity-maintenance overhead because
there is no parity.

For a logical write, the RAID layer must maintain both mirror copies:

```text
New Data
 ↓
Copy 1
+
Copy 2
```

So writes still produce duplicate physical writes, but there is no
P/Q parity calculation.

---

# 23. Why is RAID 10 attractive for random I/O?

RAID 10 provides:

```text
Striping
+
Multiple disks
+
Multiple mirror copies
+
No parity calculation
```

This can provide strong I/O performance, particularly for workloads
with high random-read/write requirements.

---

# 24. RAID 10 vs RAID 5

| Feature                 | RAID 5                 | RAID 10                      |
| ----------------------- | ---------------------- | ---------------------------- |
| Redundancy              | XOR parity             | Mirroring                    |
| Striping                | Yes                    | Yes                          |
| Parity                  | Yes                    | No                           |
| Usable capacity         | `(N-1) × size`         | Approximately `(N/2) × size` |
| Small-write parity work | Yes                    | No                           |
| Rebuild                 | Parity reconstruction  | Mirror copy                  |
| Two-member failure      | Normally not tolerated | Depends on failure placement |

---

# 25. RAID 10 vs RAID 6

| Feature                 | RAID 6             | RAID 10                      |
| ----------------------- | ------------------ | ---------------------------- |
| Redundancy              | P + Q              | Mirroring                    |
| Parity                  | Yes                | No                           |
| Usable capacity         | `(N-2) × size`     | Approximately `(N/2) × size` |
| Two-member failure      | Normally tolerated | Depends on mirror placement  |
| Small-write parity work | Higher             | No parity calculation        |
| Rebuild                 | P/Q reconstruction | Mirror copy                  |
| Performance model       | Parity-based       | Mirror + stripe              |

---

# 26. Interview Scenario — One Drive Fails

### Question

A RAID 10 array loses one member. What happens?

### Answer

The array enters degraded mode, but the surviving mirror member still
contains the data. The RAID layer can continue serving requests from
the surviving copy.

---

# 27. Interview Scenario — Two Drives Fail

### Question

Two RAID 10 members fail. Is the array necessarily lost?

### Answer

No. It depends on the failure pattern.

If the two failed members belong to different mirror groups, the array
can remain operational because each group retains one surviving member.

If both failed members belong to the same mirror group, that entire
mirror group is lost.

---

# 28. Interview Scenario — Why No Parity?

### Question

Why doesn't RAID 10 use parity?

### Answer

RAID 10 provides redundancy through mirroring. Every data region has
multiple copies, so parity is unnecessary.

---

# 29. Interview Scenario — Rebuild

### Question

How does RAID 10 rebuild a failed member?

### Answer

The RAID layer reads the surviving mirror copy and writes the same
data to the replacement member. It is primarily a mirror-copy
operation rather than a parity reconstruction.

---

# 30. Interview Scenario — RAID 10 vs RAID 6

### Question

Why might an application choose RAID 10 over RAID 6?

### Answer

RAID 10 can provide strong I/O performance and avoids parity
calculation overhead. The trade-off is approximately 50% usable
capacity and failure tolerance that depends on mirror placement.

---

# 31. Interview Scenario — Hot Spare

### Question

What does a hot spare do in RAID 10?

### Answer

A hot spare provides a standby replacement target. After a member
failure, the surviving mirror can be copied onto the spare to restore
the mirror.

---

# 32. Interview Scenario — Capacity

### Question

What is the usable capacity of a 4-drive RAID 10 with 2 TB drives?

### Answer

Raw capacity:

```text
4 × 2 TB = 8 TB
```

Approximate usable capacity:

```text
8 TB / 2 = 4 TB
```

---

# 33. Interview Scenario — RAID 10 Failure Rule

### Question

What is the most important failure rule in RAID 10?

### Answer

Every mirror group must retain at least one surviving member.

---

# 34. Strong Interview Answer — Explain RAID 10

> RAID 10 combines mirroring and striping. It provides redundancy
> through multiple copies of the data and performance through
> striping across the mirrored groups. Unlike RAID 5 and RAID 6,
> RAID 10 does not use parity. A single member failure is normally
> survivable because its mirror remains available. Two member
> failures can also be survived when they occur in different mirror
> groups, but losing both members of the same mirror group causes
> loss of that mirrored data. Rebuild is primarily a mirror-copy
> operation from the surviving member to the replacement.

---

# 35. Quick Interview Revision

```text
RAID 10
→ Mirroring + Striping

Redundancy
→ Multiple copies

Parity
→ None

Minimum conventional members
→ 4

Usable capacity
→ Approximately 50%

1 failure
→ Normally survives

2 failures
→ Depends on mirror placement

Same mirror group lost
→ Data unavailable

Rebuild
→ Copy surviving mirror to replacement

Hot spare
→ Rebuild target

mdadm layouts
→ near / far / offset

near=2
→ 2 copies using near layout
```

# 36. Interview Checklist

```text
[ ] Explain RAID 10
[ ] Explain RAID 1 + RAID 0 relationship
[ ] Explain mirror sets
[ ] Explain striping
[ ] Explain why there is no parity
[ ] Explain single-member failure
[ ] Explain two-member failure patterns
[ ] Explain the mirror-group rule
[ ] Explain RAID 10 rebuild
[ ] Explain hot spare
[ ] Calculate capacity
[ ] Explain random I/O advantage
[ ] Explain near/far/offset
[ ] Explain near=2
[ ] Compare RAID 10 with RAID 5
[ ] Compare RAID 10 with RAID 6
```

