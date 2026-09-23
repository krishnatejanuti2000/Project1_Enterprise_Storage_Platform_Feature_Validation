# RAID 50 — Data Integrity Validation Evidence

## 1. Validation Objective

The objective of this test was to verify that RAID50 preserved the
contents of the test data during:

```text
Healthy State
    ↓
RAID5 Member Failure
    ↓
Degraded Operation
    ↓
Member Replacement
    ↓
Rebuild
    ↓
Healthy State
```

The validation used:

```text
File accessibility
+
File-content verification
+
SHA-256 checksum comparison
```

The purpose was to verify that RAID50 failure and recovery did not alter
the tested data.

---

# 2. RAID50 Data Validation Architecture

The test data was stored on:

```text
/dev/md50
```

with the following storage stack:

```text
                    ext4
                     ↓
                  /dev/md50
                     ↓
                 RAID 0 layer
                /           \
               /             \
            md51             md52
           RAID 5           RAID 5
```

The filesystem was mounted at:

```text
/mnt/raid50
```

---

# 3. Test Dataset

Three test files were created:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

These files were intentionally small and contained fixed validation
content.

Recorded sizes:

```text
testfile_01.txt → 226 bytes
testfile_02.txt → 252 bytes
testfile_03.txt → 243 bytes
```

The files were used as persistent reference data throughout the RAID50
failure and recovery tests.

---

# 4. Testfile 01 Content

```text
RAID50-TEST-DATA-001
Enterprise Storage Validation
RAID Level: RAID 50
Architecture: RAID 0 across two RAID 5 groups
Primary Test: Data Accessibility
Member Count: 6
Component Arrays: md51 + md52
Status: Initial Healthy State
```

This file established the initial RAID50 topology and healthy-state
test condition.

---

# 5. Testfile 02 Content

```text
RAID50-TEST-DATA-002
Storage Engineering Laboratory
Test Scenario: Component RAID5 Failure
Failure Domain: Individual RAID5 Group
Expected Result: RAID50 Remains Accessible After One Member Failure
Rebuild Validation: Required
Data Integrity: Required
```

This file specifically represented the component RAID5 failure scenario.

---

# 6. Testfile 03 Content

```text
RAID50-TEST-DATA-003
Linux mdadm Nested RAID Validation
Filesystem: ext4
Mount Point: /mnt/raid50
Topology: RAID5 + RAID5 striped by RAID0
Purpose: RAID50 Failure and Recovery Testing
Expected Result: Data Remains Consistent
Final Check: PASS
```

This file represented the overall nested RAID50 validation.

---

# 7. Healthy-State Baseline

Before introducing any RAID member failure, SHA-256 checksums were
captured.

Command:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The healthy-state baseline was:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

These values were treated as the authoritative integrity baseline for
the fresh final RAID50 run.

---

# 8. Integrity Validation Method

The test used the following validation model:

```text
Healthy baseline
       ↓
SHA-256 captured
       ↓
Failure injected
       ↓
Files read
       ↓
SHA-256 recalculated
       ↓
Member rebuilt
       ↓
Files read again
       ↓
SHA-256 recalculated
       ↓
Compare all results
```

Expected result:

```text
Baseline SHA-256
       =
Post-failure SHA-256
       =
Post-rebuild SHA-256
```

---

# 9. Data Validation — md51 Single-Member Failure

The first failure scenario affected:

```text
md51
```

Failed member:

```text
/dev/loop10
```

The resulting degraded state was:

```text
md51
[3/2] [_UU]
```

while:

```text
md52
[3/3] [UUU]
```

The top-level RAID50 remained accessible.

---

# 10. File Accessibility During md51 Failure

The test files were accessed from:

```text
/mnt/raid50
```

The following files remained readable:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

Commands used for validation included:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All tested files remained accessible during the degraded state.

---

# 11. SHA-256 Validation During md51 Failure

Checksums were recalculated while `md51` was degraded:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The results matched the original baseline for all three files.

Validation:

```text
testfile_01.txt → MATCH
testfile_02.txt → MATCH
testfile_03.txt → MATCH
```

Therefore:

```text
md51 degraded
      ↓
RAID50 remains accessible
      ↓
Test files remain readable
      ↓
SHA-256 remains unchanged
```

---

# 12. Data Validation After md51 Rebuild

The failed `loop10` member was replaced with:

```text
/dev/loop16
```

and `md51` was rebuilt.

Final component state:

```text
md51
[3/3] [UUU]
```

The checksums were recalculated again:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

All three checksum values matched the healthy baseline.

Therefore the tested data remained intact after the complete
md51 failure/rebuild sequence.

---

# 13. Data Validation — md52 Single-Member Failure

The second failure scenario affected:

```text
md52
```

Failed member:

```text
/dev/loop13
```

The degraded state was:

```text
md52
[3/2] [_UU]
```

while:

```text
md51
[3/3] [UUU]
```

The top-level RAID50 remained active.

---

# 14. File Accessibility During md52 Failure

The same test dataset was validated while `md52` was degraded.

Commands:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All three files remained readable.

This demonstrated continued logical access to the filesystem during the
degraded state.

---

# 15. SHA-256 Validation During md52 Failure

Checksums were recalculated:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

Results:

```text
testfile_01.txt → MATCH
testfile_02.txt → MATCH
testfile_03.txt → MATCH
```

All values matched the healthy baseline.

---

# 16. Data Validation After md52 Rebuild

The failed `loop13` member was replaced with:

```text
/dev/loop17
```

and `md52` was rebuilt.

Final component state:

```text
md52
[3/3] [UUU]
```

The test files were read again and the checksums were recalculated.

All SHA-256 values matched the original healthy baseline.

Therefore the data remained intact throughout the complete md52
failure/rebuild sequence.

---

# 17. Distributed Failure Validation

The RAID50 configuration was then tested with one failed member in each
component:

```text
md51 → loop16 failed
md52 → loop17 failed
```

Resulting component states:

```text
md51 → [3/2] [_UU]
md52 → [3/2] [_UU]
```

The top-level RAID50 remained active.

This created simultaneous degraded operation in both component RAID5
failure domains.

---

# 18. File Accessibility During Distributed Degradation

The filesystem remained mounted at:

```text
/mnt/raid50
```

The test files were checked:

```bash
ls -lh /mnt/raid50
```

All three files remained present.

They were then read:

```bash
cat /mnt/raid50/testfile_01.txt
cat /mnt/raid50/testfile_02.txt
cat /mnt/raid50/testfile_03.txt
```

All three files remained readable while both component RAID5 arrays were
degraded.

---

# 19. SHA-256 Validation During Distributed Degradation

The checksums were recalculated while both component arrays were
degraded:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The resulting values matched the healthy baseline:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

Result:

```text
Baseline
   =
Distributed degraded state
```

for every tested file.

---

# 20. Data Validation During Recovery

The distributed failures were recovered sequentially.

First:

```text
md51
```

was rebuilt.

Then:

```text
md52
```

was rebuilt.

During recovery, the test dataset remained available for validation.

The recovery sequence was:

```text
md51 failed
    ↓
md51 rebuilt
    ↓
md51 healthy

md52 failed
    ↓
md52 rebuilt
    ↓
md52 healthy
```

---

# 21. Final Data Integrity Validation

After both component RAID5 arrays returned to healthy state, checksums
were recalculated:

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

These matched the healthy-state baseline exactly.

---

# 22. Integrity Comparison

| File            | Healthy Baseline | During Failure | After Rebuild | Result |
| --------------- | ---------------- | -------------- | ------------- | ------ |
| testfile_01.txt | `7c30f4...c4a0`  | MATCH          | MATCH         | PASS   |
| testfile_02.txt | `ef08c2...0072`  | MATCH          | MATCH         | PASS   |
| testfile_03.txt | `bde554...1e7c1` | MATCH          | MATCH         | PASS   |

The complete SHA-256 values are recorded above as the authoritative
reference values.

---

# 23. Data Validation Layers

The integrity test validated the storage stack at multiple levels:

```text
Layer 1
RAID member failure

        ↓

Layer 2
RAID 5 degraded operation

        ↓

Layer 3
RAID 0 parent remains active

        ↓

Layer 4
/dev/md50 remains accessible

        ↓

Layer 5
ext4 filesystem remains mounted

        ↓

Layer 6
Files remain readable

        ↓

Layer 7
SHA-256 remains unchanged
```

This is stronger evidence than checking only the RAID status.

---

# 24. Why SHA-256 Was Used

A checksum provides a reproducible representation of the file contents.

For this laboratory:

```text
Healthy file contents
       ↓
SHA-256 baseline
```

After a failure or rebuild:

```text
Current file contents
       ↓
SHA-256
```

Comparison:

```text
MATCH
→ tested file contents unchanged

MISMATCH
→ investigate before declaring recovery successful
```

The checksum comparison was therefore used as a data-integrity
validation mechanism.

---

# 25. Data Integrity Acceptance Criteria

The integrity test required:

```text
[✓] Baseline SHA-256 captured

[✓] Test files remained present after failure

[✓] Test files remained readable during degraded operation

[✓] SHA-256 matched during md51 degradation

[✓] SHA-256 matched after md51 rebuild

[✓] SHA-256 matched during md52 degradation

[✓] SHA-256 matched after md52 rebuild

[✓] Files remained readable during distributed degradation

[✓] SHA-256 matched during distributed degradation

[✓] SHA-256 matched after complete recovery
```

---

# 26. Overall Integrity Result

```text
DATA INTEGRITY RESULT: PASS
```

All three test files retained their original SHA-256 values throughout
the tested failure and recovery scenarios.

---

# 27. Integrity Validation Summary

The fresh RAID50 run demonstrated:

```text
Single md51 failure
        ↓
Data accessible
        ↓
Checksum unchanged

Single md52 failure
        ↓
Data accessible
        ↓
Checksum unchanged

Distributed failure
        ↓
Both RAID5 components degraded
        ↓
Data accessible
        ↓
Checksum unchanged

Recovery
        ↓
Both components rebuilt
        ↓
Checksum unchanged
```

Therefore the test dataset remained consistent throughout the tested
RAID50 failure and recovery lifecycle.

---

# 28. Final Conclusion

The RAID50 data-integrity validation passed.

The healthy-state SHA-256 values were preserved:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

The validation established that, for the tested RAID50 failure and
recovery scenarios:

```text
RAID degradation
      +
Member replacement
      +
RAID 5 rebuild
      +
Distributed component failure
```

did not alter the contents of the tested files.

Final validation condition:

```text
RAID state healthy
        +
Filesystem accessible
        +
Files readable
        +
SHA-256 matches baseline
        ↓
DATA INTEGRITY VALIDATED
```

