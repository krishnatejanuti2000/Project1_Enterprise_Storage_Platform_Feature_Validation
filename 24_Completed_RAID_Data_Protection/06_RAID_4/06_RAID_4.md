# RAID 4 (Block-Level Striping with Dedicated Parity)

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

---

# 1. Introduction

RAID 4 is a historical RAID level that provides data protection using block-level striping combined with one dedicated parity disk.

RAID 4 was developed as an improvement over RAID 3 by changing the striping method from byte-level striping to block-level striping.

The primary architectural characteristics of RAID 4 are:

- Block-level striping
- Dedicated parity disk
- XOR-based parity
- Single-disk fault tolerance
- Independent block-level I/O

Although RAID 4 improved the I/O flexibility of RAID 3, it retained the dedicated parity-disk bottleneck and therefore became a legacy RAID architecture.

---

# 2. Why RAID 4 Was Introduced

RAID 3 uses byte-level striping.

This tightly couples the data disks during I/O operations.

RAID 4 changed the data striping method to block-level striping.

Therefore:

```text
RAID 3
Byte-level striping
        +
Dedicated parity


RAID 4
Block-level striping
        +
Dedicated parity
```

The goal was to allow more independent block-level I/O while retaining parity-based fault tolerance.

---

# 3. What is RAID 4?

RAID 4 is a RAID level that distributes data using block-level striping across multiple data disks and stores parity information on one dedicated parity disk.

Parity is calculated using XOR.

RAID 4 can tolerate the failure of one member disk.

---

# 4. RAID 4 Architecture

Conceptually:

```text
                Application
                      |
                      v
              RAID Controller
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       Disk 1      Disk 2      Disk 3
        Data        Data        Data
                      |
                      |
                      v
                 Parity Disk
```

A more complete representation is:

```text
Disk 1 → Data Blocks
Disk 2 → Data Blocks
Disk 3 → Data Blocks
Disk 4 → Dedicated Parity
```

The operating system sees the RAID array as one logical storage device.

---

# 5. RAID 3 vs RAID 4

The major architectural difference is the striping technique.

```text
RAID 3
→ Byte-level striping
→ Dedicated parity disk


RAID 4
→ Block-level striping
→ Dedicated parity disk
```

RAID 4 therefore retains the parity model of RAID 3 but changes the data distribution method.

---

# 6. Minimum Number of Drives

RAID 4 requires a minimum of three drives.

Conceptually:

```text
Disk 1 → Data
Disk 2 → Data
Disk 3 → Dedicated Parity
```

At least two data disks are required for striping, while one disk is dedicated to parity.

Therefore:

```text
Minimum RAID 4 configuration = 3 disks
```

---

# 7. Core Technique – Block-Level Striping

RAID 4 distributes data at block-level granularity.

For example:

```text
Disk 1      Disk 2      Disk 3
 Block A     Block B     Block C
 Block D     Block E     Block F
 Block G     Block H     Block I
```

Each block remains together on a particular disk.

This allows independent block-level I/O.

For example:

```text
Read Block B
      ↓
Disk 2
```

or:

```text
Read Block D
      ↓
Disk 1
```

This provides more independent I/O behavior than RAID 3's byte-level striping.

---

# 8. RAID Stripe

A RAID stripe represents the corresponding data blocks across the data disks together with their associated parity.

Example:

```text
┌────────┬────────┬────────┬────────┐
│ Disk 1 │ Disk 2 │ Disk 3 │ Parity │
├────────┼────────┼────────┼────────┤
│   A    │   B    │   C    │   P    │
└────────┴────────┴────────┴────────┘
```

The parity corresponds to the data blocks in that stripe.

---

# 9. Dedicated Parity

RAID 4 uses one dedicated parity disk.

For a stripe containing:

```text
Disk 1 → A
Disk 2 → B
Disk 3 → C
```

the parity is:

```text
P = A XOR B XOR C
```

The calculated parity is stored on the dedicated parity disk.

The parity disk contains redundancy information rather than a mirror copy of the application data.

---

# 10. XOR Parity

RAID 4 uses XOR to calculate parity.

Example:

```text
A = 1011
B = 0101
C = 1100
```

First:

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
Parity = 0010
```

The stripe becomes:

```text
Disk 1 → 1011
Disk 2 → 0101
Disk 3 → 1100
Parity → 0010
```

---

# 11. Data Reconstruction

If one data block is missing, the controller can reconstruct it using the surviving data and parity.

Suppose Disk 2 fails:

```text
A = 1011
C = 1100
P = 0010
```

The controller calculates:

```text
B = A XOR C XOR P
```

Therefore:

```text
1011 XOR 1100 = 0111

0111 XOR 0010 = 0101
```

Recovered:

```text
B = 0101
```

---

# 12. Normal Read Flow

During a healthy read:

```text
Application
    |
    v
RAID Controller
    |
    v
Required Data Disk
    |
    v
Application
```

For example:

```text
Read Block B
      |
      v
Disk 2
```

The parity disk is normally not required for every healthy read.

---

# 13. Independent Reads

Because RAID 4 uses block-level striping, independent requests can target different data disks.

For example:

```text
Request 1 → Block A → Disk 1

Request 2 → Block B → Disk 2

Request 3 → Block C → Disk 3
```

This provides more independent I/O behavior than RAID 3.

---

# 14. Degraded Read

If a data disk fails:

```text
Disk 1 → Data
Disk 2 → Failed
Disk 3 → Data
Parity → Available
```

and the application requests data from Disk 2, the controller reconstructs the missing block.

Conceptually:

```text
Remaining Data
      +
    Parity
      |
      v
     XOR
      |
      v
Missing Data
```

The reconstructed data is returned to the application.

---

# 15. Normal Write Flow

During a write:

1. The application sends data to the RAID controller.
2. The controller places data blocks on the appropriate data disks.
3. The controller calculates parity.
4. Data is written to the data members.
5. Parity is written to the dedicated parity disk.

Conceptually:

```text
Application
    |
    v
RAID Controller
    |
    +---- Data Blocks ----> Data Disks
    |
    +---- Parity ---------> Parity Disk
```

---

# 16. Full-Stripe Write

A full-stripe write provides the complete new data for the data portion of a stripe.

For example:

```text
New A
New B
New C
```

The controller can directly calculate:

```text
New P = New A XOR New B XOR New C
```

It then writes:

```text
New A → Disk 1
New B → Disk 2
New C → Disk 3
New P → Parity Disk
```

No old data or old parity is required to calculate the new parity.

---

# 17. Partial-Stripe Write

A partial write changes only part of an existing stripe.

For example:

```text
Old B = 0101
New B = 0111
```

The parity must be updated because the old parity was calculated using the old B.

The controller can calculate:

```text
New P =
Old P XOR Old B XOR New B
```

This introduces additional parity-related processing.

---

# 18. Dedicated Parity Bottleneck

The major architectural limitation of RAID 4 is its dedicated parity disk.

All parity information is stored on one physical disk.

Conceptually:

```text
Data Disk 1 ─┐
Data Disk 2 ─┤
Data Disk 3 ─┤
Data Disk 4 ─┤
Data Disk N ─┘
       |
       v
Dedicated Parity Disk
```

As write workload increases, parity updates continue to converge on the same disk.

This can create a performance bottleneck.

---

# 19. Performance

RAID 4 improves independent block-level I/O compared with RAID 3.

Benefits include:

* Independent block access
* Parallel access to different data disks
* Good sequential throughput
* Better flexibility for independent block requests

However, write performance can be limited by the dedicated parity disk.

---

# 20. Degraded Write

If one data disk fails:

```text
Disk 1 → Data
Disk 2 → Failed
Disk 3 → Data
Parity → Available
```

The array continues operating in degraded mode.

The controller must maintain the parity relationship while the failed member is unavailable.

Reads for missing data require reconstruction.

Writes also introduce additional RAID management and parity work.

---

# 21. Data-Disk Failure

RAID 4 can tolerate one data-disk failure.

Example:

```text
Disk 1 → Data
Disk 2 → Failed
Disk 3 → Data
Parity → Available
```

The controller can reconstruct missing blocks using surviving data and parity.

The array remains available but operates in degraded mode.

---

# 22. Parity-Disk Failure

If the dedicated parity disk fails:

```text
Data Disk 1 → Available
Data Disk 2 → Available
Data Disk 3 → Available
Parity Disk → Failed
```

The existing application data remains available because the data itself is still present.

However, redundancy is temporarily lost.

After replacing the parity disk, parity can be regenerated from the surviving data.

Conceptually:

```text
Data Disk 1 ─┐
Data Disk 2 ─┼──> XOR
Data Disk 3 ─┘
              |
              v
        New Parity
              |
              v
    Replacement Parity Disk
```

---

# 23. Multiple Disk Failure

RAID 4 provides single-disk fault tolerance.

Therefore:

```text
1 disk failure → Recoverable

2 disk failures → Not fully recoverable
```

If two required members fail before recovery completes, the available parity information is insufficient to uniquely reconstruct both missing members.

---

# 24. Rebuild Process

After replacing a failed data disk:

```text
Disk Failure
     |
     v
Degraded RAID
     |
     v
Replacement Disk
     |
     v
Read Surviving Data + Parity
     |
     v
Reconstruct Missing Blocks
     |
     v
Write Blocks to Replacement Disk
     |
     v
Rebuild Complete
     |
     v
Healthy RAID
```

The rebuild restores full redundancy.

---

# 25. Reconstruction vs Rebuild

### Reconstruction

Reconstruction is the calculation of missing data.

```text
Remaining Data
      +
    Parity
      |
      v
     XOR
      |
      v
Missing Data
```

### Rebuild

Rebuild restores the reconstructed data onto a replacement disk.

```text
Reconstructed Data
        |
        v
Replacement Disk
```

They are related but different concepts.

---

# 26. Capacity Calculation

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

Usable capacity:

```text
(4 - 1) × 4 TB
= 12 TB
```

---

# 27. Why RAID 4 Became Legacy

RAID 4 improved RAID 3 by introducing block-level striping.

However, it retained the dedicated parity disk.

The primary limitation is:

```text
Many data disks
      |
      v
One parity disk
      |
      v
Parity bottleneck
```

Other limitations include:

* Dedicated parity bottleneck
* Small-write parity overhead
* Limited write scalability
* Single-disk fault tolerance

Later RAID architectures introduced distributed parity to overcome the centralized parity limitation.

---

# 28. Enterprise Use Cases

RAID 4 is not commonly deployed in modern enterprise storage.

It is primarily important for understanding RAID evolution and the transition from dedicated parity toward distributed parity architectures.

Historically, its block-level striping made it suitable for workloads requiring independent block access and sequential throughput.

---

# 29. Customer Scenario

### Customer Requirement

A customer requires:

* Block-level striping
* Single-disk fault tolerance
* Independent block I/O

### Historical RAID 4 Characteristics

RAID 4 provides:

* Block-level striping
* Dedicated parity
* XOR-based fault tolerance
* Single-disk protection

However, its dedicated parity disk can become a bottleneck under write-heavy workloads.

### Modern Recommendation

RAID 4 would generally not be selected for a modern enterprise deployment because more scalable RAID architectures are available.

---

# 30. Practical Validation

RAID 4 is treated primarily as a theoretical and architectural RAID level in this project.

The validation focus is:

* Block-level striping
* Dedicated parity
* XOR parity
* Normal reads
* Normal writes
* Partial writes
* Degraded operation
* Data reconstruction
* Parity-disk failure
* Data-disk failure
* Rebuild behavior
* Performance limitations
* Dedicated parity bottleneck

---

# 31. Summary

RAID 4 uses block-level striping with one dedicated parity disk.

It was developed as an improvement over RAID 3 by replacing byte-level striping with block-level striping.

RAID 4 provides:

* Block-level striping
* XOR-based parity
* Single-disk fault tolerance
* Independent block-level I/O
* Good sequential throughput

However, all parity information is concentrated on one dedicated parity disk.

This creates a centralized parity bottleneck and limits write scalability.

Therefore RAID 4 became a legacy RAID architecture and was eventually superseded by RAID designs using distributed parity.

The key concepts are:

* Block-level striping
* Dedicated parity
* XOR
* Single-disk fault tolerance
* Independent block I/O
* Degraded operation
* Reconstruction
* Rebuild
* Dedicated parity bottleneck
* Legacy architecture

