# RAID 50 — Single Member Failure in md51

## 1. Test Objective

The objective of this test was to validate RAID50 behavior when a single
member of the first RAID 5 component array fails.

The component under test was:

```text
md51
RAID 5
```

Initial members:

```text
/dev/loop10
/dev/loop11
/dev/loop12
```

Failure injected:

```text
/dev/loop10
```

Expected behavior:

```text
loop10 failure
      ↓
md51 becomes degraded
      ↓
md52 remains healthy
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
md51 returns to healthy state
```

---

# 2. Initial Healthy State

Before failure injection, the RAID50 configuration was healthy.

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

The first component array was:

```text
md51
→ loop10
→ loop11
→ loop12
```

Healthy state:

```text
md51
[3/3] [UUU]
```

The second component remained:

```text
md52
[3/3] [UUU]
```

The top-level array was also active.

---

# 3. Failure Injection

The member failure was intentionally injected using:

```bash
sudo mdadm --manage /dev/md51 --fail /dev/loop10
```

This marked `/dev/loop10` as failed within the RAID 5 component.

The failure was therefore introduced at the component RAID level rather
than by modifying the top-level RAID 0 device directly.

---

# 4. md51 Degraded State

After the failure, `/proc/mdstat` showed:

```text
md51
[3/2] [_UU]
```

This indicates:

```text
Expected members = 3
Active members   = 2

loop10 → failed
loop11 → active
loop12 → active
```

The affected RAID 5 component was therefore operating in degraded mode.

The second component remained healthy:

```text
md52
[3/3] [UUU]
```

The top-level `md50` remained active.

---

# 5. md51 Detailed Validation

The component was then inspected using:

```bash
sudo mdadm --detail /dev/md51
```

The degraded state showed:

```text
State            : clean, degraded
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 1
Spare Devices    : 0
```

This confirmed that:

```text
md51
→ one failed member
→ two working members
→ RAID 5 still operational
```

The failed member was:

```text
/dev/loop10
```

---

# 6. Failure-Domain Analysis

The failed member belonged to:

```text
md51
```

and not:

```text
md52
```

Therefore:

```text
md51 → degraded
md52 → healthy
md50 → active
```

The failure was contained within a single RAID 5 component.

This is the key RAID50 failure-domain observation for this scenario.

---

# 7. Top-Level RAID50 Validation

After the member failure, the parent array was checked:

```bash
sudo mdadm --detail /dev/md50
```

The top-level RAID 0 remained active with both component arrays available:

```text
md51 → degraded but operational
md52 → healthy
```

Therefore the RAID50 logical device remained accessible.

The outer RAID 0 layer did not perform any redundancy operation.

The redundancy for the failed member was provided by the RAID 5 component
`md51`.

---

# 8. Filesystem Accessibility Test

The RAID50 filesystem was still mounted at:

```text
/mnt/raid50
```

The mount was validated.

Expected storage path:

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

The filesystem remained accessible after the single-member failure.

---

# 9. Directory Validation

The RAID50 test directory was checked:

```bash
ls -lh /mnt/raid50
```

The expected test files remained available:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

This confirmed that the filesystem remained accessible while `md51` was
degraded.

---

# 10. File Read Validation

The test data was read after the failure.

Example:

```bash
cat /mnt/raid50/testfile_01.txt
```

and:

```bash
cat /mnt/raid50/testfile_02.txt
```

and:

```bash
cat /mnt/raid50/testfile_03.txt
```

All test files remained readable.

This demonstrated:

```text
Single member failure
        ↓
md51 degraded
        ↓
RAID50 remains accessible
        ↓
Files remain readable
```

---

# 11. Data Integrity Validation After Failure

The healthy-state SHA-256 values were:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

After injecting the `loop10` failure, the files were hashed again:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The resulting SHA-256 values remained unchanged from the healthy
baseline.

Therefore:

```text
Baseline checksum
        =
Post-failure checksum
```

The tested data remained intact while `md51` was degraded.

---

# 12. Failed Member Removal

After confirming the degraded state and data accessibility, the failed
member was removed from the component RAID 5 array.

Command:

```bash
sudo mdadm --manage /dev/md51 --remove /dev/loop10
```

The member was no longer an active working member of `md51`.

At this stage the component remained degraded and was waiting for a
replacement member.

---

# 13. Replacement Member Preparation

A new 1 GiB replacement image was created for the failed member.

The replacement was:

```text
replacement1.img
```

It was attached as:

```text
/dev/loop16
```

The replacement device was checked before adding it to the RAID array.

The new member contained no existing md RAID metadata.

This ensured that the replacement device was clean before use.

---

# 14. Add Replacement Member

The replacement member was added to `md51`:

```bash
sudo mdadm --manage /dev/md51 --add /dev/loop16
```

The RAID 5 component then entered recovery/rebuild activity.

Logical recovery path:

```text
Failed loop10
      ↓
Replacement loop16
      ↓
RAID 5 reconstruction
      ↓
Rebuild
```

---

# 15. Rebuild Monitoring

The rebuild was monitored using:

```bash
cat /proc/mdstat
```

The rebuild progressed through the recovery operation.

An observed intermediate state was approximately:

```text
37.4%
```

with a recorded recovery rate of approximately:

```text
130824K/sec
```

Another observed progress point was approximately:

```text
75.6%
```

with a recovery rate of approximately:

```text
113176K/sec
```

The exact percentage and rate varied during the rebuild as the operation
progressed.

---

# 16. Rebuild Completion

After recovery completed, `/proc/mdstat` showed:

```text
md51
[3/3] [UUU]
```

This indicated that all three expected RAID 5 members were operational
again.

The member layout after recovery was:

```text
loop16 → role 0
loop11 → role 1
loop12 → role 2
```

The failed `loop10` member had been replaced by `loop16`.

---

# 17. md51 Post-Rebuild Validation

The recovered component was inspected using:

```bash
sudo mdadm --detail /dev/md51
```

Expected final state:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

This confirmed complete restoration of the RAID 5 component.

---

# 18. Parent RAID50 Validation

After `md51` recovered, the parent array was checked again:

```bash
sudo mdadm --detail /dev/md50
```

The RAID50 hierarchy was fully operational:

```text
md50 → RAID 0 → clean/active
md51 → RAID 5 → clean
md52 → RAID 5 → clean
```

The single-member failure had therefore been completely recovered.

---

# 19. Post-Rebuild Filesystem Validation

The filesystem remained accessible at:

```text
/mnt/raid50
```

The mount remained:

```text
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

The filesystem was therefore successfully retained across:

```text
Failure
→ degraded operation
→ member replacement
→ rebuild
→ healthy RAID state
```

---

# 20. Post-Rebuild Data Validation

The test files were read again after rebuild:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

The test data remained readable.

This confirmed that the storage stack continued to provide access after
the failed member had been replaced.

---

# 21. Post-Rebuild SHA-256 Validation

The files were hashed after recovery:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The resulting hashes matched the original healthy-state baseline:

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

for the tested data.

---

# 22. Test Result

The test passed.

### Failure handling

```text
[PASS] loop10 successfully marked failed
[PASS] md51 entered degraded state
[PASS] md52 remained healthy
[PASS] md50 remained active
```

### Data availability

```text
[PASS] Filesystem remained accessible
[PASS] Test files remained readable
```

### Recovery

```text
[PASS] Failed member removed
[PASS] Replacement loop16 added
[PASS] Rebuild completed
[PASS] md51 returned to [3/3] [UUU]
[PASS] md51 returned to clean state
```

### Data integrity

```text
[PASS] SHA-256 unchanged after failure
[PASS] SHA-256 unchanged after rebuild
```

---

# 23. Failure and Recovery Flow

The complete tested sequence was:

```text
Healthy md51
[3/3] [UUU]
      |
      ↓
Fail loop10
      |
      ↓
md51
[3/2] [_UU]
      |
      ↓
Validate data access
      |
      ↓
Validate SHA-256
      |
      ↓
Remove loop10
      |
      ↓
Add replacement loop16
      |
      ↓
Rebuild
      |
      ↓
md51
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

# 24. Engineering Observations

This test demonstrated several important RAID50 behaviors.

### 1. Single-member failure is contained within the component RAID 5

The failure occurred in:

```text
md51
```

while:

```text
md52
```

remained healthy.

---

### 2. RAID50 remained accessible

Because `md51` remained operational in degraded mode and `md52` was
healthy, the top-level RAID50 logical device remained accessible.

---

### 3. RAID 0 did not provide the redundancy

The recovery came from:

```text
md51
RAID 5
```

not from:

```text
md50
RAID 0
```

The outer RAID 0 layer only combined the component arrays.

---

### 4. Data integrity survived the degraded state

The SHA-256 hashes remained unchanged while the RAID 5 component was
degraded.

---

### 5. Rebuild restored the component redundancy

The replacement member:

```text
loop16
```

restored `md51` to:

```text
[3/3] [UUU]
```

and:

```text
Failed Devices = 0
```

---

# 25. Final Acceptance Criteria

The single-member `md51` failure test was accepted because:

```text
[✓] Failure injected successfully

[✓] Correct RAID 5 component entered degraded state

[✓] md52 remained healthy

[✓] md50 remained active

[✓] Filesystem remained accessible

[✓] Test files remained readable

[✓] SHA-256 values remained unchanged

[✓] Failed member removed successfully

[✓] Replacement member added successfully

[✓] Rebuild completed successfully

[✓] md51 returned to [3/3] [UUU]

[✓] md51 returned to clean state

[✓] Final data integrity verified
```

---

# 26. Final Test Conclusion

The RAID50 single-member failure test for `md51` passed successfully.

The tested behavior was:

```text
One member failure in md51
        ↓
md51 degraded
        ↓
RAID50 remained accessible
        ↓
Data remained readable
        ↓
Data integrity remained unchanged
        ↓
Replacement member added
        ↓
RAID 5 rebuild completed
        ↓
md51 returned healthy
```

This validates that the tested RAID50 configuration can survive a
single-member failure within one component RAID 5 group and recover the
failed member without losing the tested filesystem data.

