# RAID 60 — Single Member Failure in md62

## 1. Test Objective

The objective of this test was to validate RAID60 behavior when a single
member of the second RAID 6 component fails.

The component under test was:

```text
md62
RAID 6
```

At the start of this test, the active members of `md62` were:

```text
/dev/loop14
/dev/loop15
/dev/loop16
/dev/loop17
```

The member intentionally failed was:

```text
/dev/loop14
```

Expected behavior:

```text
loop14 failure
      ↓
md62 enters degraded state
      ↓
md61 remains healthy
      ↓
md60 remains active
      ↓
Filesystem remains accessible
      ↓
Test data remains readable
      ↓
Replacement member added
      ↓
RAID 6 rebuild
      ↓
md62 returns healthy
```

---

# 2. Initial Healthy State

Before failure injection, `md62` was a healthy four-member RAID 6
component.

Topology:

```text
                         md60
                        RAID 0
                     /         \
                    /           \
                 md61           md62
                RAID 6          RAID 6
               / | | \         / | | \
          loop18 11 12 13  loop14 15 16 17
```

The first component was healthy:

```text
md61
[4/4] [UUUU]
```

The second component under test was also healthy:

```text
md62
[4/4] [UUUU]
```

The top-level device was:

```text
md60
```

and was active.

---

# 3. Failure Injection

The single-member failure was intentionally injected using:

```bash
sudo mdadm --manage /dev/md62 --fail /dev/loop14
```

This marked `/dev/loop14` as failed within the second RAID 6 component.

---

# 4. RAID State After Failure

After the failure, `md62` entered degraded mode.

The observed state was:

```text
md62
[4/3] [_UUU]
```

Interpretation:

```text
Expected members = 4
Active members   = 3
```

Member state:

```text
loop14 → failed
loop15 → active
loop16 → active
loop17 → active
```

The unaffected component remained healthy:

```text
md61
[4/4] [UUUU]
```

The top-level `md60` remained active.

---

# 5. Failure-Domain Analysis

The failure occurred in:

```text
md62
```

and not in:

```text
md61
```

Therefore:

```text
md61 → 0 failed members
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
            [UUUU]       [_UUU]
```

The failure was contained within the second RAID 6 failure domain.

A single failed member remains within RAID 6's normal fault-tolerance
capability.

---

# 6. Verify RAID Status

The RAID state was monitored using:

```bash
cat /proc/mdstat
```

The important condition was:

```text
md61 → [4/4] [UUUU]
md62 → [4/3] [_UUU]
md60 → active
```

This confirmed:

```text
[PASS] md62 entered degraded state
[PASS] md61 remained healthy
[PASS] md60 remained active
```

---

# 7. Detailed md62 Validation

The affected component was inspected using:

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
/dev/loop14
```

The surviving members were:

```text
/dev/loop15
/dev/loop16
/dev/loop17
```

---

# 8. Validate md61

The unaffected RAID 6 component was also checked:

```bash
sudo mdadm --detail /dev/md61
```

Its state remained:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

This confirmed that the failure remained isolated to `md62`.

---

# 9. Validate Top-Level RAID60

The parent logical device was checked:

```bash
sudo mdadm --detail /dev/md60
```

The hierarchy remained operational:

```text
md60
 ├── md61 → healthy
 └── md62 → degraded but operational
```

The RAID 0 layer did not provide the redundancy for `loop14`.

The protection came from the RAID 6 component `md62`.

---

# 10. Filesystem Accessibility Test

The RAID60 filesystem remained mounted at:

```text
/mnt/raid60
```

The storage stack was:

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

The filesystem remained accessible while `md62` was degraded.

---

# 11. Directory Validation

The test directory was checked:

```bash
ls -lh /mnt/raid60
```

The expected files remained present:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

No tested file disappeared after the member failure.

---

# 12. File Read Validation

The test files were read while `md62` was degraded:

```bash
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

Therefore:

```text
md62 degraded
      ↓
md61 healthy
      ↓
md60 active
      ↓
Filesystem accessible
      ↓
Files readable
```

---

# 13. Data Integrity Validation After Failure

The healthy-state SHA-256 baseline was:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Checksums were recalculated after the failure:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

The resulting values matched the healthy-state baseline.

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
Post-failure SHA-256
```

The tested data remained intact during degraded operation.

---

# 14. Failed Member Removal

After the degraded state and data integrity had been validated, the
failed member was removed:

```bash
sudo mdadm --manage /dev/md62 --remove /dev/loop14
```

The component remained degraded while waiting for a replacement.

---

# 15. Replacement Member Preparation

A new replacement image was prepared:

```text
replacement4.img
```

It was attached as:

```text
/dev/loop21
```

The replacement member was checked before use and was clean.

No existing md RAID metadata was present on the replacement member.

---

# 16. Add Replacement Member

The replacement was added to `md62`:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loop21
```

This initiated RAID 6 recovery.

Recovery path:

```text
Failed loop14
      ↓
Replacement loop21
      ↓
RAID 6 P/Q reconstruction
      ↓
Rebuild
```

---

# 17. Rebuild Monitoring

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

An observed intermediate rebuild state reached approximately:

```text
54.3%
```

The recovery continued until the missing member was completely
reconstructed.

---

# 18. Rebuild Completion

After the rebuild completed, `md62` returned to:

```text
[4/4] [UUUU]
```

The recovered member layout was:

```text
loop21 → role 0
loop15 → role 1
loop16 → role 2
loop17 → role 3
```

The failed `loop14` member had therefore been replaced by `loop21`.

---

# 19. md62 Post-Rebuild Validation

The recovered component was inspected using:

```bash
sudo mdadm --detail /dev/md62
```

Final state:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed that the second RAID 6 component had returned to full
health.

---

# 20. Parent RAID60 Validation After Rebuild

The top-level RAID60 device was checked again:

```bash
sudo mdadm --detail /dev/md60
```

The complete hierarchy was operational:

```text
md60 → RAID 0 → active
 |
 +-- md61 → RAID 6 → clean
 |
 +-- md62 → RAID 6 → clean
```

The RAID60 configuration had therefore returned to its healthy state.

---

# 21. Filesystem Validation After Rebuild

The filesystem was checked:

```bash
mount | grep raid60
```

The filesystem remained:

```text
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

This confirmed filesystem continuity through the member failure and
rebuild.

---

# 22. Post-Rebuild File Validation

The test files were read again:

```bash
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

---

# 23. Post-Rebuild SHA-256 Validation

The checksums were recalculated after recovery:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

The final values matched the original baseline:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Therefore:

```text
Healthy baseline
      =
After failure
      =
After rebuild
```

for all tested files.

---

# 24. Test Result

```text
TEST RESULT: PASS
```

### Failure handling

```text
[PASS] loop14 successfully marked failed
[PASS] md62 entered degraded state
[PASS] md61 remained healthy
[PASS] md60 remained active
```

### Data availability

```text
[PASS] Filesystem remained accessible
[PASS] Test files remained readable
```

### Recovery

```text
[PASS] Failed loop14 removed
[PASS] Replacement loop21 added
[PASS] RAID 6 rebuild completed
[PASS] md62 returned to [4/4] [UUUU]
[PASS] md62 returned to clean state
```

### Data integrity

```text
[PASS] SHA-256 unchanged after failure
[PASS] SHA-256 unchanged after rebuild
```

---

# 25. Failure and Recovery Flow

The complete tested sequence was:

```text
Healthy md62
[4/4] [UUUU]
      |
      ↓
Fail loop14
      |
      ↓
md62
[4/3] [_UUU]
      |
      ↓
Validate RAID state
      |
      ↓
Validate filesystem
      |
      ↓
Validate test files
      |
      ↓
Validate SHA-256
      |
      ↓
Remove loop14
      |
      ↓
Add replacement loop21
      |
      ↓
RAID 6 rebuild
      |
      ↓
md62
[4/4] [UUUU]
      |
      ↓
Validate md62
      |
      ↓
Validate md60
      |
      ↓
Validate filesystem
      |
      ↓
Validate SHA-256
      |
      ↓
PASS
```

---

# 26. Engineering Observations

## 26.1 RAID 6 tolerated the single failed member

The failed member was:

```text
loop14
```

while:

```text
loop15
loop16
loop17
```

remained active.

Therefore `md62` continued operating in degraded mode.

---

## 26.2 The first RAID 6 failure domain was unaffected

During the test:

```text
md61 → [4/4] [UUUU]
md62 → [4/3] [_UUU]
```

This clearly isolated the failure to `md62`.

---

## 26.3 The top-level RAID 0 remained active

The parent `md60` continued exposing the logical RAID60 device while the
component RAID 6 array remained operational.

---

## 26.4 Data integrity remained unchanged

The SHA-256 values matched the original baseline during the degraded
state and after the rebuild.

---

## 26.5 Rebuild restored full RAID 6 protection

The component changed from:

```text
[4/3] [_UUU]
```

to:

```text
[4/4] [UUUU]
```

and:

```text
Failed Devices = 0
```

---

# 27. Acceptance Criteria

The single-member `md62` failure test was accepted because:

```text
[✓] loop14 failure injected

[✓] md62 entered degraded state

[✓] md61 remained healthy

[✓] md60 remained active

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] SHA-256 remained unchanged after failure

[✓] Failed loop14 removed

[✓] Replacement loop21 added

[✓] RAID 6 rebuild completed

[✓] md62 returned to [4/4] [UUUU]

[✓] md62 returned to clean state

[✓] md60 remained operational

[✓] Final SHA-256 matched baseline
```

---

# 28. Final Test Conclusion

The RAID60 single-member failure test for `md62` passed successfully.

The tested behavior was:

```text
One member failure in md62
        ↓
md62 degraded
        ↓
md61 remained healthy
        ↓
md60 remained active
        ↓
Filesystem remained accessible
        ↓
Test data remained readable
        ↓
SHA-256 remained unchanged
        ↓
Failed member removed
        ↓
Replacement loop21 added
        ↓
RAID 6 reconstruction completed
        ↓
md62 returned to [4/4] [UUUU]
        ↓
Final integrity verified
```

This validates that the tested RAID60 configuration can survive a single
member failure within the second RAID 6 component and successfully
rebuild the failed member without loss of the tested filesystem data.

