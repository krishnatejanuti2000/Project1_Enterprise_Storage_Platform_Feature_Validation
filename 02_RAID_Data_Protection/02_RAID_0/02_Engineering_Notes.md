# Engineering Notes – RAID 0

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 0**

---

# 1. Engineering Perspective

RAID 0 is a performance-oriented RAID implementation that improves storage throughput by distributing I/O operations across multiple physical disks using block-level striping.

Unlike RAID levels that provide redundancy, RAID 0 does not perform mirroring or parity calculations. Every member disk stores only user data, allowing 100% utilization of the available storage capacity.

---

# 2. RAID 0 Implementation

RAID 0 can be implemented using:

- Hardware RAID Controllers
- Linux Software RAID (`mdadm`)

During this project, RAID 0 was implemented using Linux Software RAID.

The logical RAID device created by Linux appears as:

```text
/dev/md0
/dev/md1
```

Applications access the logical RAID device rather than the individual physical disks.

---

# 3. RAID Creation

Example command used during validation:

```bash
sudo mdadm --create /dev/md1 \
    --level=0 \
    --raid-devices=2 \
    /dev/sdb /dev/sde
```

Important parameters:

| Parameter | Purpose |
|-----------|---------|
| `--create` | Creates a new RAID array |
| `--level=0` | Specifies RAID 0 (Striping) |
| `--raid-devices=2` | Number of participating disks |
| `/dev/sdb /dev/sde` | Member disks |

---

# 4. RAID Metadata

Linux stores RAID metadata (superblock) on every RAID member disk.

The metadata contains configuration information required to identify and assemble the RAID array.

Typical information includes:

- RAID UUID
- Device UUID
- RAID Level
- RAID Device Count
- Chunk Size
- Array State
- Device Role
- Event Counter
- Creation Time

This metadata enables Linux to automatically assemble RAID arrays after reboot.

---

# 5. Metadata Version

During RAID creation, Linux reported:

```text
mdadm: Defaulting to version 1.2 metadata
```

Metadata version 1.2 stores the RAID superblock near the beginning of each member disk.

---

# 6. RAID Superblock

Every RAID member contains a RAID superblock.

The superblock stores:

- RAID UUID
- Device UUID
- RAID Level
- Number of RAID Devices
- Chunk Size
- Device Role
- Array State
- Event Counter

The Linux kernel reads the superblock during RAID assembly.

---

# 7. Chunk Size

The RAID 0 array was configured with:

```text
Chunk Size : 512 KB
```

Chunk size determines how much data is written to one disk before the RAID controller switches to the next member.

Example:

```text
512 KB → Disk A

512 KB → Disk B

512 KB → Disk A

512 KB → Disk B
```

Chunk size influences:

- Sequential I/O performance
- Random I/O performance
- Database workloads
- Large file transfers

---

# 8. Member Ordering

Each member disk has a unique RAID role.

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

The RAID controller uses these roles to determine stripe placement.

---

# 9. Array UUID vs Device UUID

## Array UUID

Identifies the RAID array.

All member disks belonging to the same RAID array share the same Array UUID.

---

## Device UUID

Identifies an individual physical disk.

Every RAID member has a unique Device UUID.

---

# 10. RAID Validation Commands

The following commands were used during validation.

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

Display disk I/O statistics:

```bash
iostat -dx 1
```

Display mounted filesystems:

```bash
df -h
```

---

# 11. Performance Validation

A large test file was written to the RAID array using:

```bash
sudo dd if=/dev/zero of=/mnt/raid0/testfile.img bs=1M count=2048 status=progress
```

Validation confirmed:

- RAID device accepted write requests successfully.
- Both member disks participated in striped I/O.
- Aggregate throughput exceeded that of a single disk.
- Filesystem remained consistent after write completion.

---

# 12. Cleanup Procedure

After validation:

- Unmount filesystem.
- Stop RAID device.
- Remove RAID metadata.
- Verify clean disks.

Example:

```bash
sudo umount /mnt/raid0

sudo mdadm --stop /dev/md1

sudo mdadm --zero-superblock /dev/sdb

sudo mdadm --zero-superblock /dev/sde
```

---

# 13. Engineering Best Practices

- Verify disk state before RAID creation.
- Remove stale RAID metadata before reusing disks.
- Validate RAID health using `cat /proc/mdstat`.
- Confirm RAID configuration using `mdadm --detail`.
- Examine member metadata using `mdadm --examine`.
- Select chunk size appropriate for the workload.
- Benchmark RAID performance after deployment.
- Maintain backups because RAID 0 provides no fault tolerance.

---

# Engineering Takeaways

- RAID 0 maximizes performance through block-level striping.
- Linux Software RAID uses `mdadm` for RAID management.
- Every member disk stores RAID metadata.
- Chunk size directly affects I/O behavior.
- Validation should include metadata verification, RAID health checks, filesystem validation, and performance observation.
- RAID 0 should only be deployed where performance is the primary objective and data protection is provided by external backups.
