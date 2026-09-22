# RAID 60 — Distributed Member Failure

## 1. Test Objective

The objective of this test was to validate RAID60 behavior when one member
fails in each underlying RAID 6 component.

This scenario is important because RAID60 contains multiple independent
RAID 6 failure domains.

The tested topology was:

```text
                         RAID 60
                            |
                           md60
                         RAID 0
                       /       \
                      /         \
                   md61         md62
                  RAID 6       RAID 6
```

The distributed failure pattern was:

```text
md61 → one failed member
md62 → one failed member
```

Expected behavior:

```text
One member fails in md61
        +
One member fails in md62
        ↓
Both RAID 6 components become degraded
        ↓
Both remain operational
        ↓
md60 remains active
        ↓
Filesystem remains accessible
        ↓
Test data remains readable
        ↓
Data integrity remains unchanged
```

---

# 2. Initial Healthy State

Before injecting the distributed failure, both component RAID 6 arrays
were healthy.

The active members were:

```text
md61:
loop18
loop11
loop12
loop13

md62:
loop21
loop15
loop16
loop17
```

The component states were:

```text
md61
[4/4] [UUUU]

md62
[4/4] [UUUU]
```

The top-level array was:

```text
md60
RAID 0
```

and was active.

The filesystem remained mounted at:

```text
/mnt/raid60
```

---

# 3. Failure Distribution

The test intentionally introduced:

```text
md61 → loop18 failure
md62 → loop21 failure
```

Therefore:

```text
md61 → 1 failed member
md62 → 1 failed member
```

The failure distribution was:

```text
                 RAID 60
                    |
                  md60
                 RAID 0
                /       \
               /         \
            md61         md62
           1 failed     1 failed
```

Each RAID 6 component remained within its normal single-member
fault-tolerance capability.

---

# 4. Inject md61 Failure

The first distributed failure was injected into `md61`:

```bash
sudo mdadm --manage /dev/md61 --fail /dev/loop18
```

The member state became:

```text
loop18 → failed
loop11 → active
loop12 → active
loop13 → active
```

The component entered degraded mode.

---

# 5. Inject md62 Failure

The second distributed failure was then injected into `md62`:

```bash
sudo mdadm --manage /dev/md62 --fail /dev/loop21
```

The member state became:

```text
loop21 → failed
loop15 → active
loop16 → active
loop17 → active
```

Both component RAID 6 arrays were now degraded simultaneously.

---

# 6. RAID State After Both Failures

The resulting `/proc/mdstat` state showed:

```text
md61
[4/3] [_UUU]

md62
[4/3] [_UUU]
```

The top-level array remained active:

```text
md60 → active
```

Therefore:

```text
md61 → degraded
md62 → degraded
md60 → active
```

This was the intended distributed failure condition.

---

# 7. Failure-Domain Analysis

The failures were distributed across the two independent RAID 6
components:

```text
md61 → 1 failed member
md62 → 1 failed member
```

Conceptually:

```text
                    md60
                  RAID 0
                 /       \
                /         \
             md61         md62
            RAID 6       RAID 6
            [_UUU]       [_UUU]
```

Neither component had exceeded its normal RAID 6 fault-tolerance
capability.

Therefore both components remained operational.

---

# 8. Verify RAID Status

The RAID state was monitored with:

```bash
cat /proc/mdstat
```

The important conditions were:

```text
md61 → [4/3] [_UUU]
md62 → [4/3] [_UUU]
md60 → active
```

This confirmed:

```text
[PASS] md61 degraded
[PASS] md62 degraded
[PASS] md60 remained active
```

---

# 9. Detailed md61 Validation

The first affected component was inspected using:

```bash
sudo mdadm --detail /dev/md61
```

The degraded state showed:

```text
State            : clean, degraded
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text
/dev/loop18
```

The surviving members were:

```text
/dev/loop11
/dev/loop12
/dev/loop13
```

---

# 10. Detailed md62 Validation

The second affected component was inspected using:

```bash
sudo mdadm --detail /dev/md62
```

The degraded state showed:

```text
State            : clean, degraded
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text
/dev/loop21
```

The surviving members were:

```text
/dev/loop15
/dev/loop16
/dev/loop17
```

---

# 11. Validate Top-Level RAID60

The parent device was checked using:

```bash
sudo mdadm --detail /dev/md60
```

The top-level RAID 0 remained active.

Hierarchy:

```text
md60
 ├── md61 → degraded but operational
 └── md62 → degraded but operational
```

The RAID 0 layer did not provide redundancy for either failed member.

The protection came from the individual RAID 6 component arrays.

---

# 12. Filesystem Accessibility Test

The RAID60 filesystem remained mounted at:

```text
/mnt/raid60
```

The storage path was:

```text
ext4
 ↓
/dev/md60
 ↓
RAID 0
 ↓
md61 + md62
 ↓
RAID 6
```

The filesystem remained accessible while both component arrays were
degraded.

---

# 13. Directory Validation

The RAID60 directory was checked:

```bash
ls -lh /mnt/raid60
```

The expected test files remained present:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

No test data disappeared as a result of the distributed failures.

---

# 14. File Read Validation

The test files were read while both RAID 6 components were degraded:

```bash
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

Therefore:

```text
md61 degraded
+
md62 degraded
        ↓
md60 active
        ↓
Filesystem accessible
        ↓
Files readable
```

---

# 15. Data Integrity Validation

The healthy-state SHA-256 baseline was:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Checksums were recalculated while both components were degraded:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

The resulting values matched the healthy baseline.

Validation:

```text
testfile_01.txt → MATCH
testfile_02.txt → MATCH
testfile_03.txt → MATCH
```

Therefore:

```text
Healthy baseline
        =
Distributed degraded state
```

for all tested files.

---

# 16. Why This Scenario Matters

This test demonstrates that RAID60 can have simultaneous degraded
component groups while the top-level logical device remains accessible.

The state was:

```text
md61 → degraded
md62 → degraded
md60 → active
```

This is different from testing only one degraded component.

The engineer must therefore validate every RAID 6 failure domain
individually.

---

# 17. Failure Distribution vs Failure Count

The total failure count was:

```text
2 failed members
```

But the important distribution was:

```text
md61 → 1
md62 → 1
```

This is a distributed:

```text
1 + 1
```

failure pattern.

The result differs from:

```text
2 + 0
```

because the failure domains are different.

The correct RAID60 troubleshooting question is therefore:

```text
How many members failed?
+
Where did those failures occur?
```

---

# 18. Protection-State Assessment

At the time of validation:

```text
md61 → one failed member
md62 → one failed member
```

Therefore each component still retained additional RAID 6 protection.

However, the system as a whole was operating in a reduced-redundancy
state.

The appropriate operational state was:

```text
Operational
+
Degraded
+
Recovery required
```

The purpose of the next recovery phase was to restore both component
arrays to their fully redundant state.

---

# 19. No Immediate Rebuild Evidence in This File

This document records the distributed failure condition itself.

At this point:

```text
md61 → [4/3] [_UUU]
md62 → [4/3] [_UUU]
```

was intentionally validated before the recovery sequence.

Member removal, replacement, rebuild progress, and final recovery are
documented separately in:

```text
08_Recovery_and_Rebuild.md
```

This keeps the failure-state evidence distinct from the recovery
evidence.

---

# 20. Test Result

```text
TEST RESULT: PASS
```

The intended distributed failure condition was successfully created and
validated.

Results:

```text
[PASS] md61 loop18 failed
[PASS] md62 loop21 failed

[PASS] md61 entered degraded state
[PASS] md62 entered degraded state

[PASS] md61 remained operational
[PASS] md62 remained operational

[PASS] md60 remained active

[PASS] Filesystem remained accessible
[PASS] Test files remained readable

[PASS] SHA-256 values matched baseline
```

---

# 21. Failure-State Summary

```text
RAID60
  |
  +-- md61 → RAID 6
  |     ├── loop18 → FAILED
  |     ├── loop11 → ACTIVE
  |     ├── loop12 → ACTIVE
  |     └── loop13 → ACTIVE
  |
  +-- md62 → RAID 6
        ├── loop21 → FAILED
        ├── loop15 → ACTIVE
        ├── loop16 → ACTIVE
        └── loop17 → ACTIVE
```

Component states:

```text
md61 → [4/3] [_UUU]
md62 → [4/3] [_UUU]
```

Top-level:

```text
md60 → active
```

Filesystem:

```text
/mnt/raid60 → accessible
```

Data:

```text
test files → readable
```

Integrity:

```text
SHA-256 → MATCH
```

---

# 22. Engineering Observations

## 22.1 Both RAID 6 components can be degraded simultaneously

The test successfully produced:

```text
md61 → degraded
md62 → degraded
```

while maintaining access to the top-level filesystem.

---

## 22.2 Failure domains remained independent

The two failed members belonged to different RAID 6 components:

```text
loop18 → md61
loop21 → md62
```

This confirms the distributed nature of the failure.

---

## 22.3 The top-level RAID 0 remained active

The outer layer continued to provide the logical striped device because
both component RAID 6 arrays remained operational.

---

## 22.4 RAID 6 provided the actual protection

The continued operation of the array depended on the RAID 6 components,
not on the outer RAID 0 layer.

---

## 22.5 Data integrity remained unchanged

The SHA-256 values matched the original healthy-state baseline while both
components were degraded.

---

# 23. Acceptance Criteria

The distributed member-failure test was accepted because:

```text
[✓] One member failed in md61

[✓] One member failed in md62

[✓] md61 entered [4/3] [_UUU]

[✓] md62 entered [4/3] [_UUU]

[✓] Both component arrays remained operational

[✓] md60 remained active

[✓] Filesystem remained mounted

[✓] Test files remained present

[✓] Test files remained readable

[✓] SHA-256 matched the healthy baseline

[✓] Distributed 1+1 failure pattern successfully demonstrated
```

---

# 24. Final Test Conclusion

The RAID60 distributed member-failure test passed successfully.

The tested sequence was:

```text
Healthy RAID60
      |
      ↓
Fail md61 loop18
      |
      ↓
md61 degraded
      |
      ↓
Fail md62 loop21
      |
      ↓
md61 degraded + md62 degraded
      |
      ↓
md60 remains active
      |
      ↓
Filesystem remains accessible
      |
      ↓
Test files remain readable
      |
      ↓
SHA-256 remains unchanged
      |
      ↓
PASS
```

This provides practical evidence that the tested RAID60 configuration can
remain operational when one member fails in each independent RAID 6
component, while preserving access to the tested filesystem and
maintaining the tested data integrity.

