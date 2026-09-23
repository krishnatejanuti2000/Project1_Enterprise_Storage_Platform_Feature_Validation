# RAID 50 — Recovery and Rebuild Evidence

## 1. Recovery Objective

The objective of this phase was to validate recovery of failed RAID 5
members within the RAID50 hierarchy.

Recovery was validated after:

```text
Single-member failure in md51
+
Single-member failure in md52
+
Distributed failure:
one failed member in md51
+
one failed member in md52
```

The recovery workflow was:

```text
Failed member
      ↓
Verify degraded state
      ↓
Remove failed member
      ↓
Prepare clean replacement
      ↓
Add replacement
      ↓
Monitor rebuild
      ↓
Verify component RAID
      ↓
Verify parent RAID50
      ↓
Verify filesystem
      ↓
Verify data integrity
```

---

# 2. Recovery Architecture

The RAID50 hierarchy was:

```text
                         md50
                        RAID 0
                     /         \
                    /           \
                 md51           md52
                RAID 5          RAID 5
```

Recovery occurred inside the affected RAID 5 component.

The top-level RAID 0 layer did not reconstruct a failed RAID 5 member.

Therefore:

```text
Failed md51 member
      ↓
md51 RAID 5 rebuild

Failed md52 member
      ↓
md52 RAID 5 rebuild
```

---

# 3. Recovery Scenario 1 — md51 Single-Member Failure

The first recovery scenario followed the failure of:

```text
/dev/loop10
```

inside:

```text
md51
```

The failure state was:

```text
md51
[3/2] [_UU]
```

The surviving members were:

```text
loop11
loop12
```

The parent `md50` remained active.

---

# 4. Remove Failed md51 Member

The failed member was removed:

```bash
sudo mdadm --manage /dev/md51 --remove /dev/loop10
```

`md51` remained degraded and was prepared to accept a replacement member.

---

# 5. Prepare md51 Replacement

The first replacement image was:

```text
replacement1.img
```

It was attached as:

```text
/dev/loop16
```

The replacement device was verified before use and contained no existing
md RAID metadata.

---

# 6. Add md51 Replacement

The replacement member was added:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loop16
```

This initiated RAID 5 recovery.

Recovery path:

```text
loop10
  ↓
failed

loop16
  ↓
replacement

md51
  ↓
RAID 5 reconstruction
  ↓
rebuild
```

---

# 7. md51 Rebuild Monitoring

Rebuild activity was monitored using:

```bash
cat /proc/mdstat
```

Observed progress included:

```text
37.4%
```

with an observed recovery rate of approximately:

```text
130824K/sec
```

A later observation showed:

```text
75.6%
```

with a recovery rate of approximately:

```text
113176K/sec
```

The rebuild continued until the replacement member was fully
reconstructed.

---

# 8. md51 Rebuild Completion

After recovery completed, `md51` returned to:

```text
[3/3] [UUU]
```

The recovered member layout was:

```text
loop16 → role 0
loop11 → role 1
loop12 → role 2
```

The failed `loop10` member had therefore been successfully replaced.

---

# 9. md51 Post-Rebuild Validation

The recovered component was inspected using:

```bash
sudo mdadm --detail /dev/md51
```

The final state was:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed complete recovery of the first RAID 5 component.

---

# 10. Recovery Scenario 2 — md52 Single-Member Failure

The second individual failure occurred in:

```text
md52
```

The failed member was:

```text
/dev/loop13
```

The failure state was:

```text
md52
[3/2] [_UU]
```

The surviving members were:

```text
loop14
loop15
```

---

# 11. Remove Failed md52 Member

The failed member was removed:

```bash
sudo mdadm --manage /dev/md52 --remove /dev/loop13
```

The component remained degraded while waiting for replacement.

---

# 12. Prepare md52 Replacement

The replacement image was:

```text
replacement2.img
```

It was attached as:

```text
/dev/loop17
```

The replacement was verified as a clean device before use.

---

# 13. Add md52 Replacement

The replacement member was added:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loop17
```

This initiated RAID 5 reconstruction.

Recovery path:

```text
loop13
  ↓
failed

loop17
  ↓
replacement

md52
  ↓
RAID 5 reconstruction
  ↓
rebuild
```

---

# 14. md52 Rebuild Monitoring

Rebuild progress was monitored using:

```bash
cat /proc/mdstat
```

An observed recovery point reached:

```text
38.4%
```

The rebuild continued until all expected RAID 5 members were restored.

---

# 15. md52 Rebuild Completion

After recovery completed, `md52` returned to:

```text
[3/3] [UUU]
```

The recovered member layout was:

```text
loop17 → role 0
loop14 → role 1
loop15 → role 2
```

The failed `loop13` member had therefore been replaced successfully.

---

# 16. md52 Post-Rebuild Validation

The recovered component was inspected:

```bash
sudo mdadm --detail /dev/md52
```

The final state was:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed complete recovery of the second RAID 5 component.

---

# 17. Distributed Failure Recovery — Initial State

After the individual failure/rebuild tests, both component arrays were
healthy again.

The active members immediately before the distributed failure were:

```text
md51:
loop16
loop11
loop12

md52:
loop17
loop14
loop15
```

The distributed failures were then injected:

```text
md51 → loop16 failed
md52 → loop17 failed
```

The resulting state was:

```text
md51 → [3/2] [_UU]
md52 → [3/2] [_UU]
```

The top-level `md50` remained active.

---

# 18. Distributed Failure Recovery Strategy

Because both RAID 5 components were degraded simultaneously, recovery
was performed sequentially.

The recovery strategy was:

```text
Recover md51
      ↓
Verify md51
      ↓
Recover md52
      ↓
Verify md52
      ↓
Verify md50
      ↓
Verify filesystem
      ↓
Verify data integrity
```

This allowed each component rebuild to be independently validated.

---

# 19. Recover Distributed Failure — md51

The failed member in `md51` was:

```text
/dev/loop16
```

It was removed:

```bash
sudo mdadm --manage /dev/md51 --remove /dev/loop16
```

The replacement image was:

```text
replacement3.img
```

and was attached as:

```text
/dev/loop18
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loop18
```

This initiated the md51 rebuild.

---

# 20. md51 Distributed-Recovery Rebuild

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

The recovery progressed through the rebuild operation.

After completion:

```text
md51
[3/3] [UUU]
```

The recovered member configuration was:

```text
loop18 → role 0
loop11 → role 1
loop12 → role 2
```

The failed `loop16` member was therefore replaced by `loop18`.

---

# 21. md51 Distributed-Recovery Validation

The recovered component was checked:

```bash
sudo mdadm --detail /dev/md51
```

The component returned to a clean state:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
```

The first failure domain was fully restored before recovering `md52`.

---

# 22. Recover Distributed Failure — md52

The remaining failed member was:

```text
/dev/loop17
```

inside:

```text
md52
```

The failed member was removed:

```bash
sudo mdadm --manage /dev/md52 --remove /dev/loop17
```

The replacement image was:

```text
replacement4.img
```

and was attached as:

```text
/dev/loop19
```

The replacement was added:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loop19
```

This initiated the second component rebuild.

---

# 23. md52 Distributed-Recovery Rebuild

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

The rebuild continued until the failed member was completely
reconstructed.

After completion:

```text
md52
[3/3] [UUU]
```

The recovered member configuration was:

```text
loop19 → role 0
loop14 → role 1
loop15 → role 2
```

The failed `loop17` member was therefore replaced by `loop19`.

---

# 24. md52 Distributed-Recovery Validation

The recovered component was inspected:

```bash
sudo mdadm --detail /dev/md52
```

The final state was:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
```

The second failure domain was fully restored.

---

# 25. Final md50 Validation

After both component RAID 5 arrays returned to a healthy state, the
top-level array was checked:

```bash
sudo mdadm --detail /dev/md50
```

The hierarchy was:

```text
md50
RAID 0
 |
 +-- md51 → RAID 5 → clean
 |
 +-- md52 → RAID 5 → clean
```

The top-level RAID50 configuration was operational again.

---

# 26. Final `/proc/mdstat` Validation

The final RAID state was checked with:

```bash
cat /proc/mdstat
```

The final hierarchy showed:

```text
md50 : active raid0 md52[1] md51[0]
      4182016 blocks super 1.2 512k chunks

md52 : active raid5
      [3/3] [UUU]

md51 : active raid5
      [3/3] [UUU]

unused devices: <none>
```

The important result was:

```text
md51 → [3/3] [UUU]
md52 → [3/3] [UUU]
```

indicating that both component RAID 5 arrays had recovered fully.

---

# 27. Final Filesystem Validation

The filesystem was checked:

```bash
mount | grep raid50
```

The final mount was:

```text
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

This confirmed that the filesystem remained available after the complete
failure and recovery sequence.

---

# 28. Final Capacity Validation

The filesystem capacity was verified:

```bash
df -h /mnt/raid50
```

The final result was:

```text
/dev/md50  3.9G  1.1M  3.7G  1% /mnt/raid50
```

The logical RAID50 filesystem therefore remained usable after complete
recovery.

---

# 29. Final File Accessibility Validation

The test files were checked:

```bash
ls -lh /mnt/raid50
```

All three test files remained available:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

The files were then read:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All files remained readable after the recovery sequence.

---

# 30. Final Data Integrity Validation

The SHA-256 checksums were recalculated:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

Final values:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

These matched the healthy-state baseline.

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

# 31. Replacement Summary

The fresh RAID50 final run used the following replacement sequence:

| Failed Member | RAID Component | Replacement      | Replacement Loop |
| ------------- | -------------- | ---------------- | ---------------- |
| loop10        | md51           | replacement1.img | loop16           |
| loop13        | md52           | replacement2.img | loop17           |
| loop16        | md51           | replacement3.img | loop18           |
| loop17        | md52           | replacement4.img | loop19           |

The final active members after the distributed recovery were:

```text
md51:
loop18
loop11
loop12

md52:
loop19
loop14
loop15
```

---

# 32. Recovery Sequence Summary

The complete fresh recovery sequence was:

```text
Single md51 failure
    loop10 ❌
       ↓
    loop16
       ↓
    rebuild
       ↓
md51 [3/3] [UUU]


Single md52 failure
    loop13 ❌
       ↓
    loop17
       ↓
    rebuild
       ↓
md52 [3/3] [UUU]


Distributed failure

md51:
    loop16 ❌

md52:
    loop17 ❌

       ↓

Recover md51:
    loop18 replacement
       ↓
    rebuild
       ↓
    md51 [3/3] [UUU]

       ↓

Recover md52:
    loop19 replacement
       ↓
    rebuild
       ↓
    md52 [3/3] [UUU]

       ↓

md50 operational
       ↓
filesystem accessible
       ↓
files readable
       ↓
SHA-256 matches baseline
```

---

# 33. Recovery Validation Matrix

| Validation                 | md51 Single Failure | md52 Single Failure | Distributed Failure |
| -------------------------- | ------------------- | ------------------- | ------------------- |
| Failure injected           | PASS                | PASS                | PASS                |
| Component degraded         | PASS                | PASS                | PASS                |
| Parent md50 active         | PASS                | PASS                | PASS                |
| Filesystem accessible      | PASS                | PASS                | PASS                |
| Test files readable        | PASS                | PASS                | PASS                |
| SHA-256 preserved          | PASS                | PASS                | PASS                |
| Failed member removed      | PASS                | PASS                | PASS                |
| Replacement added          | PASS                | PASS                | PASS                |
| Rebuild completed          | PASS                | PASS                | PASS                |
| Component returned healthy | PASS                | PASS                | PASS                |
| Final data integrity       | PASS                | PASS                | PASS                |

---

# 34. Important Engineering Observations

## 34.1 Replacement occurs at the component RAID level

The failed member is repaired inside the appropriate RAID 5 component.

For example:

```text
loop10 failure
      ↓
md51 recovery
```

not:

```text
loop10 failure
      ↓
md50 RAID 0 recovery
```

---

## 34.2 Rebuild restores redundancy

A degraded RAID 5 array has reduced protection.

The objective of the rebuild is:

```text
Degraded
   ↓
Replacement
   ↓
Reconstruction
   ↓
Healthy
```

---

## 34.3 Distributed recovery must account for both components

When both `md51` and `md52` are degraded:

```text
md51 → degraded
md52 → degraded
```

the recovery process must restore both independent failure domains.

---

## 34.4 Data integrity should be validated after recovery

A healthy RAID state alone is not sufficient.

The final validation included:

```text
RAID state
+
Filesystem state
+
File accessibility
+
SHA-256 integrity
```

---

# 35. Final Acceptance Criteria

The recovery and rebuild phase was accepted because:

```text
[✓] md51 single-member failure recovered

[✓] md52 single-member failure recovered

[✓] md51 distributed failure recovered

[✓] md52 distributed failure recovered

[✓] Failed members successfully removed

[✓] Replacement members successfully added

[✓] Rebuilds completed

[✓] md51 returned to [3/3] [UUU]

[✓] md52 returned to [3/3] [UUU]

[✓] md50 remained operational

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] Final SHA-256 values matched baseline

[✓] Failed Devices = 0 on recovered components

[✓] Final RAID hierarchy returned to healthy state
```

---

# 36. Final Result

```text
RECOVERY RESULT: PASS
```

All tested RAID50 failure/recovery scenarios were successfully
recovered.

The final healthy hierarchy was:

```text
                         md50
                        RAID 0
                     /         \
                    /           \
                 md51           md52
                RAID 5          RAID 5
                [UUU]           [UUU]
```

Final active members:

```text
md51:
loop18
loop11
loop12

md52:
loop19
loop14
loop15
```

Final condition:

```text
md51 → clean
md52 → clean
md50 → active
Filesystem → accessible
Data → readable
SHA-256 → unchanged
```

---

# 37. Final Recovery Conclusion

The RAID50 recovery testing demonstrated that the tested Linux
configuration can recover failed members inside its RAID 5 component
arrays while preserving access to the RAID50 filesystem.

The complete recovery behavior was:

```text
Member failure
      ↓
Component RAID degraded
      ↓
RAID50 remains accessible
      ↓
Failed member removed
      ↓
Clean replacement added
      ↓
RAID 5 reconstruction
      ↓
Component returns to [3/3] [UUU]
      ↓
Top-level RAID50 validated
      ↓
Filesystem validated
      ↓
Data validated
      ↓
SHA-256 validated
      ↓
Recovery PASS
```

The recovery evidence establishes that the fresh final RAID50 run
successfully exercised member replacement, RAID 5 reconstruction,
distributed component recovery, filesystem continuity, and data
integrity validation.

