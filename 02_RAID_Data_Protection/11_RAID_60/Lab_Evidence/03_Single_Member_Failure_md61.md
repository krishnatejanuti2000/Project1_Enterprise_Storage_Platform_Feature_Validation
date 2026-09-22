# RAID 60 — Single Member Failure in md61

## 1. Test Objective

The objective of this test was to validate RAID60 behavior when a single
member of the first RAID 6 component fails.

The component under test was:

```text id="3k8u72"
md61
RAID 6
```

Initial members:

```text id="cz9n4j"
/dev/loop10
/dev/loop11
/dev/loop12
/dev/loop13
```

Failure injected:

```text id="92wl33"
/dev/loop10
```

Expected behavior:

```text id="5slx7j"
loop10 failure
      ↓
md61 enters degraded state
      ↓
md62 remains healthy
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
md61 returns healthy
```

---

# 2. Initial Healthy State

Before failure injection, the RAID60 configuration was healthy.

The topology was:

```text id="0x2bnn"
                         md60
                        RAID 0
                     /         \
                    /           \
                 md61           md62
                RAID 6          RAID 6
               / | | \         / | | \
          loop10 11 12 13  loop14 15 16 17
```

The first RAID 6 component was:

```text id="2l9sq7"
md61
→ loop10
→ loop11
→ loop12
→ loop13
```

Healthy state:

```text id="5zj7bt"
md61
[4/4] [UUUU]
```

The second component was healthy:

```text id="y3c5kp"
md62
[4/4] [UUUU]
```

The top-level device:

```text id="9rj8mv"
md60
```

was active.

---

# 3. Failure Injection

The single-member failure was intentionally injected using:

```bash id="9g81pv"
sudo mdadm --manage /dev/md61 --fail /dev/loop10
```

This marked `/dev/loop10` as failed inside the first RAID 6 component.

---

# 4. RAID State After Failure

Immediately after failure injection, `md61` entered degraded mode.

The observed state was:

```text id="3qxq0d"
md61
[4/3] [_UUU]
```

Interpretation:

```text id="72t06p"
Expected members = 4
Active members   = 3
```

Member state:

```text id="3p5r6y"
loop10 → failed
loop11 → active
loop12 → active
loop13 → active
```

The second RAID 6 component remained healthy:

```text id="g7s3w9"
md62
[4/4] [UUUU]
```

The parent `md60` remained active.

---

# 5. Failure-Domain Analysis

The failure occurred entirely inside:

```text id="mc5d93"
md61
```

The failure distribution was:

```text id="n9k2s8"
md61 → 1 failed member
md62 → 0 failed members
```

Conceptually:

```text id="5m81c8"
                    md60
                  RAID 0
                 /       \
                /         \
             md61         md62
            RAID 6       RAID 6
            [_UUU]       [UUUU]
```

RAID 6 provides protection against up to two failed members within the
component.

Therefore the first component remained operational in degraded mode.

---

# 6. Verify RAID Status

The RAID status was monitored with:

```bash id="w6q6ab"
cat /proc/mdstat
```

The important condition was:

```text id="ot1gr3"
md61 → [4/3] [_UUU]
md62 → [4/4] [UUUU]
md60 → active
```

This confirmed:

```text id="2b3gy2"
[PASS] md61 entered degraded state
[PASS] md62 remained healthy
[PASS] md60 remained active
```

---

# 7. Detailed md61 Validation

The affected RAID 6 component was inspected:

```bash id="p7y1jt"
sudo mdadm --detail /dev/md61
```

The degraded component showed:

```text id="n5r6d8"
State            : clean, degraded
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text id="1v7tk6"
/dev/loop10
```

The surviving members were:

```text id="kntr0x"
/dev/loop11
/dev/loop12
/dev/loop13
```

---

# 8. Validate Parent RAID60

The parent RAID device was inspected:

```bash id="cgln03"
sudo mdadm --detail /dev/md60
```

The top-level RAID 0 remained active because both component RAID 6
arrays were still operational:

```text id="n7h7fj"
md61 → degraded but operational
md62 → healthy
```

The outer RAID 0 layer did not provide redundancy for the failed member.

The redundancy came from `md61` itself.

---

# 9. Filesystem Accessibility Test

The filesystem remained mounted at:

```text id="7v9s1u"
/mnt/raid60
```

The storage path remained:

```text id="tqw9p0"
ext4
 ↓
/dev/md60
 ↓
RAID 0
 ↓
md61 + md62
 ↓
RAID 6 members
```

The filesystem remained accessible while `md61` was degraded.

---

# 10. Directory Validation

The RAID60 test directory was checked:

```bash id="x6kwl2"
ls -lh /mnt/raid60
```

The expected test files remained present:

```text id="bs2j0g"
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

No test file disappeared as a result of the failed member.

---

# 11. File Read Validation

The test files were read while `md61` was degraded:

```bash id="uj9i9c"
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

Therefore:

```text id="n2x6sw"
md61 degraded
      ↓
md62 healthy
      ↓
md60 active
      ↓
Filesystem accessible
      ↓
Files readable
```

---

# 12. Data Integrity Validation After Failure

The healthy-state SHA-256 baseline was:

```text id="nfbf49"
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Checksums were recalculated after the failure:

```bash id="81h8j8"
sha256sum /mnt/raid60/testfile_*.txt
```

The resulting values matched the healthy-state baseline.

Validation:

```text id="m2g1jq"
testfile_01.txt → MATCH
testfile_02.txt → MATCH
testfile_03.txt → MATCH
```

Therefore:

```text id="6v0ae5"
Baseline SHA-256
       =
Post-failure SHA-256
```

The tested data remained intact during degraded operation.

---

# 13. Failed Member Removal

After validating the degraded RAID state and data accessibility, the
failed member was removed:

```bash id="2gk1qd"
sudo mdadm --manage /dev/md61 --remove /dev/loop10
```

The component remained degraded and was ready to accept a replacement
member.

---

# 14. Replacement Member Preparation

A new replacement image was prepared:

```text id="2kqg98"
replacement1.img
```

The replacement was attached as:

```text id="5vmm3u"
/dev/loop18
```

The replacement device was verified as clean before being added.

No existing md RAID metadata was present on the replacement member.

---

# 15. Add Replacement Member

The replacement was added to `md61`:

```bash id="xy6e8b"
sudo mdadm --manage /dev/md61 --add /dev/loop18
```

Adding the replacement initiated RAID 6 recovery.

Recovery path:

```text id="o0nsw2"
Failed member
    loop10
       ↓
Replacement
    loop18
       ↓
RAID 6 reconstruction
       ↓
Rebuild
```

---

# 16. Rebuild Monitoring

Rebuild activity was monitored using:

```bash id="ahcb11"
cat /proc/mdstat
```

An observed rebuild state reached approximately:

```text id="4xjyxn"
27.2%
```

The rebuild continued until the replacement member was fully
reconstructed.

---

# 17. Rebuild Completion

After the rebuild completed, `md61` returned to:

```text id="q3q1x8"
[4/4] [UUUU]
```

The recovered member layout was:

```text id="8j7v4y"
loop18 → role 0
loop11 → role 1
loop12 → role 2
loop13 → role 3
```

The failed `loop10` member had therefore been replaced by `loop18`.

---

# 18. md61 Post-Rebuild Validation

The recovered component was inspected:

```bash id="f5c6a0"
sudo mdadm --detail /dev/md61
```

Final state:

```text id="0x3gdm"
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed that the first RAID 6 component had returned to full
health.

---

# 19. Parent RAID60 Validation After Rebuild

The top-level RAID60 device was checked again:

```bash id="o8nlr4"
sudo mdadm --detail /dev/md60
```

The hierarchy was operational:

```text id="7qd1f5"
md60 → RAID 0 → active
 |
 +-- md61 → RAID 6 → clean
 |
 +-- md62 → RAID 6 → clean
```

The complete RAID60 hierarchy had returned to its healthy state.

---

# 20. Filesystem Validation After Rebuild

The filesystem was checked:

```bash id="5m0x7g"
mount | grep raid60
```

The filesystem remained:

```text id="3qf4z0"
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

The RAID member failure and rebuild therefore did not interrupt the
tested filesystem access.

---

# 21. Post-Rebuild File Validation

The test files were read again:

```bash id="wqg8xm"
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

---

# 22. Post-Rebuild SHA-256 Validation

Checksums were recalculated after the rebuild:

```bash id="m0l0vo"
sha256sum /mnt/raid60/testfile_*.txt
```

Final values matched the healthy baseline:

```text id="m1h5ej"
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Therefore:

```text id="5ws3na"
Healthy baseline
        =
After failure
        =
After rebuild
```

for all tested files.

---

# 23. Test Result

```text id="t0o3x4"
TEST RESULT: PASS
```

### Failure handling

```text id="vchcqz"
[PASS] loop10 successfully marked failed
[PASS] md61 entered degraded state
[PASS] md62 remained healthy
[PASS] md60 remained active
```

### Data availability

```text id="3e7ttx"
[PASS] Filesystem remained accessible
[PASS] Test files remained readable
```

### Recovery

```text id="j2e3n7"
[PASS] Failed loop10 removed
[PASS] Replacement loop18 added
[PASS] RAID 6 rebuild completed
[PASS] md61 returned to [4/4] [UUUU]
[PASS] md61 returned to clean state
```

### Data integrity

```text id="8q0dfn"
[PASS] SHA-256 unchanged after failure
[PASS] SHA-256 unchanged after rebuild
```

---

# 24. Failure and Recovery Flow

The complete tested sequence was:

```text id="z4oqpl"
Healthy md61
[4/4] [UUUU]
      |
      ↓
Fail loop10
      |
      ↓
md61
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
Remove loop10
      |
      ↓
Add replacement loop18
      |
      ↓
RAID 6 rebuild
      |
      ↓
md61
[4/4] [UUUU]
      |
      ↓
Validate md61
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

# 25. Engineering Observations

## 25.1 RAID 6 remained operational after one member failure

The failed member was:

```text id="3m5v1k"
loop10
```

while:

```text id="4vkw7n"
loop11
loop12
loop13
```

remained available.

Therefore the RAID 6 component continued operating in degraded mode.

---

## 25.2 The second RAID 6 failure domain was unaffected

During this test:

```text id="2z6n0b"
md61 → degraded
md62 → healthy
```

This clearly isolated the failure to one component RAID 6 group.

---

## 25.3 The top-level RAID 0 did not reconstruct the failed member

The failed member was recovered entirely by:

```text id="1udj6k"
md61
RAID 6
```

The `md60` RAID 0 layer continued providing the logical striped device.

---

## 25.4 Data integrity remained unchanged

The SHA-256 values matched before and after the failure and rebuild.

---

## 25.5 Rebuild restored the RAID 6 protection level

The component returned from:

```text id="v7xwul"
[4/3] [_UUU]
```

to:

```text id="jvkf9w"
[4/4] [UUUU]
```

This restored the RAID 6 component to its healthy member count.

---

# 26. Final Acceptance Criteria

The single-member `md61` failure test was accepted because:

```text id="t4z5m8"
[✓] loop10 failure injected

[✓] md61 entered degraded state

[✓] md62 remained healthy

[✓] md60 remained active

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] SHA-256 remained unchanged after failure

[✓] Failed loop10 removed

[✓] Replacement loop18 added

[✓] RAID 6 rebuild completed

[✓] md61 returned to [4/4] [UUUU]

[✓] md61 returned to clean state

[✓] md60 remained operational

[✓] Final SHA-256 matched baseline
```

---

# 27. Final Test Conclusion

The RAID60 single-member failure test for `md61` passed successfully.

The tested behavior was:

```text id="1nq0tz"
One member failure in md61
        ↓
md61 degraded
        ↓
md62 remained healthy
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
Replacement loop18 added
        ↓
RAID 6 reconstruction completed
        ↓
md61 returned to [4/4] [UUUU]
        ↓
Final integrity verified
```

This validates that the tested RAID60 configuration can survive a single
member failure within the first RAID 6 component and successfully rebuild
the failed member without loss of the tested filesystem data.

