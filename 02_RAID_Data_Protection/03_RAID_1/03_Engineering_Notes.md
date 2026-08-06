# Engineering Notes – RAID 1

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 1**

---

# 1. Engineering Perspective

RAID 1 is a fault-tolerant RAID implementation that protects data by maintaining identical copies (mirrors) on multiple physical disks.

Unlike RAID 0, RAID 1 prioritizes data protection and high availability over maximum storage capacity and write performance.

---

# 2. RAID 1 Implementation

RAID 1 can be implemented using:

- Hardware RAID Controllers
- Linux Software RAID (`mdadm`)

During this project, RAID 1 was implemented using Linux Software RAID.

The logical RAID device created by Linux appears as:

```text
/dev/md1
```

Applications interact with the logical RAID device rather than the physical disks.

---

# 3. RAID Creation

Example command used during validation:

```bash
sudo mdadm --create /dev/md1 \
    --level=1 \
    --raid-devices=2 \
    /dev/sdb /dev/sde
```

Important parameters:

| Parameter | Purpose |
|-----------|---------|
| `--create` | Creates a new RAID array |
| `--level=1` | Specifies RAID 1 (Mirroring) |
| `--raid-devices=2` | Number of mirror members |
| `/dev/sdb /dev/sde` | Member disks |

---

# 4. Initial Synchronization (Resync)

Immediately after RAID creation, Linux begins an initial synchronization process.

Example:

```text
resync = 19%
finish = 20 min
```

Purpose:

- Ensure all mirror members contain identical data.
- Establish a consistent RAID state.
- Prepare the array for fault-tolerant operation.

The RAID should not be tested for failure scenarios until the initial resync is complete.

---

# 5. Write-Intent Bitmap

During RAID creation, `mdadm` may prompt:

```text
Enable write-intent bitmap? [y/N]
```

A write-intent bitmap records which regions of the RAID array have been modified.

Benefits:

- Faster recovery after unexpected shutdowns.
- Reduced rebuild time.
- Less unnecessary disk I/O.
- Improved resynchronization efficiency.

During this project, an **internal bitmap** was enabled.

---

# 6. RAID Metadata

Linux stores RAID metadata (superblock) on every RAID member.

The metadata contains:

- RAID UUID
- Device UUID
- RAID Level
- Member Role
- Array State
- Event Counter
- Creation Time
- Bitmap Information

This metadata enables Linux to identify and assemble RAID arrays automatically.

---

# 7. Metadata Version

During RAID creation:

```text
mdadm: Defaulting to version 1.2 metadata
```

Version 1.2 stores the RAID superblock near the beginning of each member disk.

---

# 8. RAID Superblock

Each RAID member stores a superblock containing:

- RAID UUID
- Device UUID
- RAID Level
- RAID Device Count
- Member Role
- Bitmap Status
- Event Counter
- Array State

Linux reads this information during RAID assembly.

---

# 9. Member Roles

Example:

```text
Disk sdb

Role:
Active Device 0
```

```text
Disk sde

Role:
Active Device 1
```

Each member has a unique role while storing identical data.

---

# 10. RAID Health Indicators

Healthy array:

```text
[UU]
```

Meaning:

- Disk 1 = Up
- Disk 2 = Up

---

Degraded array:

```text
[U_]
```

Meaning:

- Disk 1 = Healthy
- Disk 2 = Missing or Failed

These indicators are visible in:

```bash
cat /proc/mdstat
```

---

# 11. RAID Validation Commands

Display block devices:

```bash
lsblk
```

Display RAID status:

```bash
cat /proc/mdstat
```

Display RAID details:

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

Display mounted filesystems:

```bash
df -h
```

Display disk I/O statistics:

```bash
iostat -dx 1
```

---

# 12. Validation Workflow

The RAID 1 validation followed these steps:

1. Verify available disks.
2. Remove stale RAID metadata.
3. Create RAID 1.
4. Verify RAID status.
5. Wait for initial resynchronization.
6. Create filesystem.
7. Mount filesystem.
8. Generate test data.
9. Simulate disk failure.
10. Verify degraded mode.
11. Confirm data accessibility.
12. Replace failed disk.
13. Monitor rebuild.
14. Verify healthy state.
15. Clean up the environment.

---

# 13. Cleanup Procedure

Unmount the filesystem:

```bash
sudo umount /mnt/raid1
```

Stop the RAID array:

```bash
sudo mdadm --stop /dev/md1
```

Remove RAID metadata:

```bash
sudo mdadm --zero-superblock /dev/sdb

sudo mdadm --zero-superblock /dev/sde
```

Verify clean disks:

```bash
lsblk
```

---

# 14. Engineering Best Practices

- Verify disk state before RAID creation.
- Remove stale RAID metadata before reusing disks.
- Wait for initial resynchronization to complete before testing failures.
- Monitor RAID health using `cat /proc/mdstat`.
- Confirm RAID configuration using `mdadm --detail`.
- Verify RAID metadata using `mdadm --examine`.
- Replace failed disks immediately to restore redundancy.
- Monitor rebuild progress until the array returns to `[UU]`.

---

# Engineering Takeaways

- RAID 1 provides redundancy through mirroring.
- Linux Software RAID uses `mdadm` to create and manage mirror arrays.
- Initial resynchronization is required after array creation.
- Write-intent bitmaps reduce rebuild time after interruptions.
- RAID health can be monitored using `[UU]` and `[U_]`.
- Proper validation includes creation, synchronization, failure simulation, rebuild verification, and cleanup.
