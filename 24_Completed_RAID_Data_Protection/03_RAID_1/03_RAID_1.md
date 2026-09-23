# RAID 1 (Mirroring)

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

---

# 1. Introduction

Modern enterprise applications require continuous access to business-critical data. Systems such as banking platforms, healthcare applications, ERP systems, virtualization hosts, and database servers cannot afford data loss due to a single disk failure.

While RAID 0 improves storage performance through striping, it provides no fault tolerance. The failure of a single member disk results in the failure of the entire array.

To address this limitation, RAID 1 was introduced. RAID 1 provides **data redundancy through mirroring**, ensuring that data remains available even if one member disk fails.

Unlike RAID 0, RAID 1 prioritizes **data protection and availability** over maximum storage capacity.

---

# 2. Why RAID 1 Was Invented

The primary objective of RAID 1 is to eliminate the single point of failure associated with individual disks.

Consider a system using only one disk.

```
Application
      │
      ▼
 Single Disk
```

If that disk fails:

- Applications lose access to data.
- Business operations stop.
- Data recovery depends entirely on backups.

RAID 1 solves this problem by maintaining an identical copy of all data on multiple disks.

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

Both disks always contain the same information.

---

# 3. What is RAID 1?

RAID 1 is a RAID level that provides fault tolerance by writing identical copies of data to multiple disks.

This technique is called **mirroring**.

The operating system sees only one logical RAID device, while the RAID controller manages synchronization between the member disks.

---

# 4. RAID 1 Architecture

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

Every write operation is sent to both disks.

---

# 5. Core Technique – Mirroring

RAID 1 stores identical copies of data on every member disk.

Example:

```
Application Data

A B C D
```

After mirroring:

```
Disk A

A
B
C
D

------------------

Disk B

A
B
C
D
```

Each disk contains a complete copy of the data.

---

# 6. RAID Controller Responsibilities

The RAID controller is responsible for:

- Creating the RAID array.
- Writing identical data to every member disk.
- Presenting one logical RAID device to the operating system.
- Managing RAID metadata.
- Monitoring disk health.
- Detecting failed disks.
- Initiating rebuild operations after disk replacement.

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
500 GB
```

One disk stores the primary data, while the second stores an identical copy.

---

## Formula

```
Usable Capacity

=

Smallest Member Disk Capacity
```

Storage Efficiency

```
50%
```

If disks have different capacities:

```
Disk A = 5 TB

Disk B = 1 TB
```

Usable Capacity:

```
1 TB
```

The remaining space cannot be mirrored and therefore cannot be used.

---

# 8. Read Flow

When an application issues a read request:

```
Application

↓

RAID Controller

↓

Disk A or Disk B
```

The RAID controller may choose the disk that is:

- Less busy
- Closer to the requested data
- Better positioned to serve the request

This improves read performance through load balancing.

---

# 9. Write Flow

Suppose the application writes:

```
Write 1 GB
```

The RAID controller sends the same data to both disks.

```
Disk A

1 GB

--------------

Disk B

1 GB
```

The write operation is acknowledged to the operating system **only after both disks successfully complete the write**.

---

# 10. Read Performance

RAID 1 generally provides better read performance than a single disk because read requests can be distributed across multiple mirror members.

Benefits include:

- Parallel read capability
- Reduced read latency
- Load balancing
- Improved read throughput

---

# 11. Write Performance

Write performance is typically lower than RAID 0 because every write must be completed on all mirror members before acknowledgement.

Benefits:

- Data protection
- High availability

Trade-off:

- Increased write latency compared to RAID 0

---

# 12. Healthy State

A healthy RAID 1 array means:

- All member disks are operational.
- All mirror copies are synchronized.

Example:

```
[UU]
```

Both disks are healthy.

---

# 13. Degraded State

If one mirror member fails:

```
[U_]
```

The RAID array continues operating using the surviving disk.

Characteristics:

- Data remains accessible.
- Fault tolerance is temporarily reduced.
- Performance may be affected.
- Immediate disk replacement is recommended.

---

# 14. Disk Failure

Example:

```
Disk A ❌ Failed

Disk B ✅ Healthy
```

The RAID controller automatically redirects all read and write requests to Disk B.

Applications continue operating without being aware that a disk has failed.

If both disks fail:

- The RAID array becomes unavailable.
- Data cannot be accessed.

---

# 15. Rebuild Process

After replacing the failed disk:

```
Healthy

↓

Disk Failure

↓

Degraded Mode

↓

Replace Failed Disk

↓

Rebuild

↓

Healthy
```

The RAID controller copies every block from the healthy disk to the replacement disk.

Once synchronization completes, the array returns to:

```
[UU]
```

---

# 16. Advantages

- Excellent data protection
- High availability
- Simple recovery process
- Faster read performance than a single disk
- Automatic failover after disk failure
- Easy rebuild process

---

# 17. Disadvantages

- 50% storage efficiency
- Higher storage cost
- Slower writes than RAID 0
- Longer rebuild time for large disks

---

# 18. Enterprise Use Cases

RAID 1 is commonly used for:

- Banking databases
- Financial systems
- Healthcare applications
- Operating system volumes
- Critical application servers
- Virtualization hosts
- Small business file servers

---

# 19. Customer Scenario

### Customer Requirement

A bank requires continuous access to its transaction database. Data protection is the highest priority.

### Recommendation

RAID 1

### Reason

RAID 1 provides redundancy through mirroring. If one disk fails, the surviving mirror continues serving application requests, ensuring business continuity while the failed disk is replaced and rebuilt.

---

# 20. Practical Validation

During this module, RAID 1 validation includes:

- RAID creation
- RAID verification
- Initial synchronization (Resync)
- Filesystem creation
- Mount validation
- Test data generation
- Disk failure simulation
- Degraded mode validation
- Data accessibility verification
- Disk replacement
- RAID rebuild
- Cleanup

All practical evidence is documented under the **Lab_Evidence** directory.

---

# 21. Summary

RAID 1 is a fault-tolerant RAID level that protects data by maintaining identical copies on multiple disks. Although it sacrifices 50% of the available storage capacity and has slower write performance than RAID 0, it provides high availability and protects against single-disk failure. RAID 1 is widely used in enterprise environments where business continuity and data integrity are more important than maximum storage efficiency.
