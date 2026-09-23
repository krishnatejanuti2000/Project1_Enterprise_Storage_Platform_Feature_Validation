# RAID 50 — Single Member Failure in md52

## 1. Test Objective

The objective of this test was to validate RAID50 behavior when a single
member of the second RAID 5 component array fails.

The component under test was:

```text
md52
RAID 5
```

Initial members:

```text
/dev/loop13
/dev/loop14
/dev/loop15
```

Failure injected:

```text
/dev/loop13
```

Expected behavior:

```text
loop13 failure
      ↓
md52 becomes degraded
      ↓
md51 remains healthy
      ↓
md50 remains accessible
      ↓
Filesystem remains accessible
      ↓
Test data remains readable
      ↓
Replacement member added
      ↓
Rebuild completes
      ↓
md52 returns to healthy state
```

---

# 2. Initial Healthy State

Before failure injection, the RAID50 environment was healthy.

Topology:

```text
                         md50
                        RAID 0
                     /         \
                    /           \
                 md51           md52
                RAID 5          RAID 5
               /  |  \         /  |  \
          loop10 loop11 loop12 loop13 loop14 loop15
```

The second component array was:

```text
md52
→ loop13
→ loop14
→ loop15
```

Healthy state:

```text
md52
[3/3] [UUU]
```

The first component remained:

```text
md51
[3/3] [UUU]
```

The top-level RAID50 device was active.

The filesystem was mounted at:

```text
/mnt/raid50
```

---

# 3. Failure Injection

The member failure was intentionally introduced using:

```bash
sudo mdadm --manage /dev/md52 --fail /dev/loop13
```

This marked `/dev/loop13` as failed within the second RAID 5 component.

---

# 4. RAID State After Failure

After failure injection, `md52` entered degraded mode.

The observed state was:

```text
md52
[3/2] [_UU]
```

This means:

```text
Expected members = 3
Active members   = 2
```

Member state:

```text
loop13 → failed
loop14 → active
loop15 → active
```

The first RAID 5 component remained healthy:

```text
md51
[3/3] [UUU]
```

The top-level `md50` remained active.

---

# 5. Failure-Domain Analysis

The failed member belonged to:

```text
md52
```

The failure distribution was therefore:

```text
md51 → 0 failed members
md52 → 1 failed member
```

Conceptually:

```text
                    md50
                  RAID 0
                 /       \
                /         \
             md51         md52
            RAID 5       RAID 5
             [UUU]         [_UU]
```

The failure was contained within the second RAID 5 component.

Because RAID 5 provides protection against one failed member, `md52`
remained operational in degraded mode.

---

# 6. Verify RAID Status

The RAID status was monitored using:

```bash
cat /proc/mdstat
```

The important state was:

```text
md51 → [3/3] [UUU]
md52 → [3/2] [_UU]
md50 → active
```

This confirmed:

```text
[✓] md52 degraded
[✓] md51 healthy
[✓] md50 still active
```

---

# 7. Detailed md52 Validation

The affected component was inspected using:

```bash
sudo mdadm --detail /dev/md52
```

The degraded state showed:

```text
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 1
Spare Devices    : 0
```

The failed member was:

```text
/dev/loop13
```

The surviving members were:

```text
/dev/loop14
/dev/loop15
```

This confirmed that the RAID 5 component was operating in its expected
single-member degraded state.

---

# 8. Top-Level RAID50 Validation

The parent RAID50 array was checked using:

```bash
sudo mdadm --detail /dev/md50
```

The top-level array remained active because:

```text
md51 → operational
md52 → degraded but operational
```

The RAID 0 layer did not provide recovery for the failed member.

Recovery capability came from the RAID 5 component `md52`.

---

# 9. Filesystem Accessibility Test

The filesystem remained mounted at:

```text
/mnt/raid50
```

The storage path remained:

```text
ext4
 ↓
/dev/md50
 ↓
RAID 0
 ↓
md51 + md52
 ↓
RAID 5 members
```

The filesystem remained accessible while `md52` was degraded.

---

# 10. Directory Validation

The test directory was checked:

```bash
ls -lh /mnt/raid50
```

The expected files remained available:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

This confirmed that the logical RAID50 filesystem remained accessible
after the single-member failure in `md52`.

---

# 11. File Read Validation

All test files were read:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

The files remained readable.

Therefore:

```text
Single member failure in md52
          ↓
md52 degraded
          ↓
md50 remains accessible
          ↓
Filesystem remains accessible
          ↓
Files remain readable
```

---

# 12. Data Integrity Validation After Failure

The healthy-state SHA-256 baseline was:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

Checksums were recalculated after the failure:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The resulting values remained unchanged from the healthy baseline.

Therefore:

```text
Baseline SHA-256
        =
Post-failure SHA-256
```

The tested data remained intact while `md52` was degraded.

---

# 13. Failed Member Removal

After validating the degraded state and data accessibility, the failed
member was removed:

```bash
sudo mdadm --manage /dev/md52 --remove /dev/loop13
```

The failed `loop13` member was no longer part of the active working set.

The RAID 5 component remained degraded while waiting for replacement.

---

# 14. Replacement Member Preparation

A new 1 GiB replacement image was prepared:

```text
replacement2.img
```

It was attached as:

```text
/dev/loop17
```

The replacement member was verified as a clean device before being added
to `md52`.

No pre-existing md RAID metadata was present on the replacement device.

---

# 15. Add Replacement Member

The replacement member was added using:

```bash
sudo mdadm --manage /dev/md52 --add /dev/loop17
```

This initiated recovery/rebuild of the failed RAID 5 member.

Recovery path:

```text
Failed loop13
      ↓
Replacement loop17
      ↓
RAID 5 reconstruction
      ↓
Rebuild
```

---

# 16. Rebuild Monitoring

Rebuild activity was monitored with:

```bash
cat /proc/mdstat
```

An observed intermediate rebuild state reached approximately:

```text
38.4%
```

The rebuild then continued until the replacement member was fully
reconstructed.

---

# 17. Rebuild Completion

After the rebuild finished, `md52` returned to:

```text
[3/3] [UUU]
```

The recovered member layout was:

```text
loop17 → role 0
loop14 → role 1
loop15 → role 2
```

The failed `loop13` member had therefore been replaced by `loop17`.

---

# 18. md52 Post-Rebuild Validation

The recovered component was inspected using:

```bash
sudo mdadm --detail /dev/md52
```

The final state showed:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed that the RAID 5 component had completely recovered.

---

# 19. Parent RAID50 Validation

The top-level RAID50 device was checked again:

```bash
sudo mdadm --detail /dev/md50
```

The hierarchy was fully operational:

```text
md50 → RAID 0 → active
md51 → RAID 5 → clean
md52 → RAID 5 → clean
```

The single-member failure in `md52` had therefore been completely
recovered.

---

# 20. Post-Rebuild Filesystem Validation

The filesystem remained available at:

```text
/mnt/raid50
```

The mount remained:

```text
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

This demonstrated filesystem continuity across:

```text
Failure
→ degraded RAID
→ member replacement
→ rebuild
→ healthy RAID state
```

---

# 21. Post-Rebuild Data Validation

The test files were read after recovery:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All three files remained readable.

The file contents remained consistent with the healthy baseline.

---

# 22. Post-Rebuild SHA-256 Validation

The checksums were recalculated after the rebuild:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The results matched the original healthy-state values:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
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

# 23. Test Result

The test passed.

### Failure handling

```text
[PASS] loop13 successfully marked failed
[PASS] md52 entered degraded state
[PASS] md51 remained healthy
[PASS] md50 remained active
```

### Data availability

```text
[PASS] Filesystem remained accessible
[PASS] Test files remained readable
```

### Recovery

```text
[PASS] Failed loop13 removed
[PASS] Replacement loop17 added
[PASS] Rebuild completed
[PASS] md52 returned to [3/3] [UUU]
[PASS] md52 returned to clean state
```

### Data integrity

```text
[PASS] SHA-256 unchanged after failure
[PASS] SHA-256 unchanged after rebuild
```

---

# 24. Failure and Recovery Flow

The complete tested sequence was:

```text
Healthy md52
[3/3] [UUU]
      |
      ↓
Fail loop13
      |
      ↓
md52
[3/2] [_UU]
      |
      ↓
Validate data access
      |
      ↓
Validate SHA-256
      |
      ↓
Remove loop13
      |
      ↓
Add replacement loop17
      |
      ↓
Rebuild
      |
      ↓
md52
[3/3] [UUU]
      |
      ↓
Validate mdadm state
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

This test demonstrated the same RAID50 failure principle on the second
component RAID 5 group.

### 1. The failure was isolated to md52

```text
md52 → degraded
md51 → healthy
```

This created a single component failure domain.

### 2. RAID50 remained accessible

Because `md52` still had two working members and therefore remained a
functional RAID 5 component, the top-level RAID50 device continued to
serve the filesystem.

### 3. Redundancy came from RAID 5

The outer RAID 0 layer did not provide redundancy.

The failed member was protected by the RAID 5 parity mechanism within
`md52`.

### 4. Data remained intact

The SHA-256 values remained identical during the degraded state and
after rebuild.

### 5. Rebuild restored redundancy

The replacement member:

```text
/dev/loop17
```

restored `md52` to:

```text
[3/3] [UUU]
```

with:

```text
Failed Devices = 0
```

---

# 26. Final Acceptance Criteria

The test was accepted because:

```text
[✓] Single-member failure injected into md52

[✓] Correct RAID 5 component entered degraded state

[✓] md51 remained healthy

[✓] md50 remained active

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] SHA-256 remained unchanged after failure

[✓] Failed loop13 removed successfully

[✓] Replacement loop17 added successfully

[✓] Rebuild completed

[✓] md52 returned to [3/3] [UUU]

[✓] md52 returned to clean state

[✓] Final SHA-256 values matched baseline
```

---

# 27. Final Test Conclusion

The RAID50 single-member failure test for `md52` passed successfully.

The tested behavior was:

```text
One member failure in md52
        ↓
md52 degraded
        ↓
md51 remained healthy
        ↓
RAID50 remained accessible
        ↓
Data remained readable
        ↓
Data integrity remained unchanged
        ↓
Replacement loop17 added
        ↓
RAID 5 rebuild completed
        ↓
md52 returned healthy
```

This validates that the tested RAID50 configuration can survive a
single-member failure in the second component RAID 5 group and restore
the failed member without loss of the tested filesystem data.

