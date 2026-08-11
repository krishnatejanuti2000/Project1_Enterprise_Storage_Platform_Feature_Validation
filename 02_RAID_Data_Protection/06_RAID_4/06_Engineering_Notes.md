# Engineering Notes – RAID 4

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 4**

---

# 1. Engineering Perspective

RAID 4 is a historical parity-based RAID architecture that uses block-level striping across multiple data disks and one dedicated parity disk.

The primary engineering characteristics are:

- Block-level striping
- Dedicated parity
- XOR-based parity calculation
- Single-disk fault tolerance
- Independent block-level I/O
- Good sequential throughput
- Centralized parity workload

RAID 4 is important for understanding the evolution from RAID 3 toward distributed-parity RAID architectures.

---

# 2. RAID 4 Architecture

A RAID 4 array consists of:

- Multiple data disks
- One dedicated parity disk

Conceptually:

```text
Data Disk 1 ──┐
Data Disk 2 ──┤
Data Disk 3 ──┤
Data Disk N ──┤
              ↓
       Dedicated Parity Disk
```

The RAID controller manages:

* Data placement
* Block-level striping
* Parity calculation
* RAID metadata
* Member health
* Failure handling
* Reconstruction
* Rebuild

---

# 3. Block-Level Striping

The defining characteristic of RAID 4 is block-level striping.

Conceptually:

```text
Disk 1      Disk 2      Disk 3
--------    --------    --------
Block A     Block B     Block C
Block D     Block E     Block F
Block G     Block H     Block I
```

Each block remains together on a particular disk.

This allows independent block-level requests to target individual disks.

For example:

```text
Read Block B
     ↓
Disk 2
```

This is different from RAID 3, where data is striped at byte level.

---

# 4. RAID 3 vs RAID 4

The primary architectural change is the striping granularity.

```text
RAID 3
→ Byte-level striping
→ Dedicated parity


RAID 4
→ Block-level striping
→ Dedicated parity
```

The parity mechanism remains fundamentally similar.

The major improvement is the ability to perform more independent block-level I/O.

---

# 5. Stripe Organization

A RAID 4 stripe contains corresponding blocks from the data disks and the associated parity.

Example:

```text
┌────────┬────────┬────────┬────────┐
│ Disk 1 │ Disk 2 │ Disk 3 │ Parity │
├────────┼────────┼────────┼────────┤
│   A    │   B    │   C    │   P    │
└────────┴────────┴────────┴────────┘
```

The parity is calculated from the data blocks:

```text
P = A XOR B XOR C
```

---

# 6. Dedicated Parity

RAID 4 uses one dedicated parity disk.

For:

```text
Disk 1 → A
Disk 2 → B
Disk 3 → C
```

the parity is:

```text
P = A XOR B XOR C
```

The parity is stored on the dedicated parity disk.

The parity disk stores redundancy information rather than a complete copy of the application data.

---

# 7. XOR Parity

XOR is the mathematical mechanism used for RAID 4 parity.

Example:

```text
A = 1011
B = 0101
C = 1100
```

Calculate:

```text
1011 XOR 0101
= 1110
```

Then:

```text
1110 XOR 1100
= 0010
```

Therefore:

```text
P = 0010
```

---

# 8. Data Reconstruction

If one data block is missing, the controller can reconstruct it using the surviving data and parity.

For example:

```text
A = 1011
C = 1100
P = 0010
```

Missing:

```text
B
```

Calculation:

```text
B = A XOR C XOR P
```

Therefore:

```text
1011 XOR 1100
= 0111

0111 XOR 0010
= 0101
```

Recovered:

```text
B = 0101
```

---

# 9. Normal Read Path

During a healthy read:

```text
Application
     ↓
RAID Controller
     ↓
Required Data Disk
     ↓
Application
```

The parity disk is normally not required for every healthy read.

Example:

```text
Read Block B
     ↓
Disk 2
```

---

# 10. Independent Block Reads

Because RAID 4 uses block-level striping, different requests can target different disks.

Example:

```text
Request 1 → Block A → Disk 1

Request 2 → Block B → Disk 2

Request 3 → Block C → Disk 3
```

This allows more independent I/O than RAID 3's byte-level synchronized architecture.

---

# 11. Degraded Read

Suppose Disk 2 fails:

```text
Disk 1 → Data
Disk 2 → FAILED
Disk 3 → Data
Parity → Available
```

If the application requests a block that was stored on Disk 2, the controller reconstructs it.

Conceptually:

```text
Remaining Data
      +
    Parity
      ↓
     XOR
      ↓
Missing Data
```

The reconstructed data is returned to the application.

---

# 12. Normal Write

During a normal write:

```text
Application
     ↓
RAID Controller
     ↓
Data blocks
     ↓
Data Disks
```

The controller also calculates the corresponding parity:

```text
Data
 ↓
XOR
 ↓
Parity
 ↓
Dedicated Parity Disk
```

Therefore a write updates both:

```text
Data member
+
Parity member
```

---

# 13. Full-Stripe Write

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

Then:

```text
New A → Disk 1
New B → Disk 2
New C → Disk 3
New P → Parity Disk
```

The controller does not need the old parity to calculate the new parity.

---

# 14. Partial-Stripe Write

A partial-stripe write changes only part of an existing stripe.

Example:

```text
Old B = 0101
New B = 0111
```

The parity must be updated because the original parity was calculated using the old B.

The controller can calculate:

```text
New P =
Old P XOR Old B XOR New B
```

This introduces additional parity-related processing.

---

# 15. Small-Write Overhead

Small writes can require additional work because the controller must maintain parity consistency.

Conceptually:

```text
Old Data
Old Parity
New Data
New Parity
```

The parity update adds I/O and processing overhead.

This becomes increasingly important for workloads dominated by small random writes.

---

# 16. Dedicated Parity Bottleneck

The major engineering limitation of RAID 4 is its dedicated parity disk.

All parity updates are directed to the same physical member.

Conceptually:

```text
Data Disk 1 ──┐
Data Disk 2 ──┤
Data Disk 3 ──┤
Data Disk 4 ──┤
Data Disk N ──┘
       ↓
Dedicated Parity Disk
```

Even when many data disks are available, parity activity remains concentrated on one disk.

---

# 17. Write Scalability

Consider:

```text
4 Data Disks + 1 Parity Disk
```

versus:

```text
8 Data Disks + 1 Parity Disk
```

versus:

```text
16 Data Disks + 1 Parity Disk
```

The number of data members increases.

However:

```text
Parity Disks = 1
```

Therefore the parity workload remains centralized.

This limits write scalability.

---

# 18. Performance Characteristics

RAID 4 provides:

* Block-level independent I/O
* Good sequential throughput
* Parallel access to different data disks
* Single-disk fault tolerance

However, performance can be limited by:

* Dedicated parity writes
* Small-write parity processing
* Parity disk saturation
* Rebuild activity

---

# 19. Degraded Operation

After a single data-disk failure:

```text
Disk 1 → Data
Disk 2 → Failed
Disk 3 → Data
Parity → Available
```

The array continues operating in degraded mode.

When data from the failed disk is requested:

```text
Surviving Data
      +
    Parity
      ↓
Reconstruction
      ↓
Requested Data
```

The array remains available but performance can be reduced.

---

# 20. Parity-Disk Failure

If the dedicated parity disk fails:

```text
Data Disk 1 → Available
Data Disk 2 → Available
Data Disk 3 → Available
Parity Disk → Failed
```

The application data remains available because the data blocks still exist.

However:

```text
Redundancy = Temporarily Lost
```

After replacing the parity disk, parity can be regenerated:

```text
Data Disk 1 ──┐
Data Disk 2 ──┤
Data Disk 3 ──┘
       ↓
      XOR
       ↓
New Parity
       ↓
Replacement Parity Disk
```

---

# 21. Multiple Disk Failure

RAID 4 provides single-disk fault tolerance.

Therefore:

```text
1 disk failure
→ Recoverable


2 disk failures
→ Not fully recoverable
```

The reason is that a single parity value provides only one independent redundancy equation.

If two members are missing, there are multiple unknown data values but insufficient independent parity information.

---

# 22. Reconstruction

Reconstruction means calculating missing data.

Example:

```text
Missing B

B = A XOR C XOR P
```

The controller performs the calculation when the missing data is required or during recovery processing.

---

# 23. Rebuild

After replacing the failed disk, the controller reconstructs the missing blocks and writes them to the replacement disk.

Lifecycle:

```text
Healthy
   ↓
Disk Failure
   ↓
Degraded
   ↓
Replacement Disk
   ↓
Reconstruct Missing Blocks
   ↓
Write to Replacement Disk
   ↓
Rebuild Complete
   ↓
Healthy
```

---

# 24. Rebuild Performance

During rebuild:

```text
Normal Application I/O
        +
Rebuild I/O
        ↓
Higher Storage Activity
        ↓
Potential Performance Degradation
```

The surviving disks must service both application requests and rebuild operations.

The parity disk may also experience additional workload.

---

# 25. Capacity

For equal-sized disks:

```text
Usable Capacity =
(N - 1) × Disk Capacity
```

Example:

```text
6 × 4 TB

Raw Capacity = 24 TB
Parity = 4 TB

Usable = 20 TB
```

For unequal-sized disks:

```text
Usable Capacity =
(N - 1) × Smallest Member Capacity
```

Example:

```text
4 TB
4 TB
6 TB
6 TB
```

Effective member size:

```text
4 TB
```

Usable:

```text
(4 - 1) × 4 TB
= 12 TB
```

---

# 26. Engineering Validation Areas

When validating a RAID 4 implementation, verify:

### Architecture

* RAID level
* Number of members
* Block-level striping
* Dedicated parity member

### Capacity

* Member sizes
* Effective member size
* Usable capacity
* Parity capacity

### Data Protection

* Healthy state
* Data-disk failure
* Parity-disk failure
* Degraded state
* Reconstruction
* Replacement disk
* Rebuild
* Return to healthy state

### Performance

* Sequential throughput
* Independent block I/O
* Small-write behavior
* Parity disk utilization
* Rebuild impact
* Parity bottleneck

---

# 27. Engineering Troubleshooting Points

When investigating a RAID 4 problem:

```text
1. Check RAID state
        ↓
2. Identify failed member
        ↓
3. Check data/parity role
        ↓
4. Check member health
        ↓
5. Determine degraded condition
        ↓
6. Validate data accessibility
        ↓
7. Investigate reconstruction/rebuild
        ↓
8. Validate final RAID state
```

The key distinction is whether the failed member is:

```text
Data Disk
```

or:

```text
Dedicated Parity Disk
```

because the recovery behavior differs.

---

# 28. RAID 4 Engineering Takeaways

* RAID 4 uses block-level striping.
* RAID 4 uses one dedicated parity disk.
* Parity is calculated using XOR.
* RAID 4 provides single-disk fault tolerance.
* Block-level striping allows more independent I/O than RAID 3.
* Healthy reads normally do not require parity.
* Full-stripe writes can calculate parity directly.
* Partial writes require parity updates.
* Small writes introduce additional overhead.
* The dedicated parity disk is the major scalability limitation.
* A parity-disk failure removes redundancy but does not immediately remove the stored data.
* A data-disk failure requires reconstruction when its data is accessed.
* Rebuild restores the missing member after replacement.
* RAID 4 is primarily a legacy architecture.

