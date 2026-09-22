# RAID 50 — Creation and Healthy-State Evidence

## 1. Objective

The objective of this phase was to create the complete RAID 50 hierarchy
using Linux `mdadm` and validate that the newly created array was healthy
before any failure was introduced.

The final topology was:

```text
                         RAID 50
                            |
                           md50
                         RAID 0
                       /       \
                      /         \
                   md51         md52
                  RAID 5       RAID 5
                 /  |  \      /  |  \
             loop10 11 12  loop13 14 15
```

---

# 2. Create First RAID 5 Component — md51

The first 3-member RAID 5 component was created using:

```bash
sudo mdadm --create /dev/md51 --level=5 --raid-devices=3 \
/dev/loop10 /dev/loop11 /dev/loop12
```

`mdadm` requested confirmation regarding the internal write-intent
bitmap.

The internal bitmap was accepted.

The resulting component array was:

```text
md51
→ RAID 5
→ 3 members
→ loop10
→ loop11
→ loop12
```

---

# 3. md51 Healthy-State Details

The first RAID 5 component reported:

```text
RAID Level       : raid5
RAID Devices     : 3
Array Size       : 2093056 blocks
Chunk Size       : 512K
Layout           : left-symmetric
Bitmap           : internal
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
```

Member roles:

```text
loop10 → role 0
loop11 → role 1
loop12 → role 2
```

The md51 UUID was:

```text
c20a7c8e:6812ea44:715359b9:e1e3dd3a
```

The mdadm array name was:

```text
winteck:51
```

Therefore md51 was confirmed healthy before continuing.

---

# 4. Create Second RAID 5 Component — md52

The second 3-member RAID 5 component was created using:

```bash
sudo mdadm --create /dev/md52 --level=5 --raid-devices=3 \
/dev/loop13 /dev/loop14 /dev/loop15
```

The internal write-intent bitmap was also accepted.

The resulting component array was:

```text
md52
→ RAID 5
→ 3 members
→ loop13
→ loop14
→ loop15
```

---

# 5. md52 Healthy-State Details

The second RAID 5 component reported:

```text
RAID Level       : raid5
RAID Devices     : 3
Array Size       : 2093056 blocks
Chunk Size       : 512K
Layout           : left-symmetric
Bitmap           : internal
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
```

Member roles:

```text
loop13 → role 0
loop14 → role 1
loop15 → role 2
```

The md52 UUID was:

```text
9ed53b51:c82b776d:0041fd39:7a01e9a8
```

The mdadm array name was:

```text
winteck:52
```

md52 was therefore also confirmed healthy.

---

# 6. Create Top-Level RAID 0 — md50

After both RAID 5 component arrays were healthy, they were combined into
the top-level RAID 0 layer:

```bash
sudo mdadm --create /dev/md50 --level=0 --raid-devices=2 \
/dev/md51 /dev/md52
```

This created the complete nested RAID 50 structure:

```text
md51
RAID 5
   \
    \
     md50
    RAID 0
     /
    /
md52
RAID 5
```

Therefore:

```text
RAID 5 + RAID 5
        ↓
      RAID 0
        ↓
      RAID 50
```

---

# 7. md50 Healthy-State Details

The top-level array reported:

```text
RAID Level       : raid0
RAID Devices     : 2
Array Size       : 4182016 blocks
Chunk Size       : 512K
State            : clean
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 0
```

Top-level members:

```text
md51 → role 0
md52 → role 1
```

No consistency bitmap was configured on the RAID 0 layer.

The md50 UUID was:

```text
35343439:413fcd35:4cf6ac98:092888a8
```

The mdadm array name was:

```text
winteck:50
```

The complete RAID 50 hierarchy was now operational.

---

# 8. Final RAID50 Topology

The completed topology was:

```text
                           md50
                         RAID 0
                    4182016 blocks
                       /       \
                      /         \
                   md51         md52
                  RAID 5       RAID 5
                2093056       2093056
                 blocks        blocks
                 / | \          / | \
                /  |  \        /  |  \
           loop10 loop11 loop12 loop13 loop14 loop15
```

Logical architecture:

```text
md51 = RAID 5
md52 = RAID 5

md50 = RAID 0 across md51 + md52

md50 + md51 + md52
        ↓
      RAID 50
```

---

# 9. Create Filesystem

An ext4 filesystem was created on the top-level RAID device:

```bash
sudo mkfs.ext4 /dev/md50
```

The resulting filesystem UUID was:

```text
8b7090e8-7f85-46b5-a253-b6b8d3e6b7b1
```

The filesystem was therefore created on the logical RAID50 device rather
than directly on an individual component array.

Correct storage stack:

```text
Physical members
      ↓
RAID 5
      ↓
RAID 5
      ↓
RAID 0
      ↓
/dev/md50
      ↓
ext4
```

---

# 10. Create Mount Point

The filesystem was mounted using:

```text
/mnt/raid50
```

The resulting mount was:

```text
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

This confirmed that Linux successfully accessed the RAID50 block device
through the ext4 filesystem.

---

# 11. Filesystem Validation

The mount point was verified and the RAID50 filesystem was accessible.

The storage stack at this point was:

```text
Host
 ↓
ext4
 ↓
/dev/md50
 ↓
RAID 0
 ↓
md51 + md52
 ↓
RAID 5 component members
 ↓
loop-backed storage
```

No failure had yet been introduced.

This state was therefore used as the healthy baseline for subsequent
failure testing.

---

# 12. Create RAID50 Test Data

Three test files were created under:

```text
/mnt/raid50
```

The files were:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

Their recorded sizes were:

```text
testfile_01.txt → 226 bytes
testfile_02.txt → 252 bytes
testfile_03.txt → 243 bytes
```

---

# 13. Test File 01

`testfile_01.txt` contained:

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

This file established the baseline test environment and topology.

---

# 14. Test File 02

`testfile_02.txt` contained:

```text
RAID50-TEST-DATA-002
Storage Engineering Laboratory
Test Scenario: Component RAID5 Failure
Failure Domain: Individual RAID5 Group
Expected Result: RAID50 Remains Accessible After One Member Failure
Rebuild Validation: Required
Data Integrity: Required
```

This file was specifically used for component RAID 5 failure validation.

---

# 15. Test File 03

`testfile_03.txt` contained:

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

This file was used throughout the failure and recovery validation.

---

# 16. Healthy-State SHA-256 Baseline

Before introducing any RAID member failure, SHA-256 checksums were
captured for all test files.

Command:

```bash
sha256sum /mnt/raid50/testfile_*.txt
```

The healthy baseline was:

```text
testfile_01.txt
7c30f4d66983818b9321e763f347f821b28ab5c1e8f8b496dfc4af2cd838c4a0

testfile_02.txt
ef08c205487d8ea98e6fc1985427707df9790506452612eb0bacfbd904d90072

testfile_03.txt
bde5543759a5c299223c09fe593a87cf460329861b31273ff29cd38a7811e7c1
```

These values became the reference integrity baseline for all subsequent
failure and rebuild tests.

---

# 17. Healthy RAID50 Validation

Before failure injection, the following conditions were established:

```text
[✓] md51 created successfully

[✓] md51 RAID level = RAID 5

[✓] md51 active members = 3

[✓] md51 failed members = 0

[✓] md51 state = clean

[✓] md52 created successfully

[✓] md52 RAID level = RAID 5

[✓] md52 active members = 3

[✓] md52 failed members = 0

[✓] md52 state = clean

[✓] md50 created successfully

[✓] md50 RAID level = RAID 0

[✓] md50 active members = 2

[✓] md50 failed members = 0

[✓] ext4 filesystem created

[✓] /mnt/raid50 mounted successfully

[✓] Test files created

[✓] Test files readable

[✓] SHA-256 baseline captured
```

---

# 18. Healthy-State Acceptance Criteria

The RAID50 configuration was accepted as the initial healthy baseline
because:

```text
md51
→ clean
→ 3/3 active
→ 0 failed

md52
→ clean
→ 3/3 active
→ 0 failed

md50
→ clean
→ 2/2 active
→ 0 failed

Filesystem
→ mounted
→ accessible

Test data
→ readable

SHA-256
→ baseline successfully recorded
```

---

# 19. Baseline Architecture Summary

The final healthy RAID50 configuration before failure testing was:

```text
                         RAID 50
                            |
                           md50
                          RAID 0
                       ____/ \____
                      /           \
                     /             \
                   md51           md52
                  RAID 5          RAID 5
                 /  |  \         /  |  \
                /   |   \       /   |   \
           loop10 loop11 loop12 loop13 loop14 loop15
```

Component arrays:

```text
md51
→ loop10
→ loop11
→ loop12

md52
→ loop13
→ loop14
→ loop15
```

Top-level logical device:

```text
/dev/md50
```

Filesystem:

```text
ext4
```

Mount point:

```text
/mnt/raid50
```

---

# 20. Evidence Conclusion

The RAID50 array was successfully created using:

```text
2 × RAID 5 component arrays
+
1 × RAID 0 top-level layer
```

The final healthy baseline contained:

```text
6 RAID members
2 RAID 5 components
1 RAID 0 parent
1 ext4 filesystem
3 test files
3 SHA-256 reference values
```

The complete RAID50 stack was operational before failure testing began.

This healthy state is the reference point against which all subsequent
failure, degradation, rebuild, and data-integrity results must be
compared.

