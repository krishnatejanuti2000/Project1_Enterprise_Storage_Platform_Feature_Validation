# RAID 60 — Lab Setup Evidence

## 1. Lab Objective

The objective of this laboratory was to build and validate a Linux software
RAID 60 configuration using `mdadm`.

The lab was designed to validate:

```text
RAID 60 Architecture
+
RAID 6 Component Groups
+
RAID 0 Striping
+
Single-Member Failure
+
Two-Member Failure
+
Distributed Failure
+
Maximum 2+2 Failure
+
Rebuild
+
Data Accessibility
+
Data Integrity
+
Final Recovery
```

The laboratory used loop-backed image files instead of real physical disks.

This isolated the experiment from the existing operating-system storage.

---

# 2. Lab Environment

Operating system:

```text
RHEL
```

RAID implementation:

```text
Linux mdadm
```

Test filesystem:

```text
ext4
```

Mount point:

```text
/mnt/raid60
```

RAID60 topology:

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
                10 11 12 13  14 15 16 17
```

The configuration therefore consisted of:

```text
2 × RAID 6 component arrays
            ↓
RAID 0 across both components
            ↓
RAID 60
```

---

# 3. Initial RAID State Check

Before creating the laboratory, the md RAID subsystem was checked:

```bash
cat /proc/mdstat
```

The initial state showed:

```text
Personalities : [raid0] [raid4] [raid5] [raid6]
unused devices: <none>
```

This confirmed that no previously active laboratory md array was present
before starting the RAID60 experiment.

---

# 4. Existing Loop-Device Check

The existing loop-device configuration was checked:

```bash
sudo losetup -a
```

At this stage, only the pre-existing host/Snap loop devices were present.

The RAID60 laboratory loop devices had not yet been created.

---

# 5. Protected Existing Storage

The existing host storage was intentionally left untouched.

The RAID60 experiment was performed entirely using temporary image files
attached through loop devices.

The storage safety model was:

```text
Existing OS/storage
        ↓
DO NOT TOUCH

Temporary image files
        ↓
RAID60 laboratory
```

This ensured that RAID failure and rebuild testing did not modify the
existing operating-system storage configuration.

---

# 6. Create RAID60 Lab Workspace

The temporary RAID60 workspace was created:

```bash
mkdir -p ~/raid60_lab
```

This directory was used to store the temporary disk-image files required
for the experiment.

---

# 7. Create Eight Test Disk Images

Eight 1 GiB image files were created:

```text
disk1.img
disk2.img
disk3.img
disk4.img
disk5.img
disk6.img
disk7.img
disk8.img
```

The intended member allocation was:

```text
RAID 6 Group 1
→ disk1
→ disk2
→ disk3
→ disk4

RAID 6 Group 2
→ disk5
→ disk6
→ disk7
→ disk8
```

Therefore:

```text
Total laboratory members = 8
```

---

# 8. Attach Image Files to Loop Devices

The eight image files were attached to loop devices.

Final initial mapping:

```text
disk1.img → /dev/loop10
disk2.img → /dev/loop11
disk3.img → /dev/loop12
disk4.img → /dev/loop13

disk5.img → /dev/loop14
disk6.img → /dev/loop15
disk7.img → /dev/loop16
disk8.img → /dev/loop17
```

The topology at this stage was:

```text
                 Eight Test Members
                        |
             ┌──────────┴──────────┐
             ↓                     ↓
        RAID 6 Group 1       RAID 6 Group 2
        loop10-13            loop14-17
```

---

# 9. Verify Loop-Device Mapping

The loop-device mappings were verified using:

```bash
sudo losetup -a
```

The newly created RAID60 laboratory members were confirmed as:

```text
/dev/loop10
/dev/loop11
/dev/loop12
/dev/loop13
/dev/loop14
/dev/loop15
/dev/loop16
/dev/loop17
```

---

# 10. Verify RAID Metadata on Initial Members

Before creating the RAID6 arrays, each loop member was checked:

```bash
sudo mdadm --examine /dev/loop10
sudo mdadm --examine /dev/loop11
sudo mdadm --examine /dev/loop12
sudo mdadm --examine /dev/loop13
sudo mdadm --examine /dev/loop14
sudo mdadm --examine /dev/loop15
sudo mdadm --examine /dev/loop16
sudo mdadm --examine /dev/loop17
```

All eight devices were clean and reported no existing md RAID
superblock.

This confirmed that the laboratory members were ready for RAID6
creation.

---

# 11. RAID60 Member Allocation

The eight initial members were divided into two independent four-member
RAID6 components.

### RAID 6 Group 1

```text
md61

/dev/loop10
/dev/loop11
/dev/loop12
/dev/loop13
```

### RAID 6 Group 2

```text
md62

/dev/loop14
/dev/loop15
/dev/loop16
/dev/loop17
```

The complete architecture was:

```text
                         RAID 60
                            |
                           md60
                         RAID 0
                      ____/    \____
                     /              \
                    /                \
                 md61              md62
                RAID 6             RAID 6
              / | |  \           / | |  \
          loop10 11 12 13    loop14 15 16 17
```

---

# 12. RAID60 Failure-Domain Design

The laboratory was intentionally built with two independent RAID6
failure domains:

```text
md61 → RAID 6 Group 1
md62 → RAID 6 Group 2
```

This allowed validation of:

```text
Single-member failure
+
Two-member failure within one component
+
One failure in each component
+
Two failures in each component
```

The final scenario was the critical:

```text
2 + 2
```

failure distribution.

---

# 13. Planned Failure Scenarios

The RAID60 laboratory was prepared to test:

```text
Scenario 1
→ Single member failure in md61

Scenario 2
→ Two member failures in md61

Scenario 3
→ Single member failure in md62

Scenario 4
→ One failure in md61
  +
  One failure in md62

Scenario 5
→ Two failures in md61
  +
  Two failures in md62
```

These scenarios validate the RAID60 failure-domain model rather than
testing failure count alone.

---

# 14. Test Filesystem Preparation

The top-level logical RAID device was planned as:

```text
/dev/md60
```

The filesystem used for the laboratory was:

```text
ext4
```

The planned mount point was:

```text
/mnt/raid60
```

The filesystem would be used for:

```text
File accessibility
+
File-content validation
+
SHA-256 integrity validation
```

---

# 15. Data Integrity Test Preparation

Three test files were used during the laboratory:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

The test strategy was:

```text
Healthy State
     ↓
Capture SHA-256 baseline
     ↓
Inject RAID failure
     ↓
Verify file accessibility
     ↓
Verify SHA-256
     ↓
Recover RAID
     ↓
Verify SHA-256 again
```

This provided an objective data-integrity reference throughout the
failure and recovery process.

---

# 16. Expected RAID60 Storage Stack

The intended complete storage stack was:

```text
Application / User
        ↓
ext4
        ↓
/dev/md60
        ↓
RAID 0
        ↓
┌───────────────────────┐
│                       │
md61                   md62
RAID 6                 RAID 6
│                       │
├─ loop10               ├─ loop14
├─ loop11               ├─ loop15
├─ loop12               ├─ loop16
└─ loop13               └─ loop17
```

Logical model:

```text
RAID 6 + RAID 6
       ↓
     RAID 0
       ↓
     RAID 60
```

---

# 17. Maximum Failure Test Design

The RAID60 configuration was specifically intended to validate a
distributed four-member failure pattern:

```text
md61 → 2 failed members
md62 → 2 failed members
```

Conceptually:

```text
md61
[__UU]

md62
[__UU]
```

This represents:

```text
2 failed members
+
2 failed members
=
4 failed members distributed as 2+2
```

The scenario was designed to verify whether both component RAID6
arrays remained operational and whether the top-level RAID60 remained
accessible.

---

# 18. Lab Setup Completion Criteria

The setup stage was considered complete when:

```text
[✓] RAID60 laboratory workspace created

[✓] Eight 1 GiB test images created

[✓] Eight loop devices attached

[✓] Loop-device mappings verified

[✓] All eight members checked for existing md metadata

[✓] No existing md superblocks detected

[✓] Existing host storage left untouched

[✓] RAID6 Group 1 member allocation defined

[✓] RAID6 Group 2 member allocation defined

[✓] RAID60 failure scenarios defined

[✓] Data-integrity validation method defined

[✓] Environment ready for RAID60 creation
```

---

# 19. Setup Summary

Initial laboratory members:

```text
loop10
loop11
loop12
loop13
loop14
loop15
loop16
loop17
```

Component allocation:

```text
md61
→ loop10
→ loop11
→ loop12
→ loop13

md62
→ loop14
→ loop15
→ loop16
→ loop17
```

Planned top-level array:

```text
md61 + md62
     ↓
    md60
     ↓
   RAID60
```

---

# 20. Final Setup Conclusion

The RAID60 laboratory environment was successfully prepared using eight
temporary loop-backed storage members.

The setup established:

```text
8 test members
+
2 RAID6 component groups
+
1 RAID0 top-level layer
+
ext4 filesystem
+
3 test files
+
SHA-256 integrity validation
```

The environment was ready for the RAID60 creation and healthy-state
validation phase.

The subsequent document records the actual creation of `md61`, `md62`,
and `md60`, followed by filesystem creation, test-data population, and
healthy-state validation.

