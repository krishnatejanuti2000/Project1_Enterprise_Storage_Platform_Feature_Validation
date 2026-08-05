# RAID Fundamentals

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

---

# 1. Introduction

Enterprise applications such as databases, virtual machines, file servers, cloud platforms, and backup systems depend heavily on storage for storing and retrieving data.

Using a single storage device introduces several limitations. A single disk can become a performance bottleneck, provides limited storage capacity, and represents a single point of failure. If that disk fails, applications may become unavailable and critical business data may be lost.

To overcome these limitations, enterprise storage systems combine multiple physical disks into a single logical storage unit using **RAID (Redundant Array of Independent Disks)**.

RAID is designed to improve one or more of the following objectives:

- Performance
- Data Protection
- Storage Capacity

Different RAID levels achieve these objectives using different techniques such as striping, mirroring, and parity.

---

# 2. What is RAID?

**RAID (Redundant Array of Independent Disks)** is a storage technology that combines multiple physical storage devices into a single logical storage device.

Instead of the operating system communicating directly with each physical disk, it interacts with a single logical RAID device.

```
Without RAID

Application
      │
      ▼
 Physical Disk


With RAID

Application
      │
      ▼
 Logical RAID Device
      │
 ┌────┴────┐
 ▼         ▼
Disk1    Disk2
```

The RAID controller determines how data is distributed, protected, and retrieved from the member disks.

---

# 3. Types of RAID

RAID can be implemented in two different ways:

- Hardware RAID
- Software RAID

Both provide RAID functionality but differ in how RAID operations are managed.

---

## 3.1 Hardware RAID

Hardware RAID uses a dedicated RAID controller installed in the server or storage system.

The RAID controller is responsible for:

- Creating RAID groups
- Managing data distribution
- Detecting disk failures
- Rebuilding failed disks
- Presenting logical disks to the operating system

Since RAID processing is handled by dedicated hardware, the operating system only sees the logical RAID device.

### Advantages

- High performance
- Dedicated RAID processor
- Lower CPU utilization
- Advanced enterprise features

### Common Hardware RAID Controllers

- Dell PERC
- HPE Smart Array
- Broadcom / LSI MegaRAID
- Lenovo ThinkSystem RAID

---

## 3.2 Software RAID

Software RAID is implemented by the operating system without requiring a dedicated RAID controller.

In Linux, Software RAID is managed using the **mdadm** utility.

The operating system performs RAID operations using CPU resources.

### Advantages

- No additional RAID hardware required
- Cost-effective
- Flexible configuration
- Common in Linux servers and cloud environments

### Example

Linux Software RAID creates logical devices such as:

```
/dev/md0
/dev/md1
/dev/md2
```

Applications use these logical devices exactly like normal storage devices.

---

## 3.3 Hardware RAID vs Software RAID

| Feature | Hardware RAID | Software RAID |
|----------|---------------|---------------|
| RAID Management | Dedicated RAID Controller | Operating System |
| Additional Hardware | Required | Not Required |
| CPU Usage | Very Low | Uses System CPU |
| Cost | Higher | Lower |
| Enterprise Usage | Enterprise Storage Arrays | Linux Servers, Cloud Platforms |

---

# 4. Why RAID Was Invented

Using a single disk introduces several limitations.

## Problem 1 – Single Point of Failure

If the disk fails:

- Data stored on that disk becomes unavailable.
- Applications depending on that data may stop functioning.
- Business operations may be interrupted.

---

## Problem 2 – Limited Performance

A single disk can perform only a limited number of read and write operations.

As workloads increase, one disk may not be able to satisfy all I/O requests efficiently.

---

## Problem 3 – Limited Capacity

A single disk provides only its own storage capacity.

Enterprise environments often require significantly larger storage pools.

---

To overcome these limitations, RAID combines multiple disks and distributes data intelligently among them.

---

# 5. Enterprise Storage Requirements

Enterprise storage systems are generally designed to satisfy three major objectives.

## Performance

Applications should read and write data with minimum latency and maximum throughput.

Examples:

- Databases
- Virtual Machines
- Video Processing
- Analytics

---

## Data Protection

Business-critical information should remain available even if one or more disks fail.

Examples:

- Banking
- Healthcare
- Cloud Infrastructure
- Enterprise Backup

---

## Capacity

Organizations require large storage pools to support continuously growing business data.

---

# 6. The Three Objectives of RAID

RAID improves one or more of the following objectives.

| Objective | Description |
|-----------|-------------|
| Performance | Increase read/write speed |
| Data Protection | Protect against disk failures |
| Capacity | Combine multiple disks into larger logical storage |

Different RAID levels prioritize these objectives differently.

---

# 7. Why One RAID Level Cannot Satisfy Every Requirement

Enterprise applications have different storage requirements.

### Customer A

Requirement:

- Maximum performance
- Temporary or non-critical data

Suitable RAID:

- RAID 0

---

### Customer B

Requirement:

- Business-critical data
- High availability
- Disk failure protection

Suitable RAID:

- RAID 1

---

### Customer C

Requirement:

- Balanced performance
- Large usable capacity
- Fault tolerance

Suitable RAID:

- RAID 5 or RAID 6

---

No single RAID level can simultaneously provide:

- Maximum performance
- Maximum usable capacity
- Maximum fault tolerance

Every RAID level is designed as a trade-off between these objectives.

---

# 8. RAID Components and Terminology

## Physical Disk

An actual HDD or SSD installed in the storage system.

---

## RAID Group

A collection of physical disks configured together using a specific RAID level.

---

## Logical Disk

The virtual storage device presented to the operating system after the RAID array is created.

Applications communicate with the logical disk rather than the individual physical disks.

---

## RAID Controller

The component responsible for managing the RAID array.

It determines:

- Data placement
- Read operations
- Write operations
- Failure handling
- Rebuild operations

RAID controllers may be implemented using dedicated hardware or software such as Linux `mdadm`.

---

# 9. Common RAID Techniques

Different RAID levels use different techniques.

| Technique | Purpose |
|-----------|---------|
| Striping | Improves performance |
| Mirroring | Provides redundancy |
| Parity | Balances capacity and fault tolerance |

Each RAID level uses one or more of these techniques.

---

# 10. RAID Levels Covered in This Module

The following RAID levels will be studied in this module:

- RAID 0
- RAID 1
- RAID 2
- RAID 3
- RAID 4
- RAID 5
- RAID 6
- RAID 10
- RAID 50
- RAID 60

Each RAID level will be covered from the following perspectives:

- Architecture
- Internal Working
- Read Flow
- Write Flow
- Capacity Calculation
- Performance Analysis
- Fault Tolerance
- Enterprise Use Cases
- Advantages
- Disadvantages
- Practical Implementation

---

# 11. Key Takeaways

- RAID combines multiple physical disks into one logical storage device.
- RAID improves performance, data protection, capacity, or a combination of these.
- RAID can be implemented using either Hardware RAID or Software RAID.
- Different enterprise workloads require different RAID levels.
- No single RAID level is suitable for every workload.
- RAID uses striping, mirroring, and parity as its fundamental techniques.

---

# Summary

RAID is a foundational technology in enterprise storage systems that combines multiple physical disks into a single logical storage device. Depending on business requirements, RAID can improve performance, provide data protection, increase storage capacity, or balance these objectives. Understanding RAID fundamentals provides the foundation for studying each RAID level and its role in enterprise storage platforms.
