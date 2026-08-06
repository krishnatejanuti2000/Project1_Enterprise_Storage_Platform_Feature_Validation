# Interview Guide – RAID 1

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 1**

---

# Basic Interview Questions

## 1. What is RAID 1?

RAID 1 is a fault-tolerant RAID level that stores identical copies of data on multiple disks using mirroring.

---

## 2. What is the primary objective of RAID 1?

The primary objective is to provide data protection and high availability by maintaining duplicate copies of all data.

---

## 3. What is mirroring?

Mirroring is the process of writing identical data to every member disk in the RAID group.

---

## 4. What is the minimum number of disks required for RAID 1?

A minimum of **2 disks**.

---

## 5. Does RAID 1 provide redundancy?

Yes.

Every block of data is stored on all mirror members.

---

# Capacity Questions

## 6. What is the storage efficiency of RAID 1?

50%.

Half of the raw capacity is used for mirrored copies.

---

## 7. How is RAID 1 capacity calculated?

Formula:

```
Usable Capacity = Smallest Member Disk Capacity
```

Example:

```
500 GB + 500 GB

=

500 GB usable
```

---

## 8. What happens if disks of different sizes are used?

The smallest disk determines the usable capacity.

Example:

```
5 TB + 1 TB

↓

Usable Capacity = 1 TB
```

---

# Performance Questions

## 9. Does RAID 1 improve read performance?

Yes.

The RAID controller can distribute read requests across multiple mirror members.

---

## 10. Why is RAID 1 read performance better than a single disk?

Because multiple disks can service different read requests simultaneously, reducing read latency and balancing the workload.

---

## 11. Does RAID 1 improve write performance?

No.

Every write must be completed on all mirror members before the RAID controller acknowledges completion.

---

## 12. Which RAID provides better write performance: RAID 0 or RAID 1?

RAID 0.

RAID 0 performs parallel writes without mirroring, whereas RAID 1 must write identical data to all mirror members.

---

# Failure Questions

## 13. What happens if one disk fails in RAID 1?

The RAID enters **degraded mode**, but applications continue accessing data using the surviving mirror.

---

## 14. What is degraded mode?

A state in which one or more mirror members have failed, but the RAID array remains operational because at least one valid copy of the data still exists.

---

## 15. What does `[UU]` mean?

Both RAID members are healthy and synchronized.

---

## 16. What does `[U_]` mean?

One member disk is healthy and the other member is missing or has failed.

---

## 17. Can applications continue working when RAID 1 is in degraded mode?

Yes.

The RAID controller redirects all I/O requests to the surviving mirror member.

---

## 18. What happens if both disks fail?

The RAID array becomes unavailable because no valid mirror copy remains.

---

# Rebuild Questions

## 19. What is a RAID rebuild?

A rebuild is the process of copying data from the healthy mirror member to a replacement disk after a disk failure.

---

## 20. When does a rebuild start?

After the failed disk has been replaced and added back to the RAID array.

---

## 21. What happens during a rebuild?

The RAID controller copies every block from the healthy disk to the replacement disk until both mirrors are identical.

---

## 22. Can applications access data during a rebuild?

Yes.

The RAID remains accessible, although performance may be reduced because rebuild I/O shares disk resources with normal application I/O.

---

# Engineering Questions

## 23. Which Linux utility is used to manage RAID 1?

`mdadm`

---

## 24. Which command displays RAID status?

```bash
cat /proc/mdstat
```

---

## 25. Which command displays RAID details?

```bash
sudo mdadm --detail /dev/md1
```

---

## 26. Which command displays RAID metadata?

```bash
sudo mdadm --examine /dev/sdb
```

---

## 27. What is a write-intent bitmap?

A write-intent bitmap tracks modified regions of the RAID array, allowing faster recovery and resynchronization after an interruption.

---

## 28. Why should you wait for the initial resync to complete before testing failures?

Because the RAID controller must first synchronize all mirror members to establish a consistent fault-tolerant state.

---

# Scenario-Based Questions

## 29. A customer is deploying a banking database. Which RAID would you recommend?

RAID 1, because data protection and availability are more important than maximizing storage capacity.

---

## 30. A customer needs maximum write performance and already has external backups. Which RAID would you recommend?

RAID 0, because the primary requirement is performance rather than redundancy.

---

## 31. A customer wants both high performance and fault tolerance. Is RAID 1 always the best choice?

Not necessarily.

For workloads requiring both high performance and redundancy, RAID 10 may be a better solution.

---

# Practical Questions

## 32. Which steps are involved in validating a RAID 1 array?

- Verify available disks
- Create RAID 1
- Monitor initial resync
- Create filesystem
- Mount filesystem
- Generate test data
- Simulate disk failure
- Verify degraded mode
- Confirm data accessibility
- Replace failed disk
- Monitor rebuild
- Verify healthy state
- Perform cleanup

---

## 33. How do you verify that RAID 1 is healthy?

Check:

```bash
cat /proc/mdstat
```

Expected:

```text
[UU]
```

Also verify:

```bash
sudo mdadm --detail /dev/md1
```

---

## 34. How do you verify that the array is degraded?

Expected:

```text
[U_]
```

and

```text
State : clean, degraded
```

---

# Quick Revision

- RAID 1 = Mirroring
- Minimum 2 disks
- 50% storage efficiency
- Better read performance than a single disk
- Writes must complete on all mirror members
- One disk failure = Array continues in degraded mode
- `[UU]` = Healthy
- `[U_]` = Degraded
- Supports rebuild
- Managed using `mdadm`
- Ideal for business-critical workloads
