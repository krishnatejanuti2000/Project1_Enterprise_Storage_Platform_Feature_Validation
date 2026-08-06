# Troubleshooting – RAID 0

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 0**

---

# 1. Introduction

This document covers common RAID 0 issues that may occur during creation, validation, testing, and cleanup using Linux Software RAID (`mdadm`).

---

# 2. RAID Device Already Exists

## Symptom

```bash
mdadm: device /dev/md1 already exists
```

## Cause

An existing RAID device is already active.

## Resolution

Check active RAID arrays:

```bash
cat /proc/mdstat
```

Stop the RAID:

```bash
sudo mdadm --stop /dev/md1
```

Verify:

```bash
lsblk
```

---

# 3. Inactive RAID Array (md127)

## Symptom

```text
md127 : inactive
```

## Cause

Linux automatically assembled RAID metadata from a previous configuration.

## Resolution

View RAID details:

```bash
sudo mdadm --detail /dev/md127
```

Stop the inactive array:

```bash
sudo mdadm --stop /dev/md127
```

Verify that it has disappeared:

```bash
lsblk
```

---

# 4. Existing RAID Metadata

## Symptom

```bash
mdadm: device or resource busy
```

or

```bash
mdadm: cannot create array
```

## Cause

The disks still contain RAID superblocks from an earlier RAID configuration.

## Resolution

Examine the disks:

```bash
sudo mdadm --examine /dev/sdb
sudo mdadm --examine /dev/sde
```

Remove the metadata:

```bash
sudo mdadm --zero-superblock /dev/sdb
sudo mdadm --zero-superblock /dev/sde
```

Verify:

```bash
sudo mdadm --examine /dev/sdb
```

Expected output:

```text
No md superblock detected
```

---

# 5. RAID Not Visible

## Symptom

```bash
lsblk
```

does not display the RAID device.

## Cause

The RAID creation failed or the array was not assembled.

## Resolution

Check RAID status:

```bash
cat /proc/mdstat
```

Verify RAID details:

```bash
sudo mdadm --detail /dev/md1
```

---

# 6. Filesystem Cannot Be Mounted

## Symptom

```bash
mount: wrong fs type
```

## Cause

The RAID device has not been formatted.

## Resolution

Create a filesystem:

```bash
sudo mkfs.ext4 /dev/md1
```

Mount again:

```bash
sudo mount /dev/md1 /mnt/raid0
```

---

# 7. RAID Metadata Verification

## Symptom

Need to verify RAID configuration stored on the disks.

## Resolution

```bash
sudo mdadm --examine /dev/sdb
sudo mdadm --examine /dev/sde
```

Verify:

- RAID Level
- Array UUID
- Device UUID
- Chunk Size
- Device Role
- Array State

---

# 8. Performance Verification

## Symptom

Need to confirm striping is functioning correctly.

## Resolution

Generate disk activity:

```bash
sudo dd if=/dev/zero of=/mnt/raid0/testfile.img bs=1M count=2048 status=progress
```

Monitor disk I/O:

```bash
iostat -dx 1
```

Expected behavior:

- Both member disks receive write requests simultaneously.
- Aggregate throughput is higher than a single disk.

---

# 9. RAID Cleanup

## Procedure

Unmount the filesystem:

```bash
sudo umount /mnt/raid0
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

Verify:

```bash
lsblk
```

Both disks should appear as standalone disks.

---

# 10. Best Practices

- Verify disk state before creating RAID.
- Remove stale RAID metadata before reusing disks.
- Confirm RAID health using `cat /proc/mdstat`.
- Validate RAID configuration using `mdadm --detail`.
- Verify RAID metadata using `mdadm --examine`.
- Benchmark RAID performance before production use.
- Always maintain backups because RAID 0 provides no fault tolerance.

---

# Troubleshooting Checklist

| Check | Command |
|--------|---------|
| View block devices | `lsblk` |
| RAID status | `cat /proc/mdstat` |
| RAID details | `sudo mdadm --detail /dev/md1` |
| RAID metadata | `sudo mdadm --examine /dev/sdb` |
| Filesystem usage | `df -h` |
| Disk I/O | `iostat -dx 1` |
| Stop RAID | `sudo mdadm --stop /dev/md1` |
| Remove metadata | `sudo mdadm --zero-superblock /dev/sdb` |

---

# Engineering Takeaways

- Always inspect the environment before creating a RAID array.
- Remove inactive arrays and stale metadata before reusing disks.
- Validate RAID health after every major operation.
- Confirm striping behavior through practical I/O testing.
- Perform proper cleanup after completing validation to return the system to a known-good state.
