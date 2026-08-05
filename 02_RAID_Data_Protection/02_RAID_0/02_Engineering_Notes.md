# Engineering Notes – RAID 0

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

Topic: RAID 0

---

# 1. Engineering Perspective

RAID 0 is designed to maximize storage performance by distributing I/O operations across multiple physical disks using block-level striping.

Unlike redundant RAID levels, RAID 0 does not perform mirroring or parity calculations. All available storage capacity is used for application data.

---

# 2. RAID 0 Implementation

RAID 0 can be implemented using:

- Hardware RAID Controllers
- Software RAID

Linux Software RAID uses the **mdadm** utility to create RAID devices such as:

```
/dev/md0
/dev/md1
```

Applications interact with these logical devices instead of individual disks.

---

# 3. Linux Software RAID

During practical implementation, RAID 0 was created using Linux Software RAID.

Example:

```bash
sudo mdadm --create /dev/md1 \
--level=0 \
--raid-devices=2 \
/dev/sdb /dev/sde
```

After creation, Linux presented the array as:

```
/dev/md1
```

---

# 4. RAID Metadata

Linux stores RAID configuration information on every member disk.

The metadata contains information such as:

- RAID UUID
- RAID Level
- Member Disk Role
- Chunk Size
- RAID State
- Event Counter
- Creation Time

This metadata allows Linux to automatically identify and assemble RAID arrays after a system reboot.

---

# 5. Metadata Version

During RAID creation the following message was observed:

```
mdadm: Defaulting to version 1.2 metadata
```

Metadata Version 1.2 stores the RAID superblock near the beginning of each member disk.

---

# 6. RAID Superblock

Every RAID member contains a RAID superblock.

The superblock stores configuration information required to identify and assemble the RAID array.

Example information:

- RAID UUID
- Device UUID
- RAID Level
- Number of RAID Devices
- Chunk Size
- Array State

Linux reads this information during RAID assembly.

---

# 7. Chunk Size

The RAID 0 array used a chunk size of:

```
512 KB
```

Chunk Size determines how much data is written to one disk before the RAID controller switches to the next member disk.

Example:

```
512 KB → Disk 1

512 KB → Disk 2

512 KB → Disk 1

512 KB → Disk 2
```

Chunk size influences:

- Sequential Performance
- Random Performance
- Database Workloads
- Large File Transfers

---

# 8. Member Ordering

Each RAID member has a unique role within the RAID group.

Example:

```
Disk sdb

Role:
Active Device 0

----------------------

Disk sde

Role:
Active Device 1
```

Member ordering determines how striped blocks are distributed across the RAID group.

---

# 9. Array UUID vs Device UUID

Each RAID member stores two different identifiers.

## Array UUID

Identifies the RAID array.

All member disks belonging to the same RAID array share the same Array UUID.

---

## Device UUID

Identifies an individual physical disk.

Every member disk has its own unique Device UUID.

---

# 10. RAID Validation Commands

The following commands were used during RAID validation.

View block devices:

```bash
lsblk
```

Display RAID status:

```bash
cat /proc/mdstat
```

Display RAID configuration:

```bash
sudo mdadm --detail /dev/md1
```

Display RAID metadata:

```bash
sudo mdadm --examine /dev/sdb
```

```bash
sudo mdadm --examine /dev/sde
```

Display disk I/O statistics:

```bash
iostat -dx 1
```

---

# 11. Performance Validation

A large file was written to the RAID array.

During the write operation:

- Both disks received write requests simultaneously.
- I/O was distributed across both member disks.
- Aggregate throughput was higher than a single disk.

This confirmed that striping was functioning correctly.

---

# 12. Engineering Limitations

Storage engineers should consider the following limitations before selecting RAID 0.

- No redundancy
- No parity
- No mirroring
- No rebuild capability
- Failure of one member causes failure of the entire RAID array

RAID 0 should only be selected when maximum performance is the primary objective.

---

# 13. Best Practices

- Use RAID 0 only for non-critical workloads.
- Never store irreplaceable business data on RAID 0 without backups.
- Validate RAID configuration after creation.
- Verify chunk size and RAID metadata.
- Monitor disk health continuously.

---

# Key Engineering Takeaways

- RAID 0 is a performance-oriented RAID implementation.
- Linux Software RAID uses `mdadm` to create and manage RAID arrays.
- RAID metadata is stored on every member disk.
- Chunk size directly influences I/O behavior.
- Validation should include `lsblk`, `/proc/mdstat`, `mdadm --detail`, `mdadm --examine`, and performance verification.
