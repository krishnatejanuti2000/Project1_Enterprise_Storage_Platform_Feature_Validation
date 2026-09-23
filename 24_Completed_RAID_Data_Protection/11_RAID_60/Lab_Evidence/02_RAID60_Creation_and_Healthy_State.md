# RAID 60 — Creation and Healthy-State Evidence

## 1. Objective

The objective of this phase was to create the complete Linux software
RAID60 hierarchy using `mdadm` and validate the initial healthy state
before introducing any failures.

The final topology was:

```text
                           RAID 60
                              |
                             md60
                           RAID 0
                         /       \
                        /         \
                     md61         md62
                    RAID 6       RAID 6
                   / | | \      / | | \
              loop10 11 12 13 loop14 15 16 17
```

The architecture was:

```text
RAID 6 + RAID 6
       ↓
     RAID 0
       ↓
     RAID 60
```

---

# 2. Create First RAID 6 Component — md61

The first four-member RAID 6 component was created using:

```bash
sudo mdadm --create /dev/md61 --level=6 --raid-devices=4 \
/dev/loop10 /dev/loop11 /dev/loop12 /dev/loop13
```

The internal write-intent bitmap was accepted during array creation.

The resulting component array was:

```text
md61
→ RAID 6
→ loop10
→ loop11
→ loop12
→ loop13
```

---

# 3. md61 Healthy-State Details

The first RAID 6 component reported:

```text
RAID Level       : raid6
RAID Devices     : 4
Array Size       : 2093056 blocks
Used Dev Size    : 1046528
Chunk Size       : 512K
Layout           : left-symmetric
Bitmap           : internal
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

Member roles:

```text
loop10 → role 0
loop11 → role 1
loop12 → role 2
loop13 → role 3
```

The md61 UUID was:

```text
387c721f:16bf5d32:501be4de:6bed98ae
```

The mdadm array name was:

```text
winteck:61
```

md61 was therefore confirmed healthy.

---

# 4. Create Second RAID 6 Component — md62

The second four-member RAID 6 component was created using:

```bash
sudo mdadm --create /dev/md62 --level=6 --raid-devices=4 \
/dev/loop14 /dev/loop15 /dev/loop16 /dev/loop17
```

The internal write-intent bitmap was also accepted.

The resulting component array was:

```text
md62
→ RAID 6
→ loop14
→ loop15
→ loop16
→ loop17
```

---

# 5. md62 Healthy-State Details

The second RAID 6 component reported:

```text
RAID Level       : raid6
RAID Devices     : 4
Array Size       : 2093056 blocks
Used Dev Size    : 1046528
Chunk Size       : 512K
Layout           : left-symmetric
Bitmap           : internal
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
```

Member roles:

```text
loop14 → role 0
loop15 → role 1
loop16 → role 2
loop17 → role 3
```

The md62 UUID was:

```text
e3c8998b:ffe6207c:9b20cb87:b2003f05
```

The mdadm array name was:

```text
winteck:62
```

md62 was therefore confirmed healthy.

---

# 6. Create Top-Level RAID 0 — md60

After both RAID 6 components were healthy, they were combined into the
top-level RAID 0 layer:

```bash
sudo mdadm --create /dev/md60 --level=0 --raid-devices=2 \
/dev/md61 /dev/md62
```

This created the complete RAID60 hierarchy:

```text
md61
RAID 6
   \
    \
     md60
    RAID 0
     /
    /
md62
RAID 6
```

Therefore:

```text
md61 + md62
      ↓
     md60
      ↓
    RAID 60
```

---

# 7. md60 Healthy-State Details

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
md61 → role 0
md62 → role 1
```

The RAID 0 layer had no consistency bitmap.

The md60 UUID was:

```text
ec6a0ed0:14bed07f:b7acc202:893b78f8
```

The mdadm array name was:

```text
winteck:60
```

---

# 8. Final RAID60 Topology

The completed nested RAID60 architecture was:

```text
                           md60
                         RAID 0
                    4182016 blocks
                       /       \
                      /         \
                   md61         md62
                  RAID 6       RAID 6
                2093056       2093056
                 blocks        blocks
                / | | \       / | | \
               /  | |  \     /  | |  \
          loop10 11 12 13 loop14 15 16 17
```

Logical structure:

```text
md61 = RAID 6
md62 = RAID 6

md60 = RAID 0 across md61 + md62

md60 = top-level RAID60 logical device
```

---

# 9. Create Filesystem

An ext4 filesystem was created on the top-level RAID60 device:

```bash
sudo mkfs.ext4 /dev/md60
```

The filesystem UUID was:

```text
0043e881-c169-45f3-9c2c-a34cd8a4ebff
```

The filesystem was therefore created on:

```text
/dev/md60
```

rather than directly on either RAID 6 component.

The complete storage stack was:

```text
Physical members
      ↓
RAID 6
      ↓
RAID 6
      ↓
RAID 0
      ↓
/dev/md60
      ↓
ext4
```

---

# 10. Create Mount Point

The filesystem was mounted at:

```text
/mnt/raid60
```

The resulting mount was:

```text
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

This confirmed successful access to the top-level RAID60 logical device.

---

# 11. Filesystem Validation

The filesystem was validated before failure testing.

Storage path:

```text
Host
 ↓
ext4
 ↓
/dev/md60
 ↓
RAID 0
 ↓
md61 + md62
 ↓
RAID 6 members
 ↓
loop-backed storage
```

No RAID member failure had been introduced at this stage.

This state was therefore used as the healthy baseline.

---

# 12. Create RAID60 Test Data

Three test files were created under:

```text
/mnt/raid60
```

The files were:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

Recorded sizes:

```text
testfile_01.txt → 226 bytes
testfile_02.txt → 233 bytes
testfile_03.txt → 243 bytes
```

These files were used during all subsequent failure and recovery tests.

---

# 13. Testfile 01 Content

`testfile_01.txt` contained:

```text
RAID60-TEST-DATA-001
Enterprise Storage Validation
RAID Level: RAID 60
Architecture: RAID 0 across two RAID 6 groups
Primary Test: Data Accessibility
Member Count: 8
Component Arrays: md61 + md62
Status: Initial Healthy State
```

This file established the initial healthy-state test condition.

---

# 14. Testfile 02 Content

`testfile_02.txt` contained:

```text
RAID60-TEST-DATA-002
Storage Engineering Laboratory
Test Scenario: Dual-Member Component Failure
Failure Domain: Individual RAID6 Group
Expected Result: RAID60 Remains Accessible
Rebuild Validation: Required
Data Integrity: Required
```

This file represented the dual-member failure scenario within a RAID 6
component.

---

# 15. Testfile 03 Content

`testfile_03.txt` contained:

```text
RAID60-TEST-DATA-003
Linux mdadm Nested RAID Validation
Filesystem: ext4
Mount Point: /mnt/raid60
Topology: RAID6 + RAID6 striped by RAID0
Purpose: RAID60 Failure and Recovery Testing
Expected Result: Data Remains Consistent
Final Check: PASS
```

This file represented the overall nested RAID60 validation.

---

# 16. Healthy-State SHA-256 Baseline

Before failure injection, SHA-256 checksums were captured:

```bash
sha256sum /mnt/raid60/testfile_*.txt
```

The healthy-state baseline was:

```text
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

These values became the reference integrity baseline for all subsequent
failure and recovery scenarios.

---

# 17. Healthy-State RAID Validation

The complete RAID60 environment was validated before failure injection.

Component state:

```text
md61
[4/4] [UUUU]

md62
[4/4] [UUUU]
```

Top-level state:

```text
md60
→ active
→ 2/2 working
→ 0 failed
```

Therefore:

```text
md61 → clean
md62 → clean
md60 → clean/active
```

---

# 18. Healthy-State Filesystem Validation

The filesystem mount was:

```text
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

Capacity was approximately:

```text
/dev/md60
≈ 3.9 GiB usable filesystem capacity
```

The test directory contained:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

All files were readable.

---

# 19. Baseline Validation Summary

Before any failure was introduced:

```text
[✓] md61 created successfully

[✓] md61 RAID level = RAID 6

[✓] md61 active members = 4

[✓] md61 failed members = 0

[✓] md61 state = clean

[✓] md62 created successfully

[✓] md62 RAID level = RAID 6

[✓] md62 active members = 4

[✓] md62 failed members = 0

[✓] md62 state = clean

[✓] md60 created successfully

[✓] md60 RAID level = RAID 0

[✓] md60 active members = 2

[✓] md60 failed members = 0

[✓] ext4 filesystem created

[✓] /mnt/raid60 mounted successfully

[✓] Test files created

[✓] Test files readable

[✓] SHA-256 baseline captured
```

---

# 20. Healthy-State Acceptance Criteria

The RAID60 configuration was accepted as the initial healthy baseline
because:

```text
md61
→ clean
→ 4/4 active
→ 0 failed

md62
→ clean
→ 4/4 active
→ 0 failed

md60
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

# 21. Baseline Failure-Domain Model

The healthy RAID60 failure domains were:

```text
                    md60
                  RAID 0
                 /      \
                /        \
             md61        md62
            RAID 6      RAID 6
            4 members   4 members
```

Each RAID 6 component independently provided dual-parity protection.

Therefore:

```text
md61
→ up to two failed members can be tolerated

md62
→ up to two failed members can be tolerated
```

The outer RAID 0 layer provided striping but no additional redundancy.

---

# 22. Baseline Architecture Summary

Final healthy RAID60 configuration:

```text
                           md60
                         RAID 0
                    ____/     \____
                   /               \
                  /                 \
                md61               md62
               RAID 6             RAID 6
              [UUUU]              [UUUU]
             / | | \             / | | \
            /  | |  \           /  | |  \
        loop10 11 12 13     loop14 15 16 17
```

Filesystem:

```text
/dev/md60
   ↓
ext4
   ↓
/mnt/raid60
```

Test data:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

---

# 23. Evidence Conclusion

The RAID60 configuration was successfully created using:

```text
2 × RAID 6 component arrays
+
1 × RAID 0 top-level layer
```

The healthy baseline contained:

```text
8 RAID members
2 RAID 6 failure domains
1 RAID 0 parent
1 ext4 filesystem
3 test files
3 SHA-256 reference values
```

All component arrays and the top-level logical device were healthy before
failure testing began.

This healthy state became the reference point for the subsequent
single-member, two-member, distributed, and maximum 2+2 failure tests.

