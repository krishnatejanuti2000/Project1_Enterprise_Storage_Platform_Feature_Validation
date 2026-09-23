# RAID 3 (Byte-Level Striping with Dedicated Parity)

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

---

# 1. Introduction

RAID 3 is a historical RAID level that provides data protection using byte-level striping combined with a dedicated parity disk.

Unlike RAID 2, which uses bit-level striping and Hamming-code-based ECC, RAID 3 uses parity information calculated using XOR.

RAID 3 was designed to provide high sequential I/O performance while allowing recovery from a single disk failure.

---

# 2. Why RAID 3 Was Invented

RAID 2 used bit-level striping and dedicated ECC disks.

This introduced significant complexity and storage overhead.

RAID 3 simplified the protection mechanism by replacing Hamming-code-based ECC with parity.

The architecture became:

    Data Disks
        +
    One Dedicated Parity Disk

RAID 3 therefore combines:

- Byte-level striping
- Dedicated parity
- XOR-based fault tolerance

---

# 3. What is RAID 3?

RAID 3 is a RAID level that distributes data at byte-level granularity across multiple data disks and stores parity information on one dedicated parity disk.

The RAID controller calculates parity using XOR.

RAID 3 can tolerate the failure of one member disk.

---

# 4. RAID 3 Architecture

Conceptually:

                Application
                     |
                     v
               RAID Controller
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Disk 1     Disk 2     Disk 3
        Data       Data       Parity

The data disks participate in byte-level striping.

The dedicated parity disk stores parity information.

---

# 5. Minimum Number of Drives

RAID 3 requires a minimum of three drives.

Example:

    Disk 1 → Data
    Disk 2 → Data
    Disk 3 → Parity

At least two data disks are required for striping, while one disk is dedicated to parity.

Therefore:

    Minimum RAID 3 configuration = 3 disks

---

# 6. Core Technique – Byte-Level Striping

RAID 3 distributes data at byte-level granularity across the data disks.

Conceptually:

    Data:
    A B C D E F

    Data Disk 1:
    A C E

    Data Disk 2:
    B D F

The parity disk stores parity information corresponding to the data stripes.

Byte-level striping differs from:

    RAID 2 → Bit-level striping
    RAID 0 → Block/chunk-level striping

---

# 7. Dedicated Parity

RAID 3 uses one dedicated parity disk.

The RAID controller calculates parity from the corresponding data in a stripe.

For two data values:

    P = A XOR B

The calculated parity is stored on the dedicated parity disk.

The parity disk therefore contains protection information rather than a complete copy of the application data.

---

# 8. XOR Parity

RAID 3 uses XOR to calculate parity.

Example:

    A = 1011
    B = 0101

    A XOR B:

      1011
    XOR 0101
    --------
      1110

Therefore:

    Parity = 1110

If B is lost:

    B = A XOR Parity

      1011
    XOR 1110
    --------
      0101

The missing data can therefore be reconstructed.

---

# 9. Capacity Calculation

For equal-sized disks:

    Usable Capacity =
    (Number of Disks - 1) × Disk Capacity

Example:

    5 × 2 TB disks

    Raw Capacity:
    5 × 2 = 10 TB

    Parity Capacity:
    2 TB

    Usable Capacity:
    (5 - 1) × 2 = 8 TB

One disk's worth of capacity is used for parity.

For unequal-sized disks, the smallest member capacity limits the effective RAID member size.

---

# 10. Read Flow

During a normal healthy read:

    Application
        |
        v
    RAID Controller
        |
        v
    Data Disks
        |
        v
    Application

The controller reads the required data from the data members.

The parity disk does not normally need to participate in every healthy read.

If a data disk has failed, the controller can reconstruct the missing data using the surviving data and parity.

---

# 11. Write Flow

During a write:

1. The application sends data to the RAID controller.
2. The controller distributes the data across the data disks.
3. The controller calculates parity using XOR.
4. Data is written to the data disks.
5. Parity is written to the dedicated parity disk.

Conceptually:

    Application
        |
        v
    RAID Controller
        |
        +------ Data ------> Data Disks
        |
        +------ Parity ----> Parity Disk

---

# 12. Full-Stripe Write

When the complete data portion of a stripe is available, the controller can calculate parity directly from the new data.

Conceptually:

    New Data
       |
       v
    XOR Calculation
       |
       v
    New Parity

The controller then writes the data and corresponding parity.

---

# 13. Small Write

When only part of an existing stripe changes, the parity must also be updated.

The controller can use the old data, old parity, and new data to calculate the new parity.

Conceptually:

    Old Parity
        XOR
    Old Data
        XOR
    New Data
        |
        v
    New Parity

This introduces additional work compared with a simple non-parity write.

---

# 14. Degraded Mode

If one disk fails:

    Disk 1 → Data
    Disk 2 → FAILED
    Disk 3 → Parity

The RAID array enters a degraded state.

The application can continue accessing data because the missing information can be reconstructed.

For example:

    Missing Data = Remaining Data XOR Parity

RAID 3 can tolerate one disk failure.

---

# 15. Reconstruction

During degraded operation, when missing data is required, the RAID controller reconstructs it using surviving data and parity.

Conceptually:

    Remaining Data
          +
        Parity
          |
          v
         XOR
          |
          v
    Missing Data

The reconstructed data is then returned to the application.

---

# 16. Rebuild Process

After replacing the failed disk:

    Disk Failure
         |
         v
    Degraded RAID
         |
         v
    Replace Failed Disk
         |
         v
    Reconstruct Missing Data
         |
         v
    Write Data to Replacement Disk
         |
         v
    Rebuild Complete
         |
         v
    Healthy RAID

The rebuild restores the missing member and returns the array to a fully protected state.

---

# 17. Performance

RAID 3 can provide high sequential throughput because data is distributed across multiple data disks.

It is historically better suited to large sequential workloads than workloads dominated by small random I/O.

Small writes can require additional parity-related operations.

---

# 18. Dedicated Parity Bottleneck

A major limitation of RAID 3 is its dedicated parity disk.

All parity information is concentrated on one physical disk.

For example:

    Data Disk 1 \
    Data Disk 2  \
    Data Disk 3   ---> Dedicated Parity Disk
    Data Disk 4  /
    Data Disk 5 /

As workload increases, the single parity disk can become a bottleneck.

This limits scalability.

---

# 19. Why RAID 3 Became Legacy

RAID 3 became obsolete because of several architectural limitations.

The most important limitation was the dedicated parity disk.

Other limitations include:

- Byte-level striping
- Tight disk synchronization
- Poor scalability for modern workloads
- Additional overhead for small writes
- Dedicated parity bottleneck

Later RAID architectures provided more flexible parity distribution.

RAID 3 is therefore primarily of historical importance today.

---

# 20. Enterprise Use Cases

RAID 3 is not commonly deployed in modern enterprise storage.

Historically, its architecture was suitable for workloads involving:

- Large sequential transfers
- Multimedia workloads
- Large datasets
- Sequential streaming workloads

Modern enterprise storage generally uses more flexible RAID architectures.

---

# 21. Summary

RAID 3 uses byte-level striping across multiple data disks and one dedicated parity disk.

Parity is calculated using XOR and provides protection against a single disk failure.

RAID 3 can provide high sequential throughput, but its dedicated parity disk creates a significant scalability and performance bottleneck.

Because of these limitations and the development of more flexible RAID architectures, RAID 3 is considered a legacy RAID level.

The key concepts are:

- Byte-level striping
- Dedicated parity
- XOR
- Single-disk fault tolerance
- Degraded operation
- Reconstruction
- Rebuild
- Sequential performance
- Dedicated parity bottleneck
- Legacy architecture
