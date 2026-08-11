# Interview Guide – RAID 4

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 4**

---

# 1. What is RAID 4?

RAID 4 is a historical RAID level that uses **block-level striping** across multiple data disks and **one dedicated parity disk**.

Parity is calculated using XOR.

RAID 4 can tolerate one disk failure.

---

# 2. What is the minimum number of disks required for RAID 4?

The minimum is:

```text
3 disks
````

They are used as:

```text
2 Data Disks
1 Dedicated Parity Disk
```

At least two data disks are required for block-level striping.

---

# 3. Why was RAID 4 introduced?

RAID 4 was introduced as an improvement over RAID 3.

The major change was the striping method:

```text
RAID 3 → Byte-level striping
RAID 4 → Block-level striping
```

RAID 4 therefore provides more independent block-level I/O while retaining dedicated parity.

---

# 4. What is the main architectural difference between RAID 3 and RAID 4?

The primary difference is the **striping granularity**.

| Feature            | RAID 3     | RAID 4      |
| ------------------ | ---------- | ----------- |
| Striping           | Byte-level | Block-level |
| Parity             | Dedicated  | Dedicated   |
| Parity calculation | XOR        | XOR         |
| Fault tolerance    | 1 disk     | 1 disk      |

RAID 4 changes the data striping method but does not eliminate the dedicated parity disk.

---

# 5. What is block-level striping?

Block-level striping means that complete blocks are distributed across the data disks.

Example:

```text
Disk 1 → Block A
Disk 2 → Block B
Disk 3 → Block C
```

A block remains together on its corresponding disk.

This allows independent block-level I/O.

---

# 6. What is the role of the dedicated parity disk?

The dedicated parity disk stores the parity information calculated from the corresponding data blocks.

For example:

```text
P = A XOR B XOR C
```

The parity disk stores redundancy information rather than a complete copy of the application data.

---

# 7. How is parity calculated in RAID 4?

RAID 4 uses XOR.

Example:

```text
A = 1011
B = 0101
C = 1100
```

Therefore:

```text
1011 XOR 0101 = 1110

1110 XOR 1100 = 0010
```

So:

```text
Parity = 0010
```

---

# 8. Why is XOR useful for RAID 4?

XOR is reversible.

If:

```text
P = A XOR B XOR C
```

then a missing value can be reconstructed using the remaining values and parity.

For example:

```text
B = A XOR C XOR P
```

This allows RAID 4 to recover from a single member failure.

---

# 9. What is a RAID 4 stripe?

A RAID 4 stripe contains corresponding data blocks across the data disks and the associated parity.

Example:

```text
┌────────┬────────┬────────┬────────┐
│ Disk 1 │ Disk 2 │ Disk 3 │ Parity │
├────────┼────────┼────────┼────────┤
│   A    │   B    │   C    │   P    │
└────────┴────────┴────────┴────────┘
```

where:

```text
P = A XOR B XOR C
```

---

# 10. How does RAID 4 perform a normal read?

In a healthy array:

```text
Application
    ↓
RAID Controller
    ↓
Required Data Disk
    ↓
Application
```

For example:

```text
Read Block B
    ↓
Disk 2
```

The parity disk is normally not required for every healthy read.

---

# 11. Why does RAID 4 provide more independent I/O than RAID 3?

RAID 4 uses block-level striping.

Therefore different blocks can be located on different disks:

```text
Block A → Disk 1
Block B → Disk 2
Block C → Disk 3
```

Independent requests can therefore target different data members.

RAID 3 instead uses byte-level striping and tightly synchronized disk access.

---

# 12. What happens during a degraded read?

Suppose Disk 2 fails:

```text
Disk 1 → A
Disk 2 → FAILED
Disk 3 → C
Parity → P
```

If the application requests B, the controller reconstructs:

```text
B = A XOR C XOR P
```

The reconstructed block is then returned to the application.

---

# 13. How does RAID 4 perform a normal write?

The basic write path is:

```text
Application
    ↓
RAID Controller
    ↓
Data blocks → Data Disks
    ↓
Parity calculation
    ↓
Parity → Dedicated Parity Disk
```

Therefore a write updates both data and parity.

---

# 14. What is a full-stripe write?

A full-stripe write provides the complete new data portion of a stripe.

For example:

```text
New A
New B
New C
```

The controller can calculate:

```text
New P = New A XOR New B XOR New C
```

and write the complete stripe.

This avoids using the old data and old parity to calculate the new parity.

---

# 15. What is a partial-stripe write?

A partial-stripe write changes only part of an existing stripe.

For example:

```text
Old B = 0101
New B = 0111
```

The controller must update parity.

A parity-update approach is:

```text
New P =
Old P XOR Old B XOR New B
```

This introduces additional processing and I/O.

---

# 16. Why can small writes be expensive in RAID 4?

A small write may require the controller to manage:

* Old data
* Old parity
* New data
* New parity

The parity update creates additional work compared with a simple non-parity write.

---

# 17. What is RAID 4's main performance limitation?

The main limitation is the **dedicated parity disk**.

All parity updates are directed to one physical disk.

Conceptually:

```text
Data Disk 1 ──┐
Data Disk 2 ──┤
Data Disk 3 ──┤
Data Disk 4 ──┤
              ↓
       Dedicated Parity Disk
```

This can create a parity bottleneck.

---

# 18. Why is the dedicated parity disk a bottleneck?

Suppose many data disks are processing writes:

```text
Write → Disk 1 + Parity
Write → Disk 2 + Parity
Write → Disk 3 + Parity
Write → Disk 4 + Parity
```

The data workload is distributed.

However, parity updates converge on:

```text
One Parity Disk
```

The parity disk can therefore become saturated and limit write scalability.

---

# 19. Does RAID 4 solve RAID 3's dedicated parity problem?

**No.**

RAID 4 solves a different problem.

It changes:

```text
RAID 3 → Byte-level striping
```

to:

```text
RAID 4 → Block-level striping
```

But both retain:

```text
One Dedicated Parity Disk
```

Therefore RAID 4 improves independent block I/O but retains the centralized parity bottleneck.

---

# 20. What happens if a data disk fails?

RAID 4 can tolerate one data-disk failure.

Example:

```text
Disk 1 → Data
Disk 2 → FAILED
Disk 3 → Data
Parity → Available
```

The missing data can be reconstructed:

```text
Missing Data =
Remaining Data XOR Parity
```

The array operates in degraded mode.

---

# 21. What happens if the parity disk fails?

The application data remains available because the data blocks are still present on the data disks.

However, the array temporarily loses redundancy.

After replacing the parity disk, parity can be regenerated from the surviving data.

---

# 22. What happens if two disks fail?

RAID 4 provides single-disk fault tolerance.

Therefore:

```text
1 Disk Failure → Recoverable
2 Disk Failures → Not fully recoverable
```

A single parity calculation does not provide enough independent information to reconstruct two missing members.

---

# 23. What is reconstruction?

Reconstruction means calculating missing data using surviving data and parity.

Example:

```text
Missing B

B = A XOR C XOR P
```

Reconstruction can occur when the missing data is required during degraded operation.

---

# 24. What is rebuild?

Rebuild is the process of restoring the missing member after a replacement disk is installed.

```text
Disk Failure
    ↓
Degraded RAID
    ↓
Replacement Disk
    ↓
Reconstruct Missing Blocks
    ↓
Write to Replacement Disk
    ↓
Rebuild Complete
    ↓
Healthy RAID
```

---

# 25. What is the difference between reconstruction and rebuild?

**Reconstruction:**

```text
Calculate missing data
```

**Rebuild:**

```text
Restore the missing data onto a replacement disk
```

They are related but not identical.

---

# 26. How is RAID 4 capacity calculated?

For equal-sized disks:

```text
Usable Capacity =
(N - 1) × Disk Capacity
```

Example:

```text
6 × 4 TB

Raw Capacity = 24 TB
Parity Capacity = 4 TB
Usable Capacity = 20 TB
```

For unequal-sized disks, the smallest member capacity determines the effective RAID member size.

---

# 27. Why did RAID 4 become legacy?

RAID 4 improved RAID 3 by introducing block-level striping.

However, it retained the dedicated parity disk.

The major limitation is:

```text
Many Data Disks
      ↓
One Dedicated Parity Disk
      ↓
Parity Bottleneck
```

Other limitations include:

* Small-write parity overhead
* Limited write scalability
* Single-disk fault tolerance

Modern RAID architectures provide more scalable parity designs.

---

# 28. RAID 4 vs RAID 0

| Feature               | RAID 0              | RAID 4          |
| --------------------- | ------------------- | --------------- |
| Striping              | Block-level         | Block-level     |
| Parity                | No                  | Yes             |
| Dedicated parity disk | No                  | Yes             |
| Fault tolerance       | None                | 1 disk          |
| Usable capacity       | 100%                | N-1 disks worth |
| Write overhead        | Low                 | Higher          |
| Parity bottleneck     | No                  | Yes             |
| Modern role           | Performance-focused | Legacy          |

---

# 29. RAID 4 vs RAID 3

| Feature               | RAID 3     | RAID 4      |
| --------------------- | ---------- | ----------- |
| Striping              | Byte-level | Block-level |
| Parity                | Dedicated  | Dedicated   |
| Parity calculation    | XOR        | XOR         |
| Fault tolerance       | 1 disk     | 1 disk      |
| Independent block I/O | Limited    | Better      |
| Parity bottleneck     | Yes        | Yes         |
| Modern role           | Legacy     | Legacy      |

---

# 30. Interview Scenario

### Question

A customer wants block-level striping and single-disk fault tolerance.

Would you recommend RAID 4 for a modern enterprise deployment?

### Expected Answer

Generally, no.

RAID 4 provides block-level striping and single-disk fault tolerance, but all parity updates are concentrated on one dedicated parity disk.

This creates a scalability and write-performance bottleneck.

Modern enterprise RAID architectures provide more flexible parity distribution.

---

# 31. Interview Summary

Remember:

```text
RAID 4
│
├── Block-level striping
├── Dedicated parity disk
├── XOR parity
├── Minimum 3 disks
├── Single-disk fault tolerance
├── Independent block I/O
├── Good sequential performance
├── Small-write parity overhead
├── Dedicated parity bottleneck
└── Legacy architecture
```

The most important interview statement is:

> **RAID 4 uses block-level striping with a dedicated parity disk. It improves independent block I/O compared with RAID 3, but retains the centralized parity bottleneck, which limits write scalability and ultimately made RAID 4 a legacy architecture.**

