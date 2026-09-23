# Troubleshooting – RAID 1

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

**Module 2 – RAID Data Protection**

**Topic: RAID 1**

---

# 1. Introduction

This document covers common RAID 1 issues encountered during creation, validation, failure simulation, rebuild, and cleanup using Linux Software RAID (`mdadm`).

The objective is to identify problems, determine their root cause, and apply the correct recovery procedure while maintaining data availability.

---

# 2. Existing RAID Metadata

## Symptom

```bash
mdadm: device or resource busy
```

or

```bash
mdadm: cannot create array
```

## Cause

The disks still contain RAID metadata from a previous RAID configuration.

## Resolution

Verify metadata:

```bash
sudo mdadm --examine /dev/sdb
sudo mdadm --examine /dev/sde
```

Remove metadata:

```bash
sudo mdadm --zero-superblock /dev/sdb
sudo mdadm --zero-superblock /dev/sde
```

Verify again:

```bash
sudo mdadm --examine /dev/sdb
```

Expected:

```text
No md superblock detected
```

---

# 3. Inactive RAID Array (md127)

## Symptom

```text
md127 : inactive
```

## Cause

Linux automatically assembled leftover RAID metadata from previous testing.

## Resolution

View details:

```bash
sudo mdadm --detail /dev/md127
```

Stop the array:

```bash
sudo mdadm --stop /dev/md127
```

Verify:

```bash
lsblk
```

---

# 4. Initial Resync In Progress

## Symptom

```text
resync = 25%
```

or

```text
finish = 15 min
```

## Cause

RAID 1 performs an initial synchronization after creation to ensure both mirror members contain identical data.

## Resolution

Allow the synchronization to complete.

Monitor progress:

```bash
cat /proc/mdstat
```

Do not simulate disk failures until the resync finishes.

---

# 5. RAID Health Verification

## Healthy State

```text
[UU]
```

Meaning:

- Both mirror members are healthy.
- Array is fully synchronized.

Verify:

```bash
cat /proc/mdstat
```

or

```bash
sudo mdadm --detail /dev/md1
```

---

# 6. RAID Degraded

## Symptom

```text
[U_]
```

or

```text
State : clean, degraded
```

## Cause

One mirror member has failed or is missing.

## Resolution

Identify the failed disk:

```bash
sudo mdadm --detail /dev/md1
```

Replace the failed disk.

Add it back:

```bash
sudo mdadm --manage /dev/md1 --add /dev/sdX
```

Monitor rebuild:

```bash
cat /proc/mdstat
```

---

# 7. Disk Failure Simulation

## Command

```bash
sudo mdadm --manage /dev/md1 --fail /dev/sde
```

Expected Result

```text
[U_]
```

Applications should continue accessing data through the surviving mirror.

---

# 8. Rebuild Monitoring

## Symptom

Replacement disk added.

## Verify

```bash
cat /proc/mdstat
```

Expected:

```text
recovery = xx%
```

or

```text
resync = xx%
```

After completion:

```text
[UU]
```

---

# 9. Filesystem Mount Failure

## Symptom

```bash
mount: wrong fs type
```

## Cause

Filesystem has not been created.

## Resolution

```bash
sudo mkfs.ext4 /dev/md1
```

Mount again:

```bash
sudo mount /dev/md1 /mnt/raid1
```

---

# 10. Data Accessibility Verification

After one disk fails:

Verify:

```bash
ls -lh /mnt/raid1
```

or

```bash
cat /mnt/raid1/<filename>
```

Expected:

Data should remain accessible.

---

# 11. Write-Intent Bitmap

## Symptom

System unexpectedly shuts down during rebuild.

## Benefit

If a write-intent bitmap is enabled:

- Only modified regions require resynchronization.
- Recovery is significantly faster.

Verify:

```bash
sudo mdadm --detail /dev/md1
```

Expected:

```text
Intent Bitmap : Internal
```

---

# 12. Cleanup Procedure

Unmount:

```bash
sudo umount /mnt/raid1
```

Stop RAID:

```bash
sudo mdadm --stop /dev/md1
```

Remove metadata:

```bash
sudo mdadm --zero-superblock /dev/sdb

sudo mdadm --zero-superblock /dev/sde
```

Verify:

```bash
lsblk
```

Both disks should appear as standalone devices.

---

# 13. Best Practices

- Always inspect disks before creating RAID.
- Remove stale metadata before reusing disks.
- Wait for initial resync to complete.
- Verify RAID health before testing failures.
- Replace failed disks immediately.
- Monitor rebuild until the array returns to `[UU]`.
- Verify application data before and after rebuild.
- Clean the environment after validation.

---

# Troubleshooting Checklist

| Check | Command |
|--------|---------|
| Block devices | `lsblk` |
| RAID status | `cat /proc/mdstat` |
| RAID details | `sudo mdadm --detail /dev/md1` |
| RAID metadata | `sudo mdadm --examine /dev/sdb` |
| Simulate failure | `sudo mdadm --manage /dev/md1 --fail /dev/sde` |
| Add replacement disk | `sudo mdadm --manage /dev/md1 --add /dev/sdX` |
| Filesystem usage | `df -h` |
| Stop RAID | `sudo mdadm --stop /dev/md1` |
| Remove metadata | `sudo mdadm --zero-superblock /dev/sdb` |

---

# Engineering Takeaways

- RAID 1 provides continuous availability during a single-disk failure.
- Always begin failure testing from a healthy `[UU]` state.
- Monitor resynchronization and rebuild using `/proc/mdstat`.
- Verify application accessibility during degraded mode.
- Replace failed disks promptly to restore redundancy.
- Use write-intent bitmaps to reduce rebuild time after interruptions.
- Perform complete cleanup after validation to return the environment to a known-good state.
