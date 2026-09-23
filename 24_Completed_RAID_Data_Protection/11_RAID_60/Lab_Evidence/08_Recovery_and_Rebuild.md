# RAID 60 — Recovery and Rebuild Evidence

## 1. Recovery Objective

The objective of this phase was to validate the recovery and rebuild
behavior of the RAID60 configuration after the planned member-failure
scenarios.

The recovery process was validated for:

```text
Single-member failure in md61
+
Two-member failure in md61
+
Single-member failure in md62
+
Distributed failure
+
Maximum 2+2 distributed failure
```

The RAID60 topology was:

```text
                         md60
                        RAID 0
                     /         \
                    /           \
                 md61           md62
                RAID 6          RAID 6
```

Recovery always occurred inside the affected RAID 6 component.

The top-level RAID 0 layer did not reconstruct failed members.

---

# 2. Recovery Model

The RAID60 recovery model was:

```text
Failed member
      ↓
Identify component RAID 6
      ↓
Confirm degraded state
      ↓
Remove failed member
      ↓
Prepare clean replacement
      ↓
Add replacement
      ↓
RAID 6 reconstruction
      ↓
Monitor rebuild
      ↓
Verify component health
      ↓
Verify md60
      ↓
Verify filesystem
      ↓
Verify data integrity
```

For a four-member RAID 6 component:

```text
Healthy:
[4/4] [UUUU]

One failed:
[4/3] [_UUU]

Two failed:
[4/2] [U__U] or [4/2] [__UU]

Recovered:
[4/4] [UUUU]
```

The exact device position depends on which member failed.

---

# 3. Recovery Scenario 1 — md61 Single-Member Failure

The first single-member failure occurred in:

```text
md61
```

Failed member:

```text
/dev/loop10
```

Initial healthy state:

```text
md61
[4/4] [UUUU]
```

After failure:

```text
md61
[4/3] [_UUU]
```

The surviving members were:

```text
loop11
loop12
loop13
```

The parent `md60` remained active.

---

# 4. Remove Failed md61 Member

The failed member was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop10
```

The component remained degraded while waiting for a replacement.

---

# 5. Prepare md61 Replacement

The first replacement image was:

```text
replacement1.img
```

It was attached as:

```text
/dev/loop18
```

The replacement member was verified as clean before being added.

---

# 6. Add md61 Replacement

The replacement member was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop18
```

This initiated RAID 6 recovery.

Recovery path:

```text
loop10 → failed
    ↓
loop18 → replacement
    ↓
RAID 6 reconstruction
    ↓
rebuild
```

---

# 7. md61 Rebuild Monitoring

Rebuild activity was monitored through:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
27.2%
```

The rebuild continued until all expected members were restored.

---

# 8. md61 Recovery Completion

After rebuild completion:

```text
md61
[4/4] [UUUU]
```

Final members:

```text
loop18 → role 0
loop11 → role 1
loop12 → role 2
loop13 → role 3
```

The failed `loop10` member had been replaced by `loop18`.

---

# 9. md61 Post-Rebuild Validation

The recovered component was inspected using:

```bash
sudo mdadm --detail /dev/md61
```

The final state was:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
Spare Devices    : 0
```

The first RAID 6 component had therefore returned to full health.

---

# 10. Recovery Scenario 2 — md61 Two-Member Failure

The second recovery scenario followed the two-member failure inside
`md61`.

Failed members:

```text
loop11
loop12
```

The failure state was:

```text
md61
[4/2] [U__U]
```

Surviving members:

```text
loop18
loop13
```

The filesystem remained accessible and SHA-256 values matched the
healthy baseline before recovery.

---

# 11. Recovery Strategy for Two Failed Members

The two failed members were recovered sequentially.

The strategy was:

```text
Recover first failed member
        ↓
Verify rebuild
        ↓
Recover second failed member
        ↓
Verify rebuild
        ↓
Verify md61 healthy
```

This made each reconstruction independently observable.

---

# 12. Recover First md61 Failed Member

The failed member:

```text
loop11
```

was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop11
```

The replacement image:

```text
replacement2.img
```

was attached as:

```text
/dev/loop19
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop19
```

This initiated RAID 6 reconstruction.

---

# 13. First md61 Rebuild Completion

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
69.1%
```

After completion, the component was:

```text
md61
[4/3] [UU_U]
```

The recovered replacement was:

```text
loop19 → role 1
```

The second failed member, `loop12`, remained faulty.

The component therefore remained degraded intentionally while the second
failed member was recovered.

---

# 14. Recover Second md61 Failed Member

The remaining failed member:

```text
loop12
```

was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop12
```

The replacement image:

```text
replacement3.img
```

was attached as:

```text
/dev/loop20
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop20
```

This initiated the second RAID 6 reconstruction.

---

# 15. Second md61 Rebuild Completion

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
91.4%
```

After completion:

```text
md61
[4/4] [UUUU]
```

Final members:

```text
loop18 → role 0
loop19 → role 1
loop20 → role 2
loop13 → role 3
```

---

# 16. md61 Two-Member Recovery Validation

The recovered component was inspected:

```bash
sudo mdadm --detail /dev/md61
```

Final state:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

The two-member failure in `md61` had therefore been completely
recovered.

---

# 17. Recovery Scenario 3 — md62 Single-Member Failure

The next single-member failure occurred in:

```text
md62
```

Failed member:

```text
/dev/loop14
```

The failure state was:

```text
md62
[4/3] [_UUU]
```

The surviving members were:

```text
loop15
loop16
loop17
```

`md61` remained healthy.

---

# 18. Remove Failed md62 Member

The failed member was removed:

```bash
sudo mdadm --manage /dev/md62 --remove /dev/loop14
```

---

# 19. Prepare md62 Replacement

The replacement image:

```text
replacement4.img
```

was attached as:

```text
/dev/loop21
```

The replacement device was verified as clean.

---

# 20. Add md62 Replacement

The replacement was added:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loop21
```

This initiated RAID 6 reconstruction.

Recovery path:

```text
loop14 → failed
    ↓
loop21 → replacement
    ↓
P/Q reconstruction
    ↓
rebuild
```

---

# 21. md62 Rebuild Monitoring

Rebuild activity was monitored using:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
54.3%
```

The rebuild continued until all expected members were restored.

---

# 22. md62 Recovery Completion

After rebuild completion:

```text
md62
[4/4] [UUUU]
```

Final members:

```text
loop21 → role 0
loop15 → role 1
loop16 → role 2
loop17 → role 3
```

The failed `loop14` member had been replaced by `loop21`.

---

# 23. md62 Post-Rebuild Validation

The recovered component was inspected:

```bash
sudo mdadm --detail /dev/md62
```

Final state:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

The second RAID 6 component had returned to full health.

---

# 24. Recovery Scenario 4 — Distributed 1+1 Failure

The distributed failure scenario used:

```text
md61 → loop18 failed
md62 → loop21 failed
```

The resulting states were:

```text
md61
[4/3] [_UUU]

md62
[4/3] [_UUU]
```

Both component arrays were degraded simultaneously.

The top-level `md60` remained active.

The test files remained accessible and SHA-256 values matched the
healthy baseline.

---

# 25. Recover Distributed md61 Failure

The failed member:

```text
loop18
```

was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop18
```

Replacement image:

```text
replacement5.img
```

was attached as:

```text
/dev/loop22
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop22
```

This initiated the md61 rebuild.

---

# 26. Distributed md61 Rebuild Completion

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
52.1%
```

After completion:

```text
md61
[4/4] [UUUU]
```

The recovered member was:

```text
loop22 → role 0
```

Final md61 members:

```text
loop22
loop19
loop20
loop13
```

---

# 27. Recover Distributed md62 Failure

The failed member in `md62` was:

```text
loop21
```

It was removed:

```bash
sudo mdadm --manage /dev/md62 --remove /dev/loop21
```

Replacement image:

```text
replacement6.img
```

was attached as:

```text
/dev/loop23
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loop23
```

This initiated the md62 rebuild.

---

# 28. Distributed md62 Rebuild Completion

The rebuild was monitored with:

```bash
cat /proc/mdstat
```

An observed rebuild progress point was approximately:

```text
36.0%
```

After completion:

```text
md62
[4/4] [UUUU]
```

Final members:

```text
loop23
loop15
loop16
loop17
```

The failed `loop21` member had been replaced by `loop23`.

---

# 29. Distributed Recovery Validation

After both distributed failures were recovered:

```text
md61 → [4/4] [UUUU]
md62 → [4/4] [UUUU]
```

The top-level `md60` remained operational.

Data integrity was validated against the original healthy-state
checksums.

The distributed failure and recovery sequence therefore completed
successfully.

---

# 30. Recovery Scenario 5 — Maximum 2+2 Failure

The critical recovery scenario began from the fully healthy state:

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

The maximum failure state had been:

```text
md61
[4/2] [__UU]

md62
[4/2] [__UU]
```

Failed members:

```text
md61:
loop22
loop19

md62:
loop23
loop15
```

The top-level `md60` remained active.

All three test files were readable and SHA-256 values matched the healthy
baseline.

---

# 31. 2+2 Recovery Strategy

The four failed members were recovered sequentially.

Recovery order:

```text
md61 failure 1
      ↓
md61 rebuild
      ↓
md61 failure 2
      ↓
md61 rebuild
      ↓
md62 failure 1
      ↓
md62 rebuild
      ↓
md62 failure 2
      ↓
md62 rebuild
      ↓
Final validation
```

This ensured that every rebuild stage could be independently verified.

---

# 32. Recover First md61 Member in 2+2 Scenario

The first failed member was:

```text
loop22
```

It was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop22
```

The replacement image:

```text
replacement7.img
```

was attached as:

```text
/dev/loop24
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop24
```

---

# 33. First 2+2 md61 Rebuild

The rebuild was monitored:

```bash
cat /proc/mdstat
```

An observed progress point was approximately:

```text
43.7%
```

After completion:

```text
md61
[4/3] [U_UU]
```

The recovered member was:

```text
loop24 → role 0
```

The second failed member, `loop19`, remained faulty.

---

# 34. Recover Second md61 Member in 2+2 Scenario

The remaining failed member:

```text
loop19
```

was removed:

```bash
sudo mdadm --manage /dev/md61 --remove /dev/loop19
```

The replacement image:

```text
replacement8.img
```

was attached as:

```text
/dev/loop25
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md61 --add /dev/loop25
```

---

# 35. Second 2+2 md61 Rebuild

The rebuild was monitored:

```bash
cat /proc/mdstat
```

An observed progress point was approximately:

```text
43.7%
```

After completion:

```text
md61
[4/4] [UUUU]
```

Final md61 membership:

```text
loop25 → role 1
loop24 → role 0
loop20 → role 2
loop13 → role 3
```

The first component was therefore completely restored.

---

# 36. Recover First md62 Member in 2+2 Scenario

The first failed member in `md62` was:

```text
loop23
```

It was removed:

```bash
sudo mdadm --manage /dev/md62 --remove /dev/loop23
```

The replacement image:

```text
replacement9.img
```

was attached as:

```text
/dev/loop26
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loop26
```

---

# 37. First 2+2 md62 Rebuild

The rebuild was monitored:

```bash
cat /proc/mdstat
```

An observed progress point was approximately:

```text
31.1%
```

After completion:

```text
md62
[4/3] [U_UU]
```

The recovered member was:

```text
loop26 → role 0
```

The second failed member, `loop15`, remained faulty.

---

# 38. Recover Second md62 Member in 2+2 Scenario

The remaining failed member:

```text
loop15
```

was removed:

```bash
sudo mdadm --manage /dev/md62 --remove /dev/loop15
```

The replacement image:

```text
replacement10.img
```

was attached as:

```text
/dev/loop27
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md62 --add /dev/loop27
```

---

# 39. Second 2+2 md62 Rebuild

The rebuild was monitored:

```bash
cat /proc/mdstat
```

An observed progress point was approximately:

```text
49.9%
```

After completion:

```text
md62
[4/4] [UUUU]
```

Final md62 membership:

```text
loop26 → role 0
loop27 → role 1
loop16 → role 2
loop17 → role 3
```

The second component was completely restored.

---

# 40. Final 2+2 Recovery State

After all four failed members had been replaced:

```text
md61
[4/4] [UUUU]

md62
[4/4] [UUUU]
```

Final members:

```text
md61:
loop25
loop24
loop20
loop13

md62:
loop27
loop26
loop17
loop16
```

The top-level `md60` remained active.

---

# 41. Final `/proc/mdstat` Validation

The final RAID state was:

```text
md60 : active raid0 md62[1] md61[0]
      4182016 blocks super 1.2 512k chunks

md62 : active raid6 loop27[5] loop26[4] loop17[3] loop16[2]
      2093056 blocks ...

md61 : active raid6 loop25[5] loop24[4] loop20[6] loop13[3]
      2093056 blocks ...

unused devices: <none>
```

The critical healthy states were:

```text
md61 → [4/4] [UUUU]
md62 → [4/4] [UUUU]
```

---

# 42. Final md61 Validation

The recovered `md61` component was inspected:

```bash
sudo mdadm --detail /dev/md61
```

Final condition:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

---

# 43. Final md62 Validation

The recovered `md62` component was inspected:

```bash
sudo mdadm --detail /dev/md62
```

Final condition:

```text
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

---

# 44. Final md60 Validation

The top-level array was inspected:

```bash
sudo mdadm --detail /dev/md60
```

Final condition:

```text
RAID Level       : raid0
RAID Devices     : 2
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 0
State            : clean
```

The top-level members were:

```text
md61 → role 0
md62 → role 1
```

---

# 45. Final Filesystem Validation

The filesystem remained:

```text
/dev/md60
```

mounted at:

```text
/mnt/raid60
```

Mount validation:

```bash
mount | grep raid60
```

Result:

```text
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

---

# 46. Final Capacity Validation

Filesystem capacity was checked using:

```bash
df -h /mnt/raid60
```

Final result:

```text
/dev/md60  3.9G  1.1M  3.7G  1% /mnt/raid60
```

The logical RAID60 filesystem therefore remained fully usable after
recovery.

---

# 47. Final File Accessibility Validation

The test files were checked:

```bash
ls -lh /mnt/raid60
```

The expected files remained present:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

They were then read:

```bash
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All three files remained readable.

---

# 48. Final Data Integrity Validation

After complete recovery, SHA-256 checksums were recalculated:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

Final values:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

These matched the original healthy-state baseline.

Therefore:

```text
Healthy baseline
      =
After failures
      =
After complete rebuild
```

for all tested files.

---

# 49. Replacement Member Summary

The complete replacement history from the fresh final run was:

| Failed Member | Component | Replacement Image | Replacement Loop |
| ------------- | --------- | ----------------- | ---------------- |
| loop10        | md61      | replacement1.img  | loop18           |
| loop11        | md61      | replacement2.img  | loop19           |
| loop12        | md61      | replacement3.img  | loop20           |
| loop14        | md62      | replacement4.img  | loop21           |
| loop18        | md61      | replacement5.img  | loop22           |
| loop21        | md62      | replacement6.img  | loop23           |
| loop22        | md61      | replacement7.img  | loop24           |
| loop19        | md61      | replacement8.img  | loop25           |
| loop23        | md62      | replacement9.img  | loop26           |
| loop15        | md62      | replacement10.img | loop27           |

Final active members:

```text
md61:
loop25
loop24
loop20
loop13

md62:
loop27
loop26
loop17
loop16
```

---

# 50. Rebuild Progress Summary

Observed rebuild progress points during the fresh final run included:

```text
md61 single-member recovery
→ 27.2%

md61 two-member recovery — first member
→ 69.1%

md61 two-member recovery — second member
→ 91.4%

md62 single-member recovery
→ 54.3%

Distributed md61 recovery
→ 52.1%

Distributed md62 recovery
→ 36.0%

2+2 md61 recovery — first member
→ 43.7%

2+2 md61 recovery — second member
→ 43.7%

2+2 md62 recovery — first member
→ 31.1%

2+2 md62 recovery — second member
→ 49.9%
```

These are observed intermediate progress points rather than fixed rebuild
times or performance guarantees.

---

# 51. Recovery Validation Matrix

| Scenario           | Component   | Failed Members | Rebuild Result | Final State         |
| ------------------ | ----------- | -------------: | -------------- | ------------------- |
| Single failure     | md61        |              1 | PASS           | `[4/4] [UUUU]`      |
| Two-member failure | md61        |              2 | PASS           | `[4/4] [UUUU]`      |
| Single failure     | md62        |              1 | PASS           | `[4/4] [UUUU]`      |
| Distributed 1+1    | md61 + md62 |          1 + 1 | PASS           | Both `[4/4] [UUUU]` |
| Maximum 2+2        | md61 + md62 |          2 + 2 | PASS           | Both `[4/4] [UUUU]` |

---

# 52. Recovery Validation Layers

Every recovery scenario was validated at multiple layers:

```text
Layer 1
Failed member identified

        ↓

Layer 2
Component RAID 6 state validated

        ↓

Layer 3
Replacement member added

        ↓

Layer 4
Rebuild progress monitored

        ↓

Layer 5
Component RAID 6 returned healthy

        ↓

Layer 6
Top-level md60 validated

        ↓

Layer 7
Filesystem validated

        ↓

Layer 8
Test files validated

        ↓

Layer 9
SHA-256 integrity validated
```

This prevented rebuild completion from being treated as the only success
criterion.

---

# 53. Important Engineering Observations

## 53.1 Recovery occurs inside the affected RAID 6 component

For example:

```text
loop10 failure
      ↓
md61 recovery
```

The top-level RAID 0 did not reconstruct the failed member.

---

## 53.2 Two failed members can be recovered sequentially

The `md61` two-member failure was recovered one member at a time.

This demonstrated that the component could remain operational while
being progressively restored.

---

## 53.3 Distributed failures require distributed recovery

When:

```text
md61 → degraded
md62 → degraded
```

both component arrays needed individual recovery.

---

## 53.4 The 2+2 scenario required four successful member recoveries

The critical sequence required:

```text
2 rebuilds in md61
+
2 rebuilds in md62
```

before full RAID60 redundancy was restored.

---

## 53.5 Final success required more than clean RAID state

The recovery was not considered complete until:

```text
RAID components healthy
+
md60 healthy
+
Filesystem accessible
+
Files readable
+
SHA-256 matched
```

---

# 54. Final Acceptance Criteria

The complete recovery and rebuild phase was accepted because:

```text
[✓] md61 single-member failure recovered

[✓] md61 two-member failure recovered

[✓] md62 single-member failure recovered

[✓] Distributed md61/md62 failures recovered

[✓] Maximum 2+2 failure recovered

[✓] All failed members removed successfully

[✓] All replacement members added successfully

[✓] All rebuild operations completed

[✓] md61 returned to [4/4] [UUUU]

[✓] md62 returned to [4/4] [UUUU]

[✓] md61 Failed Devices = 0

[✓] md62 Failed Devices = 0

[✓] md60 remained operational

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] Final SHA-256 values matched baseline
```

---

# 55. Final Recovery Result

```text
RECOVERY AND REBUILD RESULT: PASS
```

The RAID60 configuration successfully recovered all tested failure
scenarios.

Final component state:

```text
md61 → clean → [4/4] [UUUU]

md62 → clean → [4/4] [UUUU]
```

Final top-level state:

```text
md60 → clean / active
```

Final filesystem:

```text
/dev/md60
    ↓
/mnt/raid60
```

Final data integrity:

```text
SHA-256
   ↓
MATCH
```

---

# 56. Final Recovery Conclusion

The fresh RAID60 laboratory demonstrated successful member replacement
and RAID 6 reconstruction across all planned failure scenarios.

The most important recovery sequence was the maximum distributed failure:

```text
md61 → 2 failed
+
md62 → 2 failed
        ↓
4 failed members total
        ↓
Sequential member recovery
        ↓
md61 → [4/4] [UUUU]
        ↓
md62 → [4/4] [UUUU]
        ↓
md60 → active
        ↓
Filesystem → accessible
        ↓
Test files → readable
        ↓
SHA-256 → unchanged
```

The recovery testing therefore established practical evidence for:

```text
Single-member RAID 6 recovery
+
Two-member RAID 6 recovery
+
Distributed component recovery
+
Maximum 2+2 recovery
+
Filesystem continuity
+
Data-integrity preservation
```

The RAID60 recovery lifecycle was successfully completed before the
final healthy-state and cleanup stages.

