# RAID 6 — Lab Evidence

## Lab Objective

Validate Linux software RAID 6 behavior using `mdadm`, including:

- RAID 6 creation
- Initial synchronization
- Filesystem creation
- Mounting
- Test-data creation
- Single-member failure
- Single-member degraded operation
- Two-member failure
- Two-member degraded operation
- Read access during two-member failure
- Write access during two-member failure
- Failed-member removal
- Sequential rebuild of two replacement members
- Final RAID health verification
- Final data-integrity verification

---

## RAID 6 Configuration

```text
RAID Device      : /dev/md6
RAID Level       : RAID 6
RAID Devices     : 4
Chunk Size       : 512K
Layout           : left-symmetric
Metadata         : 1.2
Consistency      : bitmap
```

### RAID Members

```text
/dev/sdb
/dev/sdc
/dev/sdd
/dev/sde
```

### Filesystem

```text
Filesystem       : ext4
Mount Point      : /mnt/raid6
```

---

## Initial Healthy State

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

## Test Data

The lab used:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
two_disk_degraded_test.txt
```

SHA-256 was used as the data-integrity verification mechanism.

---

# Failure Scenario 1 — Single Member Failure

Failed member:

```text
/dev/sdb
```

Result:

```text
[4/3] [_UUU]
```

RAID remained operational.

### Data Access

```text
testfile_01.txt → PASS
testfile_02.txt → PASS
testfile_03.txt → PASS
```

### Integrity

All original SHA-256 values matched the healthy baseline.

Result:

```text
PASS
```

---

# Failure Scenario 2 — Two Member Failures

Failed members:

```text
/dev/sdb
/dev/sdc
```

Result:

```text
[4/2] [__UU]
```

Working members:

```text
/dev/sdd
/dev/sde
```

The RAID 6 array remained active at its normal two-member
fault-tolerance limit.

---

## Two-Member Read Test

All original test files remained readable:

```text
testfile_01.txt → PASS
testfile_02.txt → PASS
testfile_03.txt → PASS
```

---

## Two-Member Write Test

A new file was written while two members were failed:

```text
two_disk_degraded_test.txt
```

Content:

```text
RAID6-TWO-DISK-DEGRADED-WRITE-TEST
```

The file was successfully read back.

Result:

```text
PASS
```

---

## Two-Member Integrity Test

All original SHA-256 values remained identical to the healthy
baseline.

Result:

```text
PASS
```

---

# Rebuild Validation

## First Replacement

Replacement member:

```text
/dev/sdb
```

Action:

```text
mdadm --manage /dev/md6 --add /dev/sdb
```

Recovery completed successfully.

Intermediate state:

```text
[4/3] [U_UU]
```

Result:

```text
PASS
```

---

## Second Replacement

Replacement member:

```text
/dev/sdc
```

Action:

```text
mdadm --manage /dev/md6 --add /dev/sdc
```

Recovery completed successfully.

Final member state:

```text
[4/4] [UUUU]
```

Result:

```text
PASS
```

---

# Final RAID State

```text
Raid Level        : raid6
State             : clean
Active Devices    : 4
Working Devices   : 4
Failed Devices    : 0
Spare Devices     : 0
Layout            : left-symmetric
Chunk Size        : 512K
Consistency Policy: bitmap
```

Members:

```text
/dev/sdb → active sync
/dev/sdc → active sync
/dev/sdd → active sync
/dev/sde → active sync
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

# Final Data Integrity

All original test-file SHA-256 values matched the original
healthy-state baseline after:

* Single-member failure
* Two-member failure
* Degraded reads
* Degraded write
* First rebuild
* Second rebuild

The additional file created during the two-member degraded state
also remained intact.

Overall data-integrity result:

```text
PASS
```

---

# Parity Failure Scope

RAID 6 uses distributed P and Q parity.

There is no permanently dedicated:

```text
P disk
Q disk
```

Therefore the lab did not perform a physical "P-only disk" or
"Q-only disk" failure simulation.

Parity-only behavior was covered during the RAID 6 theory phase.

See:

```text
../08_RAID_6.md
../Dual-Parity_P_and_Q_and_Two_Disk_Recovery_Calculation.md
```

---

# Environment Anomaly

During an earlier RAID 6 creation attempt, an external SSH
framework/cleanup process removed RAID metadata from the lab disks.

The issue was resolved by:

```text
Unmounting /dev/sdb
Removing the remaining ext4 signature
Verifying clean disks
Recreating RAID 6
Repeating synchronization
Repeating the complete lab
```

The successful RAID 6 validation run completed normally afterward.

See:

```text
28_lab_anomaly_ssh_framework.txt
```

---

# Final Validation Summary

```text
RAID 6 Creation                    PASS
Initial Synchronization            PASS
Healthy RAID State                 PASS
Filesystem Creation                PASS
Filesystem Mount                   PASS
Test Data Creation                 PASS
Baseline SHA-256                   PASS

Single Member Failure              PASS
Single Member Degraded Operation  PASS
Single Member Data Access          PASS
Single Member Integrity            PASS

Two Member Failure                 PASS
Two Member Degraded Operation      PASS
Two Member Read Access             PASS
Two Member Write Access            PASS
Two Member Integrity               PASS

First Replacement Rebuild          PASS
Second Replacement Rebuild         PASS

Final RAID Health                  PASS
Final Data Integrity               PASS
```

# Overall Result

```text
========================================
 RAID 6 ENTERPRISE FEATURE VALIDATION
========================================

                PASS
```

The RAID 6 practical lab successfully demonstrated two-member
fault tolerance, degraded operation, data accessibility, degraded
writes, sequential rebuild, and final data integrity preservation.

```
