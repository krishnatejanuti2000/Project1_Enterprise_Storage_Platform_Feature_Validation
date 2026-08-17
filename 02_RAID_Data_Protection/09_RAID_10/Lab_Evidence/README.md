
# `03_create_raid10.txt`

```bash
nano 03_create_raid10.txt
```

Paste:

```text
RAID 10 LAB — RAID ARRAY CREATION
=================================

COMMAND
-------
sudo mdadm --create /dev/md10 \
  --level=10 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde


INTERACTIVE OPTIONS
-------------------
Write-intent bitmap:
    y

Continue creating array:
    y


OUTPUT
------
To optimize recovery speed, it is recommended to enable write-intent bitmap.
Write-intent bitmap:
    y

mdadm: largest drive (/dev/sdb) exceeds size (175693824K) by more than 1%
Continue creating array [y/N]?
    y

mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md10 started.


RAID CONFIGURATION
------------------
RAID device:
    /dev/md10

RAID level:
    RAID 10

RAID devices:
    4

Members:
    /dev/sdb
    /dev/sdc
    /dev/sdd
    /dev/sde

Metadata:
    Version 1.2

Intent bitmap:
    Internal


CAPACITY OBSERVATION
--------------------
The member disks are unequal in size.

Smallest member:
    /dev/sdc → approximately 167.7G

mdadm therefore selected a common member size of:

    175693824K


RESULT
------
PASS — RAID 10 array /dev/md10 created successfully.
```

Save:

```text
Ctrl+O
Enter
Ctrl+X
```

---

# `04_initial_raid_sync.txt`

```bash
nano 04_initial_raid_sync.txt
```

Paste:

```text
RAID 10 LAB — INITIAL RAID SYNCHRONIZATION
==========================================

COMMAND
-------
cat /proc/mdstat


OUTPUT
------
md10 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      351387648 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
      [>....................]  resync =  1.0% (3611904/351387648) finish=27.2min speed=212464K/sec
      bitmap: 3/3 pages [12KB], 65536KB chunk


RAID STATE DURING INITIAL RESYNC
--------------------------------
RAID level:
    RAID 10

Expected members:
    4

Active members:
    4

Member state:
    [UUUU]

Copies:
    2 near-copies

Chunk size:
    512K

Initial resync:
    In progress


RESULT
------
PASS — RAID 10 initial synchronization started successfully.
```

---

# `05_raid10_healthy_state.txt`

```bash
nano 05_raid10_healthy_state.txt
```

Paste:

```text
RAID 10 LAB — HEALTHY RAID STATE
================================

COMMAND
-------
sudo mdadm --detail /dev/md10


OUTPUT
------
Version           : 1.2
Creation Time     : Mon Aug 17 03:54:07 2026
Raid Level        : raid10
Array Size        : 351387648 (335.11 GiB 359.82 GB)
Used Dev Size     : 175693824 (167.55 GiB 179.91 GB)
Raid Devices      : 4
Total Devices     : 4
Persistence       : Superblock is persistent

Intent Bitmap     : Internal

State             : clean
Active Devices    : 4
Working Devices   : 4
Failed Devices    : 0
Spare Devices     : 0

Layout            : near=2
Chunk Size        : 512K
Consistency Policy: bitmap


RAID MEMBERS
------------
/dev/sdb → active sync set-A
/dev/sdc → active sync set-B
/dev/sdd → active sync set-A
/dev/sde → active sync set-B


ARRAY IDENTITY
--------------
Name:
    winteck:10

UUID:
    4a00c997:d17a84de:26d02f93:2164f3fe


RESULT
------
PASS — RAID 10 is fully synchronized and healthy.
All four members are active and synchronized.
```

---

# `06_create_filesystem.txt`

```bash
nano 06_create_filesystem.txt
```

Paste:

```text
RAID 10 LAB — FILESYSTEM CREATION
=================================

COMMAND
-------
sudo mkfs.ext4 /dev/md10


INITIAL WARNING
---------------
mke2fs reported that /dev/md10 already contained an ext4 filesystem:

    /dev/md10 contains a ext4 file system
    last mounted on Thu Aug 13 10:52:07 2026

The filesystem creation was explicitly confirmed:

    Proceed anyway? (y,N) y


OUTPUT
------
Creating filesystem with:
    87846912 4k blocks
    21962752 inodes

Filesystem UUID:
    1d787833-a419-4a5f-a7b4-e7f71b21d456

Superblock backups stored on blocks:
    32768
    98304
    163840
    229376
    294912
    819200
    884736
    1605632
    2654208
    4096000
    7962624
    11239424
    20480000
    23887872
    71663616
    78675968

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done


FILESYSTEM
----------
Type:
    ext4

Device:
    /dev/md10

New UUID:
    1d787833-a419-4a5f-a7b4-e7f71b21d456


RESULT
------
PASS — Fresh ext4 filesystem successfully created on /dev/md10.

NOTE
----
An old ext4 signature was detected and explicitly overwritten during
filesystem creation.
```

---

# `07_mount_raid10.txt`

```bash
nano 07_mount_raid10.txt
```

Paste:

```text
RAID 10 LAB — MOUNT RAID 10 FILESYSTEM
======================================

MOUNT POINT
-----------
/mnt/raid10


COMMANDS
--------
sudo mkdir -p /mnt/raid10

sudo mount /dev/md10 /mnt/raid10


RESULT
------
Mount completed successfully.

RAID device:
    /dev/md10

Filesystem:
    ext4

Mount point:
    /mnt/raid10

Access mode:
    read/write
```

---

# `08_mount_verification.txt`

```bash
nano 08_mount_verification.txt
```

Paste:

```text
RAID 10 LAB — MOUNT VERIFICATION
================================

COMMAND
-------
mount | grep /mnt/raid10


OUTPUT
------
/dev/md10 on /mnt/raid10 type ext4 (rw,relatime,stripe=256)


VERIFICATION
------------
RAID device:
    /dev/md10

Filesystem:
    ext4

Mount point:
    /mnt/raid10

Access:
    read/write

Stripe parameter:
    stripe=256


RESULT
------
PASS — RAID 10 ext4 filesystem is mounted successfully.
```

---

# `09_write_test_data.txt`

```bash
nano 09_write_test_data.txt
```

Paste:

```text
RAID 10 LAB — TEST DATA CREATION
================================

TEST DATA LOCATION
------------------
/mnt/raid10


FILE 1
------
Filename:
    testfile_01.txt

Contents:
    RAID10-TEST-DATA-001
    Enterprise Storage Validation
    RAID Level: RAID 10
    Primary Test: Data Accessibility
    Member Count: 4
    Layout: near=2
    Status: Initial Healthy State


FILE 2
------
Filename:
    testfile_02.txt

Contents:
    RAID10-TEST-DATA-002
    Storage Engineering Laboratory
    Test Scenario: Mirror Failure Recovery
    Single Failure: Supported
    Distributed Failure: Layout Dependent
    Rebuild Validation: Required
    Data Integrity: Required


FILE 3
------
Filename:
    testfile_03.txt

Contents:
    RAID10-TEST-DATA-003
    Linux mdadm RAID Validation
    Filesystem: ext4
    Mount Point: /mnt/raid10
    Purpose: Mirroring and Rebuild Testing
    Expected Result: Data Remains Consistent
    Final Check: PASS


VERIFICATION
------------
All three files were successfully created and read back.

RESULT
------
PASS — RAID 10 test data created successfully.
```

---

# `10_pre_failure_raid_state.txt`

```bash
nano 10_pre_failure_raid_state.txt
```

Paste:

```text
RAID 10 LAB — PRE-FAILURE HEALTHY STATE
=======================================

COMMAND
-------
cat /proc/mdstat


OUTPUT
------
md10 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      351387648 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
      bitmap: 2/3 pages [8KB], 65536KB chunk


RAID DETAIL
-----------
sudo mdadm --detail /dev/md10

State:
    clean / active

Active Devices:
    4

Working Devices:
    4

Failed Devices:
    0

Layout:
    near=2

Chunk Size:
    512K


FILESYSTEM
----------
/dev/md10 on /mnt/raid10 type ext4 (rw,relatime,stripe=256)


RESULT
------
PASS — RAID 10 healthy baseline captured before failure simulation.
```

---

# `11_fail_member_1.txt`

```bash
nano 11_fail_member_1.txt
```

Paste:

```text
RAID 10 LAB — FIRST MEMBER FAILURE
==================================

TARGET MEMBER
-------------
/dev/sdb


COMMAND
-------
sudo mdadm --manage /dev/md10 --fail /dev/sdb


RESULT
------
/dev/sdb was successfully marked as faulty.


POST-FAILURE STATE
------------------
/dev/sdb → faulty
/dev/sdc → active
/dev/sdd → active
/dev/sde → active


RAID STATE
----------
[4/3] [_UUU]


RESULT
------
PASS — First RAID 10 member failure successfully simulated.
```

---

# `12_single_member_degraded_state.txt`

```bash
nano 12_single_member_degraded_state.txt
```

Paste:

```text
RAID 10 LAB — SINGLE-MEMBER DEGRADED STATE
==========================================

COMMAND
-------
cat /proc/mdstat


OUTPUT
------
md10 : active raid10 sde[3] sdd[2] sdc[1] sdb[0](F)
      351387648 blocks super 1.2 512K chunks 2 near-copies [4/3] [_UUU]
      bitmap: 2/3 pages [8KB], 65536KB chunk


INTERPRETATION
--------------
Expected members:
    4

Working members:
    3

Failed members:
    1

Failed member:
    /dev/sdb

RAID state:
    [4/3] [_UUU]


RESULT
------
PASS — RAID 10 remained operational in degraded mode.
```

---

# `13_single_member_data_access.txt`

```bash
nano 13_single_member_data_access.txt
```

Paste:

```text
RAID 10 LAB — SINGLE-MEMBER DATA ACCESS
=======================================

FAILURE CONDITION
------------------
/dev/sdb → faulty

RAID state:
    [4/3] [_UUU]


TEST
----
All three RAID 10 test files were read after /dev/sdb failed.

Files:
    testfile_01.txt → READ SUCCESS
    testfile_02.txt → READ SUCCESS
    testfile_03.txt → READ SUCCESS


OBSERVATION
-----------
The surviving RAID 10 mirror copy continued to provide access
to the data.


RESULT
------
PASS — Single-member degraded read access confirmed.
```

---

# `14_single_member_integrity.txt`

```bash
nano 14_single_member_integrity.txt
```

Paste:

```text
RAID 10 LAB — SINGLE-MEMBER DATA INTEGRITY
==========================================

FAILURE CONDITION
------------------
/dev/sdb → faulty


COMMAND
-------
sha256sum /mnt/raid10/testfile_01.txt \
          /mnt/raid10/testfile_02.txt \
          /mnt/raid10/testfile_03.txt


BASELINE HASHES
---------------
testfile_01.txt
193c5a752e5d8d85dbb7a2b6cf196a528a16cd85e11c990421fe26caf6520fa7

testfile_02.txt
1bc12abe6b6fa40b1b26f586a2ee9a25a0e1b59102b321922f42f8d29300571e

testfile_03.txt
f72a17cefd111ed983773719bdb268091dd729a258158d4d6961f58c5a98c51c


RESULT
------
All three SHA-256 values matched the healthy-state baseline.

Conclusion:
    PASS — No data corruption observed during single-member
    degraded operation.
```

---

# `15_single_member_degraded_detail.txt`

```bash
nano 15_single_member_degraded_detail.txt
```

Paste:

```text
RAID 10 LAB — SINGLE-MEMBER DEGRADED DETAIL
===========================================

COMMAND
-------
sudo mdadm --detail /dev/md10


OUTPUT SUMMARY
--------------
State             : clean, degraded
Active Devices    : 3
Working Devices   : 3
Failed Devices    : 1
Spare Devices     : 0

Layout            : near=2
Chunk Size        : 512K
Consistency Policy: bitmap


MEMBERS
-------
/dev/sdb → faulty
/dev/sdc → active sync set-B
/dev/sdd → active sync set-A
/dev/sde → active sync set-B


RAID DEVICE MAP
---------------
RaidDevice 0 → removed / faulty /dev/sdb
RaidDevice 1 → /dev/sdc
RaidDevice 2 → /dev/sdd
RaidDevice 3 → /dev/sde


RESULT
------
PASS — Detailed single-member degraded state captured.
```

---

# `16_fail_member_2_different_mirror.txt`

```bash
nano 16_fail_member_2_different_mirror.txt
```

Paste:

```text
RAID 10 LAB — SECOND MEMBER FAILURE
===================================

FIRST FAILED MEMBER
-------------------
/dev/sdb


SECOND TARGET MEMBER
--------------------
/dev/sdd


COMMAND
-------
sudo mdadm --manage /dev/md10 --fail /dev/sdd


RESULT
------
/dev/sdd was successfully marked as faulty.


POST-FAILURE STATE
------------------
/dev/sdb → faulty
/dev/sdc → active
/dev/sdd → faulty
/dev/sde → active


RAID STATE
----------
[4/2] [_U_U]


FAILURE PATTERN
---------------
The two failed members belong to different mirror relationships
for the near=2 RAID 10 geometry.


RESULT
------
PASS — Two-member distributed failure successfully simulated.
```

---

# `17_two_member_degraded_state.txt`

```bash
nano 17_two_member_degraded_state.txt
```

Paste:

```text
RAID 10 LAB — TWO-MEMBER DEGRADED STATE
=======================================

COMMAND
-------
cat /proc/mdstat


OUTPUT
------
md10 : active raid10 sde[3] sdd[2](F) sdc[1] sdb[0](F)
      351387648 blocks super 1.2 512K chunks 2 near-copies [4/2] [_U_U]
      bitmap: 3/3 pages [12KB], 65536KB chunk


INTERPRETATION
--------------
Expected members:
    4

Working members:
    2

Failed members:
    2

Failed:
    /dev/sdb
    /dev/sdd

Working:
    /dev/sdc
    /dev/sde

RAID state:
    [4/2] [_U_U]


RESULT
------
PASS — RAID 10 remained active with two members failed because
the failures were distributed across different mirror relationships.
```

---

# `18_two_member_data_access.txt`

```bash
nano 18_two_member_data_access.txt
```

Paste:

```text
RAID 10 LAB — TWO-MEMBER DATA ACCESS
====================================

FAILURE CONDITION
------------------
/dev/sdb → faulty
/dev/sdd → faulty

RAID state:
    [4/2] [_U_U]


TEST
----
All three original test files were read while two members were failed.

Results:
    testfile_01.txt → READ SUCCESS
    testfile_02.txt → READ SUCCESS
    testfile_03.txt → READ SUCCESS


OBSERVATION
-----------
Each affected mirror relationship retained a surviving member.

The RAID 10 filesystem remained accessible.


RESULT
------
PASS — Two-member distributed-failure read access confirmed.
```

---

# `19_two_member_integrity.txt`

```bash
nano 19_two_member_integrity.txt
```

Paste:

```text
RAID 10 LAB — TWO-MEMBER DATA INTEGRITY
=======================================

FAILURE CONDITION
------------------
/dev/sdb → faulty
/dev/sdd → faulty


COMMAND
-------
sha256sum /mnt/raid10/testfile_01.txt \
          /mnt/raid10/testfile_02.txt \
          /mnt/raid10/testfile_03.txt


BASELINE HASHES
---------------
testfile_01.txt
193c5a752e5d8d85dbb7a2b6cf196a528a16cd85e11c990421fe26caf6520fa7

testfile_02.txt
1bc12abe6b6fa40b1b26f586a2ee9a25a0e1b59102b321922f42f8d29300571e

testfile_03.txt
f72a17cefd111ed983773719bdb268091dd729a258158d4d6961f58c5a98c51c


RESULT
------
All three hashes matched the healthy-state baseline.

Conclusion:
    PASS — Data integrity was preserved during the two-member
    distributed RAID 10 failure.
```

---

# `20_two_member_degraded_detail.txt`

```bash
nano 20_two_member_degraded_detail.txt
```

Paste:

```text
RAID 10 LAB — TWO-MEMBER DEGRADED DETAIL
========================================

COMMAND
-------
sudo mdadm --detail /dev/md10


OUTPUT SUMMARY
--------------
State             : clean, degraded
Active Devices    : 2
Working Devices   : 2
Failed Devices    : 2
Spare Devices     : 0

Layout            : near=2
Chunk Size        : 512K
Consistency Policy: bitmap


MEMBERS
-------
/dev/sdb → faulty
/dev/sdc → active sync set-B
/dev/sdd → faulty
/dev/sde → active sync set-B


RAID DEVICE MAP
---------------
RaidDevice 0 → removed /dev/sdb
RaidDevice 1 → /dev/sdc
RaidDevice 2 → removed /dev/sdd
RaidDevice 3 → /dev/sde


RAID STATE
----------
[4/2] [_U_U]


RESULT
------
PASS — Detailed two-member distributed-failure state captured.
```

---

# `21_same_mirror_failure_attempt.txt`

```bash
nano 21_same_mirror_failure_attempt.txt
```

Paste:

```text
RAID 10 LAB — SAME-MIRROR FAILURE ATTEMPT
=========================================

PURPOSE
-------
Demonstrate the behavior when a second member belonging to the
same RAID 10 mirror relationship as an already-failed member is
targeted.


FIRST FAILED MEMBER
-------------------
/dev/sdb


SECOND TARGET
-------------
/dev/sdc


COMMAND
-------
sudo mdadm --manage /dev/md10 --fail /dev/sdc


OUTPUT
------
mdadm: Cannot remove /dev/sdc from /dev/md10, array will be failed.


INTERPRETATION
--------------
/dev/sdb was already faulty.

Attempting to fail /dev/sdc would have removed the remaining member
required by that mirror relationship.

mdadm therefore refused the operation because the array would fail.


IMPORTANT OBSERVATION
---------------------
The lab did NOT force this failure with --force.

The objective was to preserve the valid RAID 10 array and capture
the actual mdadm safety behavior.


RESULT
------
PASS — Same-mirror failure protection behavior demonstrated.

Operational lesson:
    Do not determine RAID 10 survivability from failure count alone.
    Determine which mirror relationship each failed member belongs to.
```

---

# `22_remove_failed_members.txt`

```bash
nano 22_remove_failed_members.txt
```

Paste:

```text
RAID 10 LAB — REMOVE FAILED MEMBERS
===================================

FAILED MEMBERS
--------------
/dev/sdb
/dev/sdd


REMOVE FIRST FAILED MEMBER
--------------------------
COMMAND
-------
sudo mdadm --manage /dev/md10 --remove /dev/sdb

OUTPUT
------
mdadm: hot removed /dev/sdb from /dev/md10


REMOVE SECOND FAILED MEMBER
---------------------------
COMMAND
-------
sudo mdadm --manage /dev/md10 --remove /dev/sdd

OUTPUT
------
mdadm: hot removed /dev/sdd from /dev/md10


POST-REMOVAL STATE
------------------
/dev/sdb → removed
/dev/sdc → active
/dev/sdd → removed
/dev/sde → active

RAID state:
    [4/2] [_U_U]


RESULT
------
PASS — Both faulty members were successfully removed.
```

---

# `23_rebuild_member_1.txt`

```bash
nano 23_rebuild_member_1.txt
```

Paste:

```text
RAID 10 LAB — FIRST MEMBER REBUILD
==================================

REPLACEMENT MEMBER
------------------
/dev/sdb


COMMAND
-------
sudo mdadm --manage /dev/md10 --add /dev/sdb


OUTPUT
------
mdadm: re-added /dev/sdb


RECOVERY OBSERVATION
--------------------
Initial recovery output:

recovery = 92.5% (162529280/175693824)
finish=1.4min
speed=155648K/sec


INITIAL STATE
-------------
[4/2] [_U_U]


RECOVERY COMPLETION
-------------------
Final observed state:

[4/3] [UU_U]


INTERPRETATION
--------------
/dev/sdb was rebuilt successfully.

Current members:
    /dev/sdb → active
    /dev/sdc → active
    /dev/sdd → missing
    /dev/sde → active


RESULT
------
PASS — First replacement member /dev/sdb was successfully rebuilt.
```

---

# `24_rebuild_member_2.txt`

```bash
nano 24_rebuild_member_2.txt
```

Paste:

```text
RAID 10 LAB — SECOND MEMBER REBUILD
===================================

REPLACEMENT MEMBER
------------------
/dev/sdd


COMMAND
-------
sudo mdadm --manage /dev/md10 --add /dev/sdd


OUTPUT
------
mdadm: re-added /dev/sdd


RECOVERY
--------
/dev/sdd was re-added successfully.

The recovery completed before the next mdstat check.


FINAL OBSERVED MEMBER STATE
---------------------------
[4/4] [UUUU]


MEMBERS
-------
/dev/sdb → active
/dev/sdc → active
/dev/sdd → active
/dev/sde → active


RESULT
------
PASS — Second replacement member /dev/sdd was successfully rebuilt.
```

---

# `25_final_raid_state.txt`

```bash
nano 25_final_raid_state.txt
```

Paste:

```text
RAID 10 LAB — FINAL RAID STATE
==============================

COMMAND
-------
sudo mdadm --detail /dev/md10


OUTPUT SUMMARY
--------------
Version           : 1.2
Raid Level        : raid10
Array Size        : 351387648 (335.11 GiB 359.82 GB)
Used Dev Size     : 175693824 (167.55 GiB 179.91 GB)
Raid Devices      : 4
Total Devices     : 4

State             : clean
Active Devices    : 4
Working Devices   : 4
Failed Devices    : 0
Spare Devices     : 0

Layout            : near=2
Chunk Size        : 512K
Consistency Policy: bitmap


FINAL MEMBERS
-------------
/dev/sdb → active sync set-A
/dev/sdc → active sync set-B
/dev/sdd → active sync set-A
/dev/sde → active sync set-B


FINAL RAID STATE
----------------
[4/4] [UUUU]


RESULT
------
PASS — RAID 10 redundancy was fully restored after the failure
and sequential rebuild cycle.
```

---

# `26_final_data_integrity.txt`

```bash
nano 26_final_data_integrity.txt
```

Paste:

```text
RAID 10 LAB — FINAL DATA INTEGRITY VERIFICATION
================================================

PURPOSE
-------
Verify that all test data remained unchanged after:

    1. Single-member failure
    2. Two-member distributed failure
    3. Failed-member removal
    4. First replacement rebuild
    5. Second replacement rebuild


COMMAND
-------
sha256sum /mnt/raid10/testfile_01.txt \
          /mnt/raid10/testfile_02.txt \
          /mnt/raid10/testfile_03.txt


HEALTHY BASELINE
----------------
testfile_01.txt
193c5a752e5d8d85dbb7a2b6cf196a528a16cd85e11c990421fe26caf6520fa7

testfile_02.txt
1bc12abe6b6fa40b1b26f586a2ee9a25a0e1b59102b321922f42f8d29300571e

testfile_03.txt
f72a17cefd111ed983773719bdb268091dd729a258158d4d6961f58c5a98c51c


FINAL OBSERVED HASHES
---------------------
testfile_01.txt
193c5a752e5d8d85dbb7a2b6cf196a528a16cd85e11c990421fe26caf6520fa7

testfile_02.txt
1bc12abe6b6fa40b1b26f586a2ee9a25a0e1b59102b321922f42f8d29300571e

testfile_03.txt
f72a17cefd111ed983773719bdb268091dd729a258158d4d6961f58c5a98c51c


RESULT
------
All final SHA-256 values exactly matched the healthy-state baseline.

Final data-integrity result:
    PASS
```

---

# `27_raid10_geometry_and_roles.txt`

```bash
nano 27_raid10_geometry_and_roles.txt
```

Paste:

```text
RAID 10 LAB — GEOMETRY AND DEVICE ROLES
=======================================

COMMAND
-------
sudo mdadm --examine /dev/sdb /dev/sdc /dev/sdd /dev/sde


COMMON ARRAY INFORMATION
------------------------
Raid Level:
    raid10

Raid Devices:
    4

Layout:
    near=2

Chunk Size:
    512K


DEVICE ROLES
------------

/dev/sdb:
    Device Role: Active device 0

/dev/sdc:
    Device Role: Active device 1

/dev/sdd:
    Device Role: Active device 2

/dev/sde:
    Device Role: Active device 3


ARRAY STATE OBSERVATIONS
------------------------
When /dev/sdb was faulty:

    /dev/sdb → Array State AAAA before failure history
    /dev/sdc → Array State .AAA
    /dev/sdd → Array State .AAA
    /dev/sde → Array State .AAA


IMPORTANT LAB OBSERVATION
-------------------------
The earlier interpretation of mdadm's:

    set-A
    set-B

labels was not sufficient by itself to determine the mirror
relationship.

The actual RAID10 geometry is defined by:

    RAID Level = raid10
    Layout     = near=2
    Device Role 0..3


OPERATIONAL LESSON
------------------
Never infer RAID10 mirror topology solely from the visible
set-A/set-B labels.

Use the actual RAID geometry and device roles together with
the mdadm behavior observed during failure testing.


SAME-MIRROR FAILURE EVIDENCE
----------------------------
With /dev/sdb already faulty, attempting to fail /dev/sdc produced:

    mdadm: Cannot remove /dev/sdc from /dev/md10, array will be failed.

This demonstrated that the second failure would have exceeded the
survivable redundancy of the array.


RESULT
------
PASS — Actual RAID10 near=2 geometry and failure behavior were
validated directly using mdadm metadata.
```

---

# `README.md`

```bash
nano README.md
```

Paste:

````markdown
# RAID 10 — Lab Evidence

## Lab Objective

Validate Linux software RAID 10 using `mdadm`, including:

- RAID 10 creation
- Initial synchronization
- Actual RAID10 layout verification
- Filesystem creation
- Mounting
- Test-data creation
- SHA-256 baseline
- Single-member failure
- Single-member degraded operation
- Two-member distributed failure
- Data access during distributed failure
- Same-mirror failure behavior
- Failed-member removal
- Sequential rebuild
- Final RAID health
- Final data-integrity verification

---

## RAID 10 Configuration

```text
RAID Device        : /dev/md10
RAID Level         : RAID 10
RAID Devices       : 4
Layout             : near=2
Copies             : 2
Chunk Size         : 512K
Metadata            : 1.2
Consistency Policy : bitmap
````

### RAID Members

```text
/dev/sdb
/dev/sdc
/dev/sdd
/dev/sde
```

### Filesystem

```text
Filesystem : ext4
Mount      : /mnt/raid10
```

---

# Healthy Baseline

```text
[4/4] [UUUU]

Active Devices  : 4
Working Devices : 4
Failed Devices  : 0
```

Result:

```text
PASS
```

---

# Test Data

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

Healthy SHA-256 baseline:

```text
testfile_01.txt
193c5a752e5d8d85dbb7a2b6cf196a528a16cd85e11c990421fe26caf6520fa7

testfile_02.txt
1bc12abe6b6fa40b1b26f586a2ee9a25a0e1b59102b321922f42f8d29300571e

testfile_03.txt
f72a17cefd111ed983773719bdb268091dd729a258158d4d6961f58c5a98c51c
```

---

# Scenario 1 — Single Member Failure

Failed:

```text
/dev/sdb
```

Observed:

```text
[4/3] [_UUU]
```

Results:

```text
RAID remained operational       PASS
Files remained readable         PASS
SHA-256 unchanged               PASS
```

---

# Scenario 2 — Two Member Failures in Different Mirror Relationships

Failed:

```text
/dev/sdb
/dev/sdd
```

Surviving:

```text
/dev/sdc
/dev/sde
```

Observed:

```text
[4/2] [_U_U]
```

Results:

```text
RAID remained operational       PASS
Files remained readable         PASS
SHA-256 unchanged               PASS
```

This demonstrates that RAID10 can survive two simultaneous
member failures when the failures are distributed across different
mirror relationships.

---

# Scenario 3 — Same-Mirror Failure Attempt

First failure:

```text
/dev/sdb
```

Second attempted failure:

```text
/dev/sdc
```

Command:

```bash
sudo mdadm --manage /dev/md10 --fail /dev/sdc
```

Observed:

```text
mdadm: Cannot remove /dev/sdc from /dev/md10, array will be failed.
```

The failure was NOT forced.

The purpose was to preserve the valid array while recording the
actual `mdadm` behavior when the second failure would exceed the
survivable RAID10 redundancy.

Result:

```text
PASS — Same-mirror failure protection behavior demonstrated.
```

---

# Rebuild

## First Replacement

Replacement:

```text
/dev/sdb
```

Command:

```bash
sudo mdadm --manage /dev/md10 --add /dev/sdb
```

Observed recovery:

```text
recovery = 92.5%
```

Final state after first rebuild:

```text
[4/3] [UU_U]
```

Result:

```text
PASS
```

---

## Second Replacement

Replacement:

```text
/dev/sdd
```

Command:

```bash
sudo mdadm --manage /dev/md10 --add /dev/sdd
```

Final state:

```text
[4/4] [UUUU]
```

Result:

```text
PASS
```

---

# Final RAID Health

```text
State             : clean
Active Devices    : 4
Working Devices   : 4
Failed Devices    : 0
Spare Devices     : 0
Layout            : near=2
Chunk Size        : 512K
```

Final members:

```text
/dev/sdb → active sync
/dev/sdc → active sync
/dev/sdd → active sync
/dev/sde → active sync
```

Result:

```text
PASS
```

---

# Final Data Integrity

All three SHA-256 values exactly matched the healthy baseline after
the complete failure and rebuild cycle.

Result:

```text
PASS
```

---

# RAID10 Geometry

Actual Linux layout:

```text
Layout : near=2
```

Device roles:

```text
/dev/sdb → role 0
/dev/sdc → role 1
/dev/sdd → role 2
/dev/sde → role 3
```

The lab demonstrated that `set-A` and `set-B` labels in
`mdadm --detail` should not be treated alone as a direct definition
of mirror pairs.

Actual RAID10 geometry and mdadm failure behavior were used to
determine survivability.

---

# Overall Validation

```text
RAID 10 Creation                  PASS
Initial Synchronization           PASS
Layout Verification               PASS
Filesystem Creation               PASS
Mount Verification                PASS
Test Data Creation                PASS
Baseline SHA-256                  PASS

Single Member Failure             PASS
Single Member Degraded Access     PASS
Single Member Integrity           PASS

Two Member Distributed Failure    PASS
Two Member Data Access            PASS
Two Member Integrity              PASS

Same-Mirror Failure Attempt       PASS
Failed Member Removal             PASS
First Rebuild                     PASS
Second Rebuild                    PASS

Final RAID Health                 PASS
Final Data Integrity              PASS
```

# Final Result

```text
========================================
 RAID 10 ENTERPRISE FEATURE VALIDATION
========================================

                PASS
```
