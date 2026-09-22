# RAID 60 — Two-Member Failure in md61

## 1. Test Objective

The objective of this test was to validate RAID60 behavior when two
members of the same underlying RAID 6 component fail.

The component under test was:

```text id="6cw6jl"
md61
RAID 6
```

The test specifically validates RAID 6's dual-member failure tolerance.

Initial healthy members before failure injection:

```text id="js5n0x"
/dev/loop18
/dev/loop11
/dev/loop12
/dev/loop13
```

The two members intentionally failed were:

```text id="z3p0f7"
/dev/loop11
/dev/loop12
```

Expected behavior:

```text id="b7k6d3"
Two members fail in md61
        ↓
md61 becomes heavily degraded
        ↓
md61 remains operational within RAID 6 tolerance
        ↓
md62 remains healthy
        ↓
md60 remains active
        ↓
Filesystem remains accessible
        ↓
Test data remains readable
        ↓
SHA-256 remains unchanged
```

---

# 2. Initial Healthy State

Before the two-member failure was injected, `md61` had been restored to
a healthy four-member configuration.

The component membership was:

```text id="sq5h7p"
md61
├── loop18
├── loop11
├── loop12
└── loop13
```

Healthy state:

```text id="v5p7fl"
md61
[4/4] [UUUU]
```

The second RAID 6 component remained healthy:

```text id="qp44w3"
md62
[4/4] [UUUU]
```

The top-level array:

```text id="1d6p19"
md60
```

was active.

---

# 3. Failure Pattern

The test injected two failures into the **same** RAID 6 component.

Failure pattern:

```text id="c8gc4c"
md61:

loop18 → healthy
loop11 → failed
loop12 → failed
loop13 → healthy
```

Therefore:

```text id="e6q10c"
md61 → 2 failed members
md62 → 0 failed members
```

This is the critical two-member failure condition for a RAID 6 group.

---

# 4. Inject First Failure

The first member was failed intentionally:

```bash id="ubm8l5"
sudo mdadm --manage /dev/md61 --fail /dev/loop11
```

After the first failure:

```text id="bpuwlv"
md61
[4/3] [U_UU]
```

The component remained operational with three active members.

---

# 5. Inject Second Failure

The second member in the same component was then failed:

```bash id="8jktis"
sudo mdadm --manage /dev/md61 --fail /dev/loop12
```

The final two-member failure state became:

```text id="63egm7"
md61
[4/2] [U__U]
```

This was the intended test condition.

---

# 6. Interpretation of `[4/2] [U__U]`

The state:

```text id="9bq9fw"
[4/2] [U__U]
```

means:

```text id="y4a1tf"
Expected RAID members = 4
Currently active      = 2
```

The device-state pattern:

```text id="x4trqq"
U__U
```

indicates:

```text id="aqpwh1"
loop18 → active
loop11 → unavailable
loop12 → unavailable
loop13 → active
```

Therefore exactly two members of `md61` were failed.

This is the maximum normal member-failure count protected by the
two-parity RAID 6 component.

---

# 7. Failure-Domain Analysis

The failure distribution was:

```text id="hz5s24"
md61 → 2 failed
md62 → 0 failed
```

Conceptually:

```text id="f4hp0z"
                    md60
                  RAID 0
                 /       \
                /         \
             md61         md62
            RAID 6       RAID 6
            [U__U]       [UUUU]
```

The affected component had exactly two failed members.

The second component was completely healthy.

---

# 8. RAID 6 Protection Analysis

RAID 6 uses dual parity:

```text id="qk79t2"
P parity
+
Q parity
```

This provides protection against two failed members within the same
RAID 6 component under normal RAID 6 fault-tolerance assumptions.

Therefore the tested state:

```text id="hzmlr5"
2 failed members in md61
```

remained within the component's normal protection capability.

However, the component was now at its maximum degraded-member condition.

No additional member failure should be considered safe.

---

# 9. Verify RAID Status

The RAID status was monitored using:

```bash id="jjvzwm"
cat /proc/mdstat
```

The important state was:

```text id="t8s8ck"
md61 → [4/2] [U__U]
md62 → [4/4] [UUUU]
md60 → active
```

This confirmed:

```text id="m3wqco"
[PASS] Two failures in md61
[PASS] md61 remains operational
[PASS] md62 remains healthy
[PASS] md60 remains active
```

---

# 10. Detailed md61 Validation

The affected component was inspected using:

```bash id="k2xx30"
sudo mdadm --detail /dev/md61
```

The resulting state showed:

```text id="l29hx4"
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 2
Spare Devices    : 0
```

The active members were:

```text id="q8o9ll"
loop18
loop13
```

The failed members were:

```text id="7p1m4x"
loop11
loop12
```

The component therefore remained operational with two active members.

---

# 11. Device-State Interpretation

The final member layout was:

```text id="jt8wul"
Role 0 → loop18 → active
Role 1 → loop11 → faulty
Role 2 → loop12 → faulty
Role 3 → loop13 → active
```

Conceptually:

```text id="j2oyc6"
md61

loop18  → ✅
loop11  → ❌
loop12  → ❌
loop13  → ✅
```

The RAID 6 component retained sufficient surviving information for the
tested logical data access.

---

# 12. Validate md62

The unaffected component was also checked:

```bash id="1v5vxi"
sudo mdadm --detail /dev/md62
```

Its state remained healthy:

```text id="eoe5rj"
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

Therefore the failure remained isolated to `md61`.

---

# 13. Top-Level RAID60 Validation

The parent device was checked:

```bash id="q9jceh"
sudo mdadm --detail /dev/md60
```

The top-level RAID 0 remained active.

Hierarchy:

```text id="ehf2ru"
md60
 ├── md61 → degraded, operational
 └── md62 → healthy
```

The outer RAID 0 layer continued exposing the logical RAID60 device.

It did not provide the two-member redundancy; that protection came from
the RAID 6 component `md61`.

---

# 14. Filesystem Accessibility Test

The RAID60 filesystem remained mounted at:

```text id="c6uk0a"
/mnt/raid60
```

The storage path remained:

```text id="iqy6gz"
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

The filesystem remained accessible while `md61` was operating with two
failed members.

---

# 15. Directory Validation

The test directory was checked:

```bash id="4cmwkr"
ls -lh /mnt/raid60
```

The expected test files remained present:

```text id="u4p8il"
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

No test file disappeared after the two-member failure.

---

# 16. File Read Validation

The test files were read while `md61` was in the `[4/2] [U__U]` state:

```bash id="0y3b8k"
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

This demonstrated:

```text id="nr9xd3"
Two failed members in md61
        ↓
RAID 6 remains operational
        ↓
md60 remains active
        ↓
Filesystem remains accessible
        ↓
Files remain readable
```

---

# 17. Data Integrity Validation

The healthy-state SHA-256 baseline was:

```text id="nvw4ab"
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Checksums were recalculated while `md61` remained in the two-member
failure state:

```bash id="0y6qcm"
sha256sum /mnt/raid60/testfile_*.txt
```

The resulting values matched the healthy baseline.

Validation:

```text id="prvn9m"
testfile_01.txt → MATCH
testfile_02.txt → MATCH
testfile_03.txt → MATCH
```

Therefore:

```text id="2c89h8"
Healthy baseline
      =
Two-member degraded state
```

for the tested files.

---

# 18. Protection State Assessment

At this point the RAID 6 component had reached:

```text id="z4m2xx"
Failed Devices = 2
```

This is a critical operational condition.

The component had used its normal dual-member failure protection.

Therefore the remaining state should be treated as:

```text id="b1f98v"
Operational
+
Maximum degraded state
+
Immediate recovery required
```

A further member failure could exceed the component's normal RAID 6
fault tolerance.

---

# 19. Why This Scenario Is Important

This test validates an important difference between:

```text id="2h5xv5"
One failed member
```

and:

```text id="n7q6ds"
Two failed members in the same RAID 6 group
```

After one failure:

```text id="dxgwrn"
Protection remains
```

After two failures:

```text id="vokhww"
The normal dual-failure tolerance has been exhausted
```

Therefore the second state has substantially less protection against any
additional hardware failure.

---

# 20. No Immediate Rebuild in This Evidence File

The purpose of this document is to capture the **two-member failure
state itself**.

At the point of this evidence:

```text id="1b9f5h"
md61
[4/2] [U__U]
```

was intentionally validated before moving into the controlled recovery
sequence.

The member-removal, replacement, rebuild, and post-rebuild validation
are documented separately in:

```text id="z9fbik"
08_Recovery_and_Rebuild.md
```

This separation preserves the exact failure-state evidence.

---

# 21. Test Result

```text id="coqb6o"
TEST RESULT: PASS
```

The intended two-member failure condition was successfully created and
validated.

Results:

```text id="d1hj2u"
[PASS] First md61 member failed
[PASS] Second md61 member failed
[PASS] md61 reached [4/2] [U__U]
[PASS] md61 remained operational
[PASS] md62 remained healthy
[PASS] md60 remained active
[PASS] Filesystem remained accessible
[PASS] Test files remained readable
[PASS] SHA-256 values matched baseline
```

---

# 22. Failure-State Summary

```text id="5w1i6r"
Component:
md61 → RAID 6

Failed members:
loop11
loop12

Surviving members:
loop18
loop13

State:
[4/2] [U__U]

Failed Devices:
2

md62:
[4/4] [UUUU]

md60:
active

Filesystem:
accessible

Data:
readable

SHA-256:
MATCH
```

---

# 23. Engineering Observations

## 23.1 RAID 6 successfully demonstrated dual-member tolerance

Two members failed within the same component:

```text id="v9ug7v"
loop11 ❌
loop12 ❌
```

while:

```text id="46xuqb"
loop18 ✅
loop13 ✅
```

remained active.

---

## 23.2 The failure remained inside one component domain

The second RAID 6 array was unaffected:

```text id="jp8gcb"
md62 → [4/4] [UUUU]
```

This clearly isolated the failure to `md61`.

---

## 23.3 RAID60 remained accessible

The top-level RAID 0 remained active because the affected RAID 6 component
was still operational.

---

## 23.4 The component reached its maximum degraded state

With:

```text id="m4q1mj"
Failed Devices = 2
```

`md61` had exhausted its normal two-member RAID 6 protection.

This makes immediate recovery important.

---

## 23.5 Data integrity remained intact

The SHA-256 values matched the healthy baseline even with two members of
the same component failed.

---

# 24. Acceptance Criteria

The two-member `md61` failure test was accepted because:

```text id="73xj1d"
[✓] Two members failed in the same RAID 6 component

[✓] md61 entered [4/2] [U__U]

[✓] md61 remained operational

[✓] md62 remained healthy

[✓] md60 remained active

[✓] Filesystem remained mounted

[✓] Test files remained present

[✓] Test files remained readable

[✓] SHA-256 matched the healthy baseline

[✓] Maximum degraded RAID 6 state was successfully demonstrated
```

---

# 25. Final Test Conclusion

The RAID60 two-member failure test for `md61` passed successfully.

The tested behavior was:

```text id="jdtj3s"
Healthy md61
[4/4] [UUUU]
      |
      ↓
Fail loop11
      |
      ↓
md61
[4/3] [U_UU]
      |
      ↓
Fail loop12
      |
      ↓
md61
[4/2] [U__U]
      |
      ↓
md61 remains operational
      |
      ↓
md62 remains healthy
      |
      ↓
md60 remains active
      |
      ↓
Filesystem remains accessible
      |
      ↓
Data remains readable
      |
      ↓
SHA-256 remains unchanged
      |
      ↓
PASS
```

This test provides practical evidence that the tested RAID60
configuration can tolerate two simultaneous member failures within the
same RAID 6 component while maintaining access to the tested filesystem
and preserving the tested data.

