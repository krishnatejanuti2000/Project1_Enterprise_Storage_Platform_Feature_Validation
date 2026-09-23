# RAID 60 — Maximum 2+2 Failure Evidence

## 1. Test Objective

The objective of this test was to validate the maximum distributed member
failure pattern tested on the two-component RAID60 configuration.

The topology consisted of:

```text
                         RAID 60
                            |
                           md60
                         RAID 0
                       /       \
                      /         \
                   md61         md62
                  RAID 6       RAID 6
                 4 members     4 members
```

The critical failure pattern was:

```text
md61 → 2 failed members
md62 → 2 failed members
```

Therefore:

```text
2 + 2 = 4 failed members total
```

The purpose was to verify whether each RAID 6 component could remain
operational at its dual-member failure limit while the top-level RAID60
remained accessible.

---

# 2. Initial Healthy State

Before injecting the maximum failure scenario, both component RAID 6
arrays had been fully recovered and were healthy.

The active members were:

```text
md61:
loop22
loop19
loop20
loop13

md62:
loop23
loop15
loop16
loop17
```

Healthy component state:

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

The filesystem was mounted at:

```text
/mnt/raid60
```

---

# 3. Healthy Failure-Domain Model

The healthy RAID60 failure domains were:

```text
                    md60
                  RAID 0
                 /       \
                /         \
             md61         md62
            RAID 6       RAID 6
            4 members   4 members
```

Each component had four members.

RAID 6 provides dual-parity protection, so the critical distributed
failure limit for this topology was:

```text
md61 → 2 failed
md62 → 2 failed
```

---

# 4. Failure Injection Plan

The intended failure pattern was:

```text
md61:
loop22 → failed
loop19 → failed

md62:
loop23 → failed
loop15 → failed
```

The failures were intentionally injected **without removing the failed
members or beginning rebuilds**.

This was necessary to capture the exact maximum degraded state before
recovery.

---

# 5. Inject First md61 Failure

The first member failure was injected:

```bash id="9k1e4b"
sudo mdadm --manage /dev/md61 --fail /dev/loop22
```

`md61` entered degraded mode.

Intermediate condition:

```text
md61
[4/3]
```

---

# 6. Inject Second md61 Failure

The second member in the same RAID 6 group was failed:

```bash id="t4f2nm"
sudo mdadm --manage /dev/md61 --fail /dev/loop19
```

The resulting `md61` condition was:

```text
[4/2] [__UU]
```

This represented two failed members in the first RAID 6 component.

---

# 7. Inject First md62 Failure

The first member of the second RAID 6 component was failed:

```bash id="4kp8sr"
sudo mdadm --manage /dev/md62 --fail /dev/loop23
```

`md62` entered degraded mode.

---

# 8. Inject Second md62 Failure

The second member in `md62` was then failed:

```bash id="k2dx0m"
sudo mdadm --manage /dev/md62 --fail /dev/loop15
```

The resulting component condition was:

```text
md62
[4/2] [__UU]
```

The complete RAID60 failure state was now:

```text
md61 → [4/2] [__UU]
md62 → [4/2] [__UU]
```

---

# 9. Exact Maximum 2+2 Failure State

The final injected failure condition was:

```text
md61:
loop22 → FAILED
loop19 → FAILED
loop20 → ACTIVE
loop13 → ACTIVE
```

and:

```text
md62:
loop23 → FAILED
loop15 → FAILED
loop16 → ACTIVE
loop17 → ACTIVE
```

Therefore:

```text
Total failed members = 4
```

with distribution:

```text
md61 → 2
md62 → 2
```

This is the maximum distributed failure pattern tested in this
laboratory.

---

# 10. `/proc/mdstat` Validation

The RAID state was checked using:

```bash id="z8x9v6"
cat /proc/mdstat
```

The important component states were:

```text
md61
[4/2] [__UU]

md62
[4/2] [__UU]
```

The top-level device remained:

```text
md60
→ active
```

This was the critical observation of the test.

---

# 11. Failure-Domain Interpretation

The exact state can be visualized as:

```text
                          md60
                         RAID 0
                      ____/   \____
                     /             \
                    /               \
                 md61             md62
                RAID 6            RAID 6
                [__UU]            [__UU]
                /   \              /   \
             ACTIVE ACTIVE      ACTIVE ACTIVE
```

Each component RAID 6 array retained two working members.

Therefore neither component exceeded its normal dual-member failure
tolerance during the test.

---

# 12. Detailed md61 Validation

The first RAID 6 component was inspected:

```bash id="9jmq2y"
sudo mdadm --detail /dev/md61
```

The observed condition was:

```text
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 2
Spare Devices    : 0
```

The active members were:

```text
loop20
loop13
```

The failed members were:

```text
loop22
loop19
```

Member roles:

```text
loop22 → role 0 → failed
loop19 → role 1 → failed
loop20 → role 2 → active
loop13 → role 3 → active
```

---

# 13. Detailed md62 Validation

The second RAID 6 component was inspected:

```bash id="hd5q1f"
sudo mdadm --detail /dev/md62
```

The observed condition was:

```text
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 2
Spare Devices    : 0
```

The active members were:

```text
loop16
loop17
```

The failed members were:

```text
loop23
loop15
```

Member roles:

```text
loop23 → role 0 → failed
loop15 → role 1 → failed
loop16 → role 2 → active
loop17 → role 3 → active
```

---

# 14. Top-Level md60 Validation

The top-level device was checked:

```bash id="l3q1u0"
sudo mdadm --detail /dev/md60
```

The important observation was:

```text
md61 → degraded but operational
md62 → degraded but operational
md60 → active
```

The RAID 0 parent remained available because both underlying RAID 6
components remained operational.

The outer RAID 0 layer itself provided no redundancy.

---

# 15. Filesystem Accessibility

The filesystem remained mounted at:

```text
/mnt/raid60
```

The storage path remained:

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

The filesystem remained accessible in the exact 2+2 degraded state.

This demonstrated that the maximum tested distributed member-failure
scenario did not immediately interrupt logical filesystem access.

---

# 16. Directory Validation

The RAID60 directory was inspected:

```bash id="7w2x4a"
ls -lh /mnt/raid60
```

The expected test files were still present:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

The directory remained accessible with all four injected failures active.

---

# 17. File Read Validation

The test files were read while both component RAID 6 arrays were in their
maximum degraded state:

```bash id="g0me4z"
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files were successfully read.

Therefore:

```text
md61 → 2 failed
+
md62 → 2 failed
        ↓
md60 remains active
        ↓
Filesystem remains accessible
        ↓
Files remain readable
```

---

# 18. Data Integrity Validation at 2+2 State

This was the most important data-integrity check of the maximum failure
scenario.

The checksums were recalculated while all four failed members remained
failed:

```bash id="x5qf5f"
sha256sum /mnt/raid60/testfile_*.txt
```

The observed values matched the healthy baseline:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

Comparison:

```text
Healthy baseline
        =
Exact 2+2 degraded state
```

for every tested file.

---

# 19. Critical Observation

The most important result was:

```text
4 failed members total
        ↓
2 failures in md61
+
2 failures in md62
        ↓
md61 still operational
md62 still operational
        ↓
md60 still active
        ↓
Filesystem accessible
        ↓
Data readable
        ↓
SHA-256 unchanged
```

This directly demonstrates the importance of failure distribution in a
nested RAID60 architecture.

---

# 20. Why the 2+2 Pattern Was Survivable

Each component was a four-member RAID 6 array.

The observed distribution was:

```text
md61 → 2 failed + 2 working
md62 → 2 failed + 2 working
```

Each component therefore remained within the tested dual-member failure
tolerance.

The top-level RAID 0 layer could continue accessing both component
arrays.

The important distinction is:

```text
2 + 2
```

rather than:

```text
3 + 1
```

The total failed-member count is identical:

```text
4 failures
```

but the failure-domain distribution is different.

---

# 21. Comparison With a 3+1 Pattern

The tested scenario was:

```text
2 + 2
```

A different hypothetical distribution would be:

```text
3 + 1
```

For the tested four-member RAID 6 components:

```text
2 + 2
→ neither component exceeds two failed members

3 + 1
→ one component exceeds its normal dual-member tolerance
```

Therefore the two patterns cannot be treated as equivalent.

The key RAID60 troubleshooting rule remains:

```text
Failure count
+
Failure distribution
```

---

# 22. No Member Removal Before Evidence Capture

The failed members were deliberately left in the failed state during the
maximum-failure evidence collection.

The failure-state evidence therefore represents:

```text
md61 → [4/2] [__UU]
md62 → [4/2] [__UU]
```

before any replacement or rebuild operation.

This preserves the integrity of the failure-state experiment.

---

# 23. Recovery Deferred to Separate Phase

Recovery was intentionally performed only after the complete 2+2 state
had been validated.

The recovery process is documented separately in:

```text
08_Recovery_and_Rebuild.md
```

This separation provides:

```text
Failure Evidence
        +
Recovery Evidence
```

without mixing the two phases.

---

# 24. Test Result

```text
MAXIMUM 2+2 FAILURE RESULT: PASS
```

The intended four-member distributed failure condition was successfully
created and validated.

Results:

```text
[PASS] md61 member 1 failed
[PASS] md61 member 2 failed

[PASS] md62 member 1 failed
[PASS] md62 member 2 failed

[PASS] md61 reached [4/2] [__UU]
[PASS] md62 reached [4/2] [__UU]

[PASS] md61 remained operational
[PASS] md62 remained operational

[PASS] md60 remained active

[PASS] Filesystem remained accessible
[PASS] Test files remained readable

[PASS] SHA-256 values matched baseline
```

---

# 25. Maximum Failure-State Summary

```text
RAID60
  |
  +-- md61 → RAID 6
  |     ├── loop22 → FAILED
  |     ├── loop19 → FAILED
  |     ├── loop20 → ACTIVE
  |     └── loop13 → ACTIVE
  |
  +-- md62 → RAID 6
        ├── loop23 → FAILED
        ├── loop15 → FAILED
        ├── loop16 → ACTIVE
        └── loop17 → ACTIVE
```

Component states:

```text
md61 → [4/2] [__UU]
md62 → [4/2] [__UU]
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

# 26. Engineering Observations

## 26.1 Four failed members were simultaneously present

The test demonstrated:

```text
2 failures in md61
+
2 failures in md62
=
4 failed members
```

without immediate loss of the tested RAID60 filesystem.

---

## 26.2 Each RAID 6 component retained two working members

The surviving members were:

```text
md61:
loop20
loop13

md62:
loop16
loop17
```

Therefore both components remained operational.

---

## 26.3 The outer RAID 0 layer remained active

The top-level `md60` remained available because neither component RAID 6
had become unavailable.

---

## 26.4 Data remained intact

The test files were readable and their SHA-256 values matched the healthy
baseline.

---

## 26.5 This was the maximum failure pattern tested

For this specific two-group, four-member-per-group RAID60 laboratory:

```text
2 + 2
```

was the maximum distributed failure scenario successfully validated.

---

# 27. Acceptance Criteria

The maximum `2 + 2` failure test was accepted because:

```text
[✓] Two members failed in md61

[✓] Two members failed in md62

[✓] md61 reported [4/2] [__UU]

[✓] md62 reported [4/2] [__UU]

[✓] md61 remained operational

[✓] md62 remained operational

[✓] md60 remained active

[✓] Filesystem remained accessible

[✓] All test files remained present

[✓] All test files remained readable

[✓] SHA-256 matched healthy baseline

[✓] Maximum distributed failure condition successfully demonstrated
```

---

# 28. Final Test Conclusion

The RAID60 maximum `2 + 2` failure test passed successfully.

The tested sequence was:

```text
Healthy md61
[4/4] [UUUU]

Healthy md62
[4/4] [UUUU]

       ↓

Fail md61 loop22
       ↓

Fail md61 loop19
       ↓

md61 [4/2] [__UU]

       ↓

Fail md62 loop23
       ↓

Fail md62 loop15
       ↓

md62 [4/2] [__UU]

       ↓

md61 + md62 both degraded
       ↓

md60 remains active
       ↓

Filesystem remains accessible
       ↓

Files remain readable
       ↓

SHA-256 remains unchanged
       ↓

Maximum 2+2 failure validated
```

This provides the critical practical evidence that the tested RAID60
configuration can remain operational with two failed members in each of
its two RAID 6 component groups, while preserving access to the tested
filesystem and maintaining the integrity of the tested data.

