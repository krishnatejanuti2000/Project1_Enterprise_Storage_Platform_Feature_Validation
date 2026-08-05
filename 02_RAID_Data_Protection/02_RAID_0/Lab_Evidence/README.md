# RAID 0 Creation and Validation Lab

## Project

**Project 1 – Enterprise Storage Platform Feature Validation**

## Module

**Module 2 – RAID Data Protection**

## Topic

**RAID 0 (Striping)**

---

# Objective

The objective of this lab was to create, validate, test, and document a RAID 0 array using Linux Software RAID (`mdadm`). The exercise covered the complete RAID lifecycle, including environment verification, RAID creation, filesystem configuration, functional testing, performance observation, metadata verification, and cleanup.

---

# Lab Environment

| Component | Details |
|----------|---------|
| Operating System | Ubuntu Linux |
| RAID Type | Software RAID |
| RAID Utility | mdadm |
| RAID Level | RAID 0 |
| Member Disks | /dev/sdb, /dev/sde |
| Filesystem | EXT4 |
| Mount Point | /mnt/raid0 |

---

# Lab Workflow

The following validation sequence was performed:

1. Verified available block devices.
2. Verified existing RAID configuration.
3. Stopped existing RAID arrays.
4. Verified selected disks were clean.
5. Created a RAID 0 array.
6. Verified RAID status.
7. Inspected detailed RAID configuration.
8. Created an EXT4 filesystem.
9. Mounted the RAID filesystem.
10. Verified successful mount.
11. Performed functional write validation.
12. Observed disk I/O distribution.
13. Verified RAID metadata on member disks.
14. Cleaned up the RAID environment.

---

# Files Included

| Evidence ID | Description |
|-------------|-------------|
| 01 | Block Device Verification |
| 02 | RAID Status Before Cleanup |
| 03 | Stop Existing RAID Array (md127) |
| 04 | Stop Existing RAID Array (md0) |
| 05 | Verify Clean Disks |
| 06 | Create RAID 0 |
| 07 | Verify RAID Status |
| 08 | Detailed RAID Configuration |
| 09 | Create EXT4 Filesystem |
| 10 | Mount Filesystem |
| 11 | Verify Mounted Filesystem |
| 12 | Functional Write Validation |
| 13 | Disk I/O Distribution Validation |
| 14 | RAID Metadata Verification |
| 15 | Cleanup and Environment Restoration |

---

# Key Validation Results

- Successfully created a RAID 0 array using two physical disks.
- Verified RAID configuration using `mdadm` and `/proc/mdstat`.
- Successfully formatted the RAID array with the EXT4 filesystem.
- Successfully mounted the RAID filesystem.
- Successfully performed sequential write validation using `dd`.
- Verified that write operations were distributed across both member disks, demonstrating RAID 0 striping.
- Verified RAID metadata consistency on all member disks.
- Successfully restored the environment by removing the RAID configuration.

---

# Engineering Concepts Validated

- RAID 0 (Striping)
- Linux Software RAID (`mdadm`)
- RAID Metadata (Superblock)
- RAID Array Verification
- Chunk Size
- Filesystem Creation
- Filesystem Mounting
- Sequential Write Validation
- Disk I/O Monitoring
- RAID Metadata Examination
- RAID Cleanup

---

# Lab Conclusion

The RAID 0 array was successfully created, validated, tested, and removed. The validation confirmed correct RAID configuration, successful filesystem integration, proper striping behavior during write operations, and successful cleanup of the storage environment. All objectives of the RAID 0 validation exercise were achieved.

---

# Skills Demonstrated

- Enterprise Storage Validation
- Linux Storage Administration
- Software RAID Management
- Filesystem Administration
- Storage Verification
- Performance Observation
- Storage Troubleshooting
- Technical Documentation
