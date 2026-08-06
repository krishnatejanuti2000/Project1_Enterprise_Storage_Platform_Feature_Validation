# Interview Guide – RAID 0

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 0**

---

# Basic Interview Questions

## 1. What is RAID 0?

RAID 0 is a RAID level that improves storage performance by distributing data across multiple disks using block-level striping. It provides maximum performance and 100% storage utilization but offers no fault tolerance.

---

## 2. What is the primary objective of RAID 0?

The primary objective of RAID 0 is to improve storage performance by allowing multiple disks to perform read and write operations simultaneously.

---

## 3. What is striping?

Striping is the process of dividing data into blocks and distributing those blocks across multiple disks. This enables parallel I/O operations.

---

## 4. What is the minimum number of disks required for RAID 0?

A minimum of **2 disks** is required.

---

## 5. Does RAID 0 provide redundancy?

No.

RAID 0 provides no mirroring, no parity, and no redundancy.

---

## 6. What is the storage efficiency of RAID 0?

100%.

All available disk capacity is usable.

---

## 7. How is RAID 0 capacity calculated?

Formula:

```
Usable Capacity = Sum of all member disks
```

Example:

```
500 GB + 500 GB = 1000 GB usable
```

---

# Performance Questions

## 8. Why is RAID 0 faster than a single disk?

Because data is striped across multiple disks, allowing parallel read and write operations.

---

## 9. Which operations benefit from RAID 0?

- Sequential Reads
- Sequential Writes
- Large File Transfers
- High Throughput Workloads

---

## 10. Does RAID 0 improve write performance?

Yes.

The RAID controller distributes write operations across multiple disks, increasing overall throughput.

---

## 11. Does RAID 0 improve read performance?

Yes.

Multiple disks can read different portions of the requested data simultaneously.

---

# Failure Questions

## 12. What happens if one disk fails in RAID 0?

The entire RAID array fails because part of every file is stored on the failed disk.

---

## 13. Can RAID 0 rebuild after a disk failure?

No.

RAID 0 has no redundant information, so missing data cannot be reconstructed.

---

## 14. Why can't RAID 0 rebuild?

Because it stores neither mirror copies nor parity information.

---

## 15. Can applications access data after a RAID 0 disk failure?

No.

The logical RAID device becomes unavailable because complete files can no longer be reconstructed.

---

# Engineering Questions

## 16. Which Linux utility is commonly used to manage Software RAID?

`mdadm`

---

## 17. Which command displays RAID status?

```bash
cat /proc/mdstat
```

---

## 18. Which command displays RAID details?

```bash
sudo mdadm --detail /dev/md1
```

---

## 19. Which command displays RAID metadata?

```bash
sudo mdadm --examine /dev/sdb
```

---

## 20. What is a RAID superblock?

A RAID superblock is metadata stored on each RAID member disk that contains RAID configuration information such as RAID level, UUID, chunk size, and member role.

---

## 21. What is chunk size?

Chunk size is the amount of data written to one disk before the RAID controller switches to the next disk.

---

## 22. What is the difference between Array UUID and Device UUID?

**Array UUID** identifies the RAID array.

**Device UUID** uniquely identifies an individual member disk.

---

# Scenario-Based Questions

## 23. A customer needs maximum performance for video editing. Which RAID would you recommend?

RAID 0, because performance is the highest priority and the source data is typically backed up elsewhere.

---

## 24. Would you recommend RAID 0 for a banking database?

No.

Banking applications require fault tolerance and data protection. RAID 0 provides neither.

---

## 25. A customer wants both maximum performance and data protection. Is RAID 0 suitable?

No.

RAID 0 provides excellent performance but no redundancy. A RAID level such as RAID 10 would be a better choice.

---

# Practical Questions

## 26. Which steps are involved in validating a RAID 0 array?

- Verify available disks
- Create RAID array
- Verify RAID status
- Create filesystem
- Mount filesystem
- Generate test data
- Observe disk I/O
- Examine RAID metadata
- Perform cleanup

---

## 27. How would you verify that striping is working?

Generate disk activity using a large file write and observe both member disks with:

```bash
iostat -dx 1
```

Both disks should show simultaneous I/O activity.

---

# Quick Revision

- RAID 0 = Striping
- Minimum 2 disks
- 100% storage efficiency
- Highest performance
- No redundancy
- No parity
- No rebuild
- One disk failure = Entire array failure
- Managed using `mdadm` in Linux
