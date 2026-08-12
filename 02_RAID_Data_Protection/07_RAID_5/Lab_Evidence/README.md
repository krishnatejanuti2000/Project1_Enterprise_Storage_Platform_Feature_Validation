# RAID 5 Lab Evidence

## 1. Lab Objective

The objective of this lab was to practically validate RAID 5
creation, normal operation, single-member failure tolerance,
degraded operation, failed-member removal, member re-addition,
recovery, and post-recovery data accessibility using Linux
software RAID (`mdadm`).

---

## 2. Lab Environment

Operating System:

    Ubuntu Linux

RAID Management Utility:

    mdadm v4.5 - 2025-12-16 - Ubuntu 4.5-5ubuntu1

RAID Device:

    /dev/md0

RAID Level:

    RAID 5

Number of RAID Members:

    4

RAID Member Devices:

    /dev/sdb
    /dev/sdc
    /dev/sdd
    /dev/sde

System Disk:

    /dev/sda

The system disk /dev/sda was not used for the RAID 5 lab.

---

## 3. RAID Configuration

RAID Level:

    RAID 5

RAID Devices:

    4

Metadata Version:

    1.2

Layout:

    left-symmetric

Chunk Size:

    512K

Consistency Policy:

    bitmap

Write-Intent Bitmap:

    Internal bitmap

Array Device:

    /dev/md0

Array Size:

    527081472 blocks
    502.66 GiB
    539.73 GB

Used Device Size:

    175693824 blocks
    167.55 GiB
    179.91 GB

---

## 4. Filesystem Configuration

Filesystem:

    ext4

Filesystem Device:

    /dev/md0

Mount Point:

    /mnt/raid5

Filesystem UUID:

    940e87ff-8c11-42a4-b949-45af37ba4657

---

## 5. Test Data

Test directory:

    /mnt/raid5/testdata

Test file:

    /mnt/raid5/testdata/testfile.txt

Test data:

    RAID5-TEST-DATA-001

The test data was written and successfully read before
performing the failure scenario.

---

## 6. Failure and Recovery Scenario

The following sequence was performed:

```text
Healthy RAID 5
      ↓
Initial synchronization
      ↓
Filesystem creation
      ↓
Filesystem mount
      ↓
Test data creation
      ↓
Healthy RAID verification
      ↓
Fail /dev/sdc
      ↓
RAID enters degraded mode
      ↓
Verify data accessibility
      ↓
Remove /dev/sdc
      ↓
Verify degraded array
      ↓
Inspect /dev/sdc RAID metadata
      ↓
Re-add /dev/sdc
      ↓
RAID returns to four-member state
      ↓
Verify final RAID health
      ↓
Verify test data
