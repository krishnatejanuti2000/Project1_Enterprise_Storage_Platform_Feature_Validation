# RAID 0 (Striping)

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

---

# 1. Introduction

Enterprise applications continuously generate and process large amounts of data. Modern workloads such as databases, virtualization, video editing, scientific computing, and analytics demand higher storage throughput than a single disk can provide.

A single HDD or SSD has physical limitations in terms of bandwidth, IOPS, and latency. As workloads increase, a single disk often becomes a performance bottleneck.

To overcome this limitation, RAID 0 was introduced to distribute I/O operations across multiple disks using **block-level striping**, allowing several disks to work simultaneously and significantly improving storage performance.

Unlike other RAID levels, RAID 0 focuses entirely on performance and capacity. It does **not** provide data redundancy or fault tolerance.

---

# 2. Why RAID 0 Was Invented

The primary goal behind RAID 0 was to improve storage performance.

Consider a system with only one disk.

```
Application
      │
      ▼
 Single Disk
```

Every read and write request must be handled by that one disk.

As the workload increases:

- The disk becomes a bottleneck.
- Applications wait longer for I/O completion.
- Throughput is limited by a single device.

Engineers solved this problem by allowing multiple disks to participate in every I/O request.

Instead of using one disk:

```
Application
      │
      ▼
 RAID Controller
      │
 ┌────┴────┐
 ▼         ▼
Disk A   Disk B
```

The workload is divided across multiple disks, enabling parallel I/O.

---

# 3. What is RAID 0?

RAID 0 is a RAID level that combines multiple physical disks into a single logical storage device using **block-level striping**.

Instead of writing all data to one disk, the RAID controller divides the data into blocks (called stripes) and distributes those blocks across all participating disks.

The operating system sees only one logical RAID device.

---

# 4. RAID 0 Architecture

```
                Application
                      │
                      ▼
              Operating System
                      │
                      ▼
             RAID Controller
                │         │
        ┌───────┘         └───────┐
        ▼                         ▼
     Disk A                    Disk B
```

Applications communicate with the logical RAID device.

The RAID controller decides where each block will be written.

---

# 5. Core Technique – Striping

RAID 0 uses **block-level striping**.

Example:

```
Application Data

A B C D E F G H
```

After striping:

```
Disk A

A
C
E
G

------------------

Disk B

B
D
F
H
```

Each disk stores different portions of the data.

No duplicate copies are maintained.

---

# 6. RAID Controller Responsibilities

The RAID controller is responsible for:

- Creating the RAID array.
- Dividing data into stripes.
- Distributing stripes across member disks.
- Presenting one logical RAID device to the operating system.
- Managing RAID metadata.
- Handling read and write requests.

Unlike RAID 1, the RAID controller does **not** maintain mirror copies.

---

# 7. Capacity Calculation

Example:

```
Disk A = 500 GB

Disk B = 500 GB
```

Raw Capacity

```
500 + 500

=

1000 GB
```

Usable Capacity

```
1000 GB
```

Since RAID 0 stores only user data and no redundancy information, all available storage capacity is usable.

---

## Formula

```
Usable Capacity

=

Sum of all member disks
```

Storage Efficiency:

```
100%
```

---

# 8. Read Flow

Suppose an application requests:

```
Read 1 GB
```

The RAID controller determines where each stripe resides.

```
Application

↓

RAID Controller

↓

Disk A → Block 1

Disk B → Block 2
```

Both disks read simultaneously.

The RAID controller reassembles the blocks and returns the complete data to the application.

This parallel read operation improves throughput.

---

# 9. Write Flow

Suppose the application writes:

```
Write 1 GB
```

The RAID controller divides the data into stripes.

Example:

```
Disk A

500 MB

--------------

Disk B

500 MB
```

Both disks perform the write operation simultaneously.

After all stripes are written successfully, the RAID controller acknowledges the write completion to the operating system.

---

# 10. Read Performance

RAID 0 provides excellent read performance because multiple disks participate in servicing read requests.

Advantages include:

- Parallel reads
- Higher throughput
- Better bandwidth
- Reduced read bottlenecks

As more disks participate, read performance generally improves.

---

# 11. Write Performance

RAID 0 also provides excellent write performance.

Because the RAID controller distributes data across multiple disks:

- All member disks perform writes simultaneously.
- Each disk writes only a portion of the data.
- Overall write throughput increases significantly.

No parity calculations or mirror copies are required.

---

# 12. Healthy State

A healthy RAID 0 array means:

- All member disks are operational.
- Every stripe is accessible.
- The logical RAID device is fully functional.

```
Disk A ✅

Disk B ✅
```

Applications can access all data normally.

---

# 13. Disk Failure Scenario

Suppose:

```
Disk A ❌ Failed

Disk B ✅ Healthy
```

What happens?

Although Disk B is still operational, only part of every file remains.

Example:

```
Disk A

A
C
E
G

-------------

Disk B

B
D
F
H
```

After Disk A fails:

```
?

B

?

D

?

F

?

H
```

Half of the stripes are missing.

The RAID controller cannot reconstruct the original data.

The RAID array fails.

Applications lose access to the logical device.

---

# 14. Why RAID 0 Cannot Rebuild

Unlike RAID 1 or RAID 5, RAID 0 stores:

- No mirror copies
- No parity information
- No redundant blocks

When a disk fails, the missing stripes cannot be reconstructed.

Therefore:

- No rebuild is possible.
- The failed disk must be replaced.
- The RAID array must be recreated.
- Data must be restored from backup.

---

# 15. Advantages

- Excellent read performance.
- Excellent write performance.
- 100% storage efficiency.
- Maximum utilization of available capacity.
- Simple RAID implementation.
- No parity overhead.
- No mirroring overhead.

---

# 16. Disadvantages

- No fault tolerance.
- No redundancy.
- Failure of one disk causes failure of the entire array.
- No rebuild capability.
- Not suitable for critical business data without backups.

---

# 17. Enterprise Use Cases

RAID 0 is suitable for workloads where performance is more important than data protection.

Examples include:

- Video editing scratch disks
- Temporary rendering storage
- Scientific computing
- High-performance computing (HPC)
- Temporary analytics datasets
- Test environments
- Benchmarking

---

# 18. Customer Scenario

### Customer Requirement

A media production company processes large video files and requires maximum storage throughput.

The source videos are backed up elsewhere.

### Recommendation

RAID 0

### Reason

The primary requirement is performance rather than redundancy. Since the data already exists in backup storage, RAID 0 provides the highest throughput while utilizing 100% of the available capacity.

---

# 19. Practical Validation

During this module, RAID 0 was validated using Linux Software RAID (`mdadm`).

Validation included:

- RAID creation
- RAID verification
- Filesystem creation
- Mount validation
- Test data generation
- Performance observation
- RAID metadata verification
- Cleanup

All practical evidence is documented under the **Lab_Evidence** directory.

---

# 20. Summary

RAID 0 is a performance-oriented RAID level that uses block-level striping to distribute data across multiple disks. By allowing multiple disks to participate in every read and write operation, RAID 0 delivers excellent throughput and fully utilizes available storage capacity. However, because it provides no redundancy, the failure of a single member disk causes the entire array to fail. RAID 0 is therefore best suited for high-performance workloads where data protection is provided through other mechanisms, such as backups.
