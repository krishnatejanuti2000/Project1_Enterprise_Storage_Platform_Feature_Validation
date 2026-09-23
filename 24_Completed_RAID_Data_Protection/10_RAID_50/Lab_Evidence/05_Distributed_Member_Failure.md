# RAID 50 — Distributed Member Failure

## 1. Test Objective

The objective of this test was to validate RAID50 behavior when one member
fails in each underlying RAID 5 component.

This scenario is important because RAID50 contains multiple independent
RAID 5 failure domains.

The tested topology was:

```text id="s0mc2j"
                         RAID 50
                            |
                           md50
                         RAID 0
                       /       \
                      /         \
                   md51         md52
                  RAID 5       RAID 5
```

At the time of failure injection, the active members were:

```text id="7l5j79"
md51:
loop16
loop11
loop12

md52:
loop17
loop14
loop15
```

The distributed failure was:

```text id="2g4h9q"
md51 → loop16 failed
md52 → loop17 failed
```

Expected behavior:

```text id="2tkjyi"
One failure in md51
        +
One failure in md52
        ↓
Both RAID 5 components degraded
        ↓
Both remain operational
        ↓
md50 remains accessible
        ↓
Filesystem remains accessible
        ↓
Test data remains readable
        ↓
Rebuild both components
        ↓
Restore full redundancy
```

---

# 2. Initial Healthy State

Before injecting the distributed failures, both component RAID 5 arrays
had been restored to healthy operation after their previous individual
failure/recovery tests.

The active configuration was:

```text id="e11b3a"
md51
→ loop16
→ loop11
→ loop12

md52
→ loop17
→ loop14
→ loop15
```

Both component arrays were healthy:

```text id="mnbm8r"
md51 → [3/3] [UUU]
md52 → [3/3] [UUU]
```

The top-level array was:

```text id="5h8w4s"
md50 → active
```

The filesystem remained mounted at:

```text id="c7owbv"
/mnt/raid50
```

---

# 3. Failure Distribution

The test intentionally introduced one member failure into each RAID 5
component.

Failure pattern:

```text id="a4v8zi"
md51:
loop16 ❌
loop11 ✅
loop12 ✅

md52:
loop17 ❌
loop14 ✅
loop15 ✅
```

Therefore the RAID50 failure distribution was:

```text id="siqfya"
md51 → 1 failed member
md52 → 1 failed member
```

This is a distributed failure rather than two failures inside the same
component RAID 5 group.

---

# 4. Inject First Distributed Failure

The first failure was introduced in `md51`:

```bash id="z5tfcz"
sudo mdadm --manage /dev/md51 --fail /dev/loop16
```

After the command completed, `md51` entered degraded mode.

Expected member state:

```text id="5k60mh"
loop16 → failed
loop11 → active
loop12 → active
```

---

# 5. Inject Second Distributed Failure

The second failure was introduced in `md52`:

```bash id="f981a0"
sudo mdadm --manage /dev/md52 --fail /dev/loop17
```

The resulting failure distribution was:

```text id="y6qq1i"
md51 → 1 failed member
md52 → 1 failed member
```

Both RAID 5 components were therefore operating in degraded mode at the
same time.

---

# 6. RAID State After Both Failures

The resulting `/proc/mdstat` state showed both component arrays degraded.

Conceptually:

```text id="n24g5u"
md51 → [3/2] [_UU]
md52 → [3/2] [_UU]
md50 → active
```

This is the key observation of the distributed failure scenario.

Both component RAID 5 arrays remained operational despite simultaneously
losing one member each.

---

# 7. Failure-Domain Analysis

The failure distribution was:

```text id="i5uynn"
         RAID 50
            |
       RAID 0 layer
        /       \
       /         \
    md51         md52
  1 failed     1 failed
```

Each component independently remained within RAID 5's single-member
failure tolerance.

Therefore:

```text id="7cy6ms"
md51 → degraded but operational
md52 → degraded but operational
md50 → remains accessible
```

This demonstrates why the component RAID failure domains must be
considered separately.

---

# 8. Detailed md51 Validation

The first affected component was checked with:

```bash id="2ws0gv"
sudo mdadm --detail /dev/md51
```

The degraded condition showed:

```text id="6vra82"
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text id="tb0fsc"
/dev/loop16
```

The surviving members were:

```text id="hw35t7"
/dev/loop11
/dev/loop12
```

---

# 9. Detailed md52 Validation

The second affected component was checked with:

```bash id="6www81"
sudo mdadm --detail /dev/md52
```

The degraded condition showed:

```text id="0f2h50"
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text id="lk22c3"
/dev/loop17
```

The surviving members were:

```text id="8f59nw"
/dev/loop14
/dev/loop15
```

---

# 10. Top-Level RAID50 Validation

The parent RAID50 device was inspected:

```bash id="r9s5ei"
sudo mdadm --detail /dev/md50
```

The top-level RAID 0 remained active.

The hierarchy was:

```text id="04b4xm"
md50
 ├── md51 → degraded
 └── md52 → degraded
```

Both component RAID 5 arrays remained available, so the logical RAID50
device continued serving the filesystem.

---

# 11. Filesystem Accessibility

The filesystem remained available at:

```text id="dsd7sy"
/mnt/raid50
```

The mount remained:

```text id="ebz3gp"
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

This demonstrated that the top-level filesystem remained accessible while:

```text id="wz1ew8"
md51 → degraded
md52 → degraded
```

simultaneously.

---

# 12. Directory Validation

The RAID50 directory was checked:

```bash id="knl0eb"
ls -lh /mnt/raid50
```

The test files remained available:

```text id="1l9k0f"
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

No test data disappeared as a result of the distributed failures.

---

# 13. File Read Validation

The test files were read while both RAID 5 components were degraded:

```bash id="uj5v9l"
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

The contents remained readable.

This established:

```text id="d1ozxh"
md51 degraded
+
md52 degraded
        ↓
RAID50 still accessible
        ↓
Files still readable
```

---

# 14. Data Integrity Validation During Distributed Failure

The test files were hashed while both component arrays were degraded:

```bash id="ri9m0w"
sha256sum /mnt/raid50/testfile_*.txt
```

The resulting values matched the healthy-state baseline:

```text id="jzh8d5"
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

Therefore:

```text id="v4h1s2"
Healthy baseline
        =
2-component degraded state
```

for all tested files.

This provided direct data-integrity evidence while both RAID 5 components
were simultaneously degraded.

---

# 15. Recovery Strategy

Because both component arrays were degraded, recovery was performed in a
controlled sequential manner.

The recovery order was:

```text id="2l50cc"
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

This made each recovery step independently observable.

---

# 16. Recover md51 — Remove Failed Member

The failed `loop16` member was removed from `md51`:

```bash id="9wgx37"
sudo mdadm --manage /dev/md51 --remove /dev/loop16
```

The component remained degraded while awaiting the replacement member.

---

# 17. Prepare md51 Replacement

The replacement image was:

```text id="8p5x1e"
replacement3.img
```

It was attached as:

```text id="g2f32n"
/dev/loop18
```

The replacement device was checked before being added to the RAID
component.

It was clean and contained no existing md RAID metadata.

---

# 18. Add md51 Replacement

The replacement member was added:

```bash id="y9y0mi"
sudo mdadm --manage /dev/md51 --add /dev/loop18
```

This initiated reconstruction of the failed RAID 5 member.

Recovery path:

```text id="q3f75o"
loop16 → failed
      ↓
removed
      ↓
loop18 → replacement
      ↓
RAID 5 reconstruction
      ↓
rebuild
```

---

# 19. md51 Rebuild Completion

The rebuild was monitored using:

```bash id="9ulq81"
cat /proc/mdstat
```

After reconstruction completed, `md51` returned to:

```text id="0d94tw"
[3/3] [UUU]
```

The recovered member layout was:

```text id="2oe0a6"
loop18 → role 0
loop11 → role 1
loop12 → role 2
```

`md51` had therefore returned to full member count.

---

# 20. md51 Post-Rebuild Validation

The recovered component was checked using:

```bash id="kge2x1"
sudo mdadm --detail /dev/md51
```

The expected final state was:

```text id="7m12ny"
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

The first RAID 5 failure domain was therefore fully restored.

---

# 21. Recover md52 — Remove Failed Member

The failed `loop17` member was then removed from `md52`:

```bash id="im4d5p"
sudo mdadm --manage /dev/md52 --remove /dev/loop17
```

The second component remained degraded while waiting for its
replacement.

---

# 22. Prepare md52 Replacement

The replacement image was:

```text id="u2eqy8"
replacement4.img
```

It was attached as:

```text id="vxlg7p"
/dev/loop19
```

The replacement device was verified as clean before being added.

---

# 23. Add md52 Replacement

The replacement member was added:

```bash id="8quqih"
sudo mdadm --manage /dev/md52 --add /dev/loop19
```

This initiated RAID 5 reconstruction for the second component.

Recovery path:

```text id="g8g9sn"
loop17 → failed
      ↓
removed
      ↓
loop19 → replacement
      ↓
RAID 5 reconstruction
      ↓
rebuild
```

---

# 24. md52 Rebuild Completion

Rebuild status was monitored using:

```bash id="q3v59x"
cat /proc/mdstat
```

After the rebuild completed, `md52` returned to:

```text id="jlsv1d"
[3/3] [UUU]
```

The recovered member layout was:

```text id="hq7w1j"
loop19 → role 0
loop14 → role 1
loop15 → role 2
```

---

# 25. md52 Post-Rebuild Validation

The recovered component was inspected using:

```bash id="43kujd"
sudo mdadm --detail /dev/md52
```

The final state showed:

```text id="q42zxp"
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

The second RAID 5 failure domain was therefore fully restored.

---

# 26. Top-Level RAID50 Recovery Validation

After both component arrays were recovered, the parent device was
validated:

```bash id="65ylja"
sudo mdadm --detail /dev/md50
```

The complete hierarchy was again healthy:

```text id="h6guv9"
md50 → RAID 0 → active
 |
 +-- md51 → RAID 5 → clean
 |
 +-- md52 → RAID 5 → clean
```

The distributed failure had therefore been completely recovered.

---

# 27. Filesystem Validation After Recovery

The filesystem was checked again:

```bash id="apvp2p"
mount | grep raid50
```

The expected mount remained:

```text id="60lyzs"
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

The logical filesystem remained available throughout the failure and
recovery process.

---

# 28. Post-Recovery File Validation

The test files were read after both rebuilds:

```bash id="78q4vy"
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All three files remained readable.

This verified filesystem and application-level data accessibility after
full recovery.

---

# 29. Post-Recovery SHA-256 Validation

The checksums were recalculated:

```bash id="lx39i9"
sha256sum /mnt/raid50/testfile_*.txt
```

The final values remained equal to the original baseline:

```text id="4r5o75"
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

Therefore:

```text id="y0qq95"
Healthy baseline
        =
During distributed degradation
        =
After md51 rebuild
        =
After md52 rebuild
```

for all tested data.

---

# 30. Complete Failure and Recovery Flow

The complete tested sequence was:

```text id="3gnc8g"
Healthy RAID50
      |
      ↓
Fail md51 loop16
      |
      ↓
md51 degraded
      |
      ↓
Fail md52 loop17
      |
      ↓
md51 degraded + md52 degraded
      |
      ↓
Validate md50
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
Remove md51 loop16
      |
      ↓
Add md51 loop18
      |
      ↓
Rebuild md51
      |
      ↓
md51 [3/3] [UUU]
      |
      ↓
Remove md52 loop17
      |
      ↓
Add md52 loop19
      |
      ↓
Rebuild md52
      |
      ↓
md52 [3/3] [UUU]
      |
      ↓
Validate md50
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

# 31. Engineering Observations

## 1. Both RAID 5 components can be degraded simultaneously

The tested configuration successfully entered:

```text id="x9r35v"
md51 → degraded
md52 → degraded
```

without immediate loss of the top-level filesystem.

---

## 2. Failure distribution matters

The failure pattern was:

```text id="v6sbe6"
1 failure in md51
+
1 failure in md52
```

Rather than:

```text id="1x4l3n"
2 failures in md51
+
0 failures in md52
```

The two scenarios represent different RAID50 failure distributions and
must be analyzed independently.

---

## 3. RAID50 remained accessible

Both RAID 5 component arrays remained operational while degraded.

Therefore:

```text id="3w81cb"
md51 degraded
+
md52 degraded
        ↓
md50 active
        ↓
Filesystem accessible
```

---

## 4. Data integrity remained intact

The SHA-256 values were unchanged during the distributed failure state
and after both rebuilds.

---

## 5. Recovery restored both failure domains

The final component states were:

```text id="041vve"
md51 → [3/3] [UUU]
md52 → [3/3] [UUU]
```

with:

```text id="e83sr9"
Failed Devices = 0
```

for both component arrays.

---

# 32. Final Acceptance Criteria

The distributed failure test was accepted because:

```text id="3r8vva"
[✓] md51 member failure injected

[✓] md52 member failure injected

[✓] Both component RAID 5 arrays entered degraded state

[✓] md50 remained active

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] SHA-256 values matched baseline during distributed degradation

[✓] md51 failed member removed

[✓] md51 replacement loop18 added

[✓] md51 rebuild completed

[✓] md51 returned to [3/3] [UUU]

[✓] md52 failed member removed

[✓] md52 replacement loop19 added

[✓] md52 rebuild completed

[✓] md52 returned to [3/3] [UUU]

[✓] md50 remained operational

[✓] Final filesystem validation passed

[✓] Final SHA-256 validation matched baseline
```

---

# 33. Test Result

```text id="glndy9"
TEST RESULT: PASS
```

The RAID50 configuration successfully tolerated the tested distributed
failure pattern:

```text id="if7u4v"
1 failed member in md51
+
1 failed member in md52
```

while maintaining:

```text id="q3u0cx"
RAID50 availability
+
Filesystem accessibility
+
File readability
+
Data integrity
```

Both failed members were subsequently replaced and rebuilt successfully.

---

# 34. Final Test Conclusion

The distributed member-failure test demonstrated that the tested RAID50
configuration can remain operational when one member fails in each
underlying RAID 5 component.

The complete tested behavior was:

```text id="tvpa89"
One member fails in md51
        +
One member fails in md52
        ↓
Both component arrays degraded
        ↓
RAID50 remains accessible
        ↓
Filesystem remains accessible
        ↓
Data remains readable
        ↓
Checksums remain unchanged
        ↓
Recover md51
        ↓
Recover md52
        ↓
Both RAID 5 components return healthy
        ↓
Top-level RAID50 remains operational
        ↓
Data integrity verified
```

This test provides practical evidence that the RAID50 failure domains
operate independently for the tested single-member distributed failure
pattern and that the configuration can recover both affected members
without loss of the tested data.
::

