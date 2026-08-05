# Interview Guide – RAID Fundamentals

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

Topic: RAID Fundamentals

---

# Basic Interview Questions

## Q1. What is RAID?

**Answer:**

RAID (Redundant Array of Independent Disks) is a storage technology that combines multiple physical disks into a single logical storage device to improve performance, provide data protection, increase storage capacity, or a combination of these objectives.

---

## Q2. Why was RAID invented?

**Answer:**

RAID was invented to overcome the limitations of a single disk.

A single disk suffers from:

- Limited performance
- Limited storage capacity
- Single point of failure

RAID addresses these limitations by combining multiple disks into a RAID group.

---

## Q3. What are the three primary objectives of RAID?

**Answer:**

The three primary objectives of RAID are:

- Performance
- Data Protection
- Capacity

Different RAID levels prioritize these objectives differently.

---

## Q4. Can one RAID level satisfy every customer requirement?

**Answer:**

No.

Different enterprise applications have different storage requirements.

For example:

- RAID 0 prioritizes performance.
- RAID 1 prioritizes redundancy.
- RAID 5 and RAID 6 balance capacity and fault tolerance.

Therefore, multiple RAID levels exist to satisfy different business requirements.

---

## Q5. What is the difference between a Physical Disk and a Logical Disk?

**Answer:**

A Physical Disk is the actual HDD or SSD installed in the system.

A Logical Disk is the virtual storage device presented to the operating system after the RAID array is created.

Applications interact with the logical disk rather than the individual physical disks.

---

## Q6. What is a RAID Group?

**Answer:**

A RAID Group is a collection of physical disks configured together using a specific RAID level.

The RAID group determines how data is distributed and protected across the member disks.

---

## Q7. What is a RAID Controller?

**Answer:**

A RAID Controller is responsible for:

- Creating RAID groups
- Managing read and write operations
- Detecting disk failures
- Rebuilding failed disks
- Presenting logical disks to the operating system

RAID controllers can be implemented using dedicated hardware or software.

---

## Q8. What is the difference between Hardware RAID and Software RAID?

| Hardware RAID | Software RAID |
|---------------|---------------|
| Managed by a dedicated RAID controller | Managed by the operating system |
| Requires additional hardware | No dedicated hardware required |
| Lower CPU utilization | Uses system CPU |
| Common in enterprise storage arrays | Common in Linux servers and cloud platforms |

---

## Q9. What are the common RAID techniques?

**Answer:**

RAID implementations use three primary techniques:

- Striping
- Mirroring
- Parity

Different RAID levels use one or more of these techniques.

---

## Q10. Why is RAID important in enterprise environments?

**Answer:**

Enterprise applications require:

- High performance
- High availability
- Data protection
- Large storage capacity

RAID helps meet these requirements by combining multiple disks into a logical storage system.

---

# Scenario-Based Questions

## Q11. A customer requires maximum performance but does not require data protection. Which RAID level would you recommend?

**Answer:**

RAID 0.

Reason:

RAID 0 uses striping to maximize read and write performance but provides no fault tolerance.

---

## Q12. A customer requires protection against disk failure and is willing to sacrifice usable capacity. Which RAID level would you recommend?

**Answer:**

RAID 1.

Reason:

RAID 1 uses mirroring to provide redundancy and can tolerate one disk failure.

---

## Q13. Why doesn't every enterprise use RAID 0?

**Answer:**

Although RAID 0 provides excellent performance, it offers no redundancy.

Failure of a single member disk results in complete loss of the RAID array.

Therefore, RAID 0 is unsuitable for business-critical data.

---

# Quick Revision

- RAID = Redundant Array of Independent Disks
- Combines multiple physical disks into one logical disk
- Objectives:
  - Performance
  - Data Protection
  - Capacity
- Hardware RAID uses a dedicated RAID controller.
- Software RAID uses the operating system (Linux `mdadm`).
- Different workloads require different RAID levels.
- RAID uses Striping, Mirroring, and Parity as its core techniques.
