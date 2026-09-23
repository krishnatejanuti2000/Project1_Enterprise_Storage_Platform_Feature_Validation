# RAID 50 — Lab Setup Evidence

## 1. Lab Objective

The objective of this laboratory was to build and validate a Linux software
RAID 50 configuration using `mdadm`.

The lab was designed to validate:

```text
RAID 50 Architecture
+
RAID 5 Component Groups
+
RAID 0 Striping
+
Single-Member Failure
+
Distributed Failure
+
Rebuild
+
Data Accessibility
+
Data Integrity
```

The laboratory used loop-backed image files instead of real physical disks.

This approach isolated the experiment from the existing operating-system
storage.

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
/mnt/raid50
```

RAID50 topology:

```text
                    RAID 50
                       |
                    md50
                   RAID 0
                 /         \
                /           \
             md51           md52
            RAID 5          RAID 5
           /  |  \         /  |  \
       loop10 loop11 loop12 loop13 loop14 loop15
```

The configuration therefore consisted of:

```text
2 × RAID 5 component arrays
            ↓
RAID 0 across both components
            ↓
RAID 50
```

---

# 3. Storage Safety Check

Before creating the laboratory, the existing software RAID state was
checked.

Command:

```bash
cat /proc/mdstat
```

Initial output showed no active md arrays:

```text
Personalities :
unused devices: <none>
```

This confirmed that no previously active md array was present before
starting the RAID50 experiment.

---

# 4. Existing Loop Devices

The existing loop-device configuration was also checked:

```bash
sudo losetup -a
```

Only the pre-existing Ubuntu Snap loop devices were present.

The RAID50 laboratory loop devices had not yet been created.

---

# 5. Protected Storage Devices

The existing host storage configuration was reviewed before creating
the test environment.

Important rule:

```text
Existing OS/storage disks
        ↓
DO NOT TOUCH
```

The RAID50 experiment was therefore performed entirely using temporary
image files and loop devices.

This avoided modifying the existing system LVM/storage configuration.

---

# 6. Create RAID50 Lab Workspace

The laboratory workspace was created:

```bash
mkdir -p ~/raid50_lab
```

The workspace was used to hold the temporary disk-image files required for
the experiment.

---

# 7. Create Six Test Disk Images

Six 1 GiB image files were created:

```text
disk1.img
disk2.img
disk3.img
disk4.img
disk5.img
disk6.img
```

Conceptually:

```text
disk1.img → 1 GiB
disk2.img → 1 GiB
disk3.img → 1 GiB
disk4.img → 1 GiB
disk5.img → 1 GiB
disk6.img → 1 GiB
```

These image files represented the six members required for the two
3-member RAID 5 component arrays.

---

# 8. Attach Image Files to Loop Devices

The six image files were attached as loop devices.

Final mapping:

```text
disk1.img → /dev/loop10
disk2.img → /dev/loop11
disk3.img → /dev/loop12

disk4.img → /dev/loop13
disk5.img → /dev/loop14
disk6.img → /dev/loop15
```

The topology at this stage was:

```text
             Six Test Members
                    |
        ┌───────────┴───────────┐
        ↓                       ↓
   RAID 5 Group 1          RAID 5 Group 2
   loop10-12               loop13-15
```

---

# 9. Verify Loop Devices

The loop-device mappings were verified using:

```bash
sudo losetup -a
```

The newly created laboratory devices were confirmed as:

```text
/dev/loop10
/dev/loop11
/dev/loop12
/dev/loop13
/dev/loop14
/dev/loop15
```

---

# 10. Verify No Existing RAID Metadata

Before using the loop devices for RAID creation, their existing md metadata
was checked:

```bash
sudo mdadm --examine /dev/loop10
sudo mdadm --examine /dev/loop11
sudo mdadm --examine /dev/loop12
sudo mdadm --examine /dev/loop13
sudo mdadm --examine /dev/loop14
sudo mdadm --examine /dev/loop15
```

The devices reported no existing md superblock.

This confirmed that the six test members were clean and ready for RAID
creation.

---

# 11. RAID50 Lab Member Allocation

The six test members were divided into two independent RAID 5 groups.

### RAID 5 Group 1

```text
md51

/dev/loop10
/dev/loop11
/dev/loop12
```

### RAID 5 Group 2

```text
md52

/dev/loop13
/dev/loop14
/dev/loop15
```

The complete architecture was therefore:

```text
                    RAID 50
                       |
                      md50
                    RAID 0
                  /         \
                 /           \
              md51           md52
            RAID 5          RAID 5
          /    |    \      /    |    \
     loop10 loop11 loop12 loop13 loop14 loop15
```

---

# 12. RAID50 Lab Design

The laboratory was intentionally designed with:

```text
2 RAID 5 component groups
+
3 members per RAID 5 group
+
1 top-level RAID 0 layer
```

Therefore:

```text
Total members = 6
```

The design provides a clear failure-domain model:

```text
md51 failure domain
        +
md52 failure domain
```

This allows single-member and distributed-member failure scenarios to be
tested independently.

---

# 13. Failure-Test Design

The laboratory was prepared to validate the following scenarios:

```text
Scenario 1
→ One member failure in md51

Scenario 2
→ One member failure in md52

Scenario 3
→ One failed member in md51
  +
  One failed member in md52
```

The distributed failure scenario was particularly important because RAID50
failure behavior depends on the component RAID 5 group in which the
failure occurs.

---

# 14. Data Integrity Test Preparation

The RAID50 validation also required a persistent test dataset.

The filesystem would later be created on:

```text
/dev/md50
```

and mounted at:

```text
/mnt/raid50
```

Test files were then used to validate:

```text
File accessibility
+
File contents
+
SHA-256 integrity
```

The baseline hashes were captured before failure injection.

---

# 15. Baseline RAID State

At the beginning of the experiment:

```text
No RAID50 arrays existed.
No RAID51 array existed.
No RAID52 array existed.
No RAID50 filesystem existed.
```

The environment was therefore in a clean state before RAID50 creation.

---

# 16. Lab Safety Principle

The most important safety rule followed during the experiment was:

```text
Temporary test storage
        ↓
Loop-backed image files
        ↓
RAID laboratory
```

rather than:

```text
Real OS disks
        ↓
RAID experiment
```

This ensured that the RAID50 failure and rebuild experiments could be
performed without intentionally modifying the existing operating-system
storage.

---

# 17. Expected Final Architecture

After the setup stage, the planned RAID50 hierarchy was:

```text
                           RAID 50
                              |
                             md50
                            RAID 0
                       ______/ \______
                      /               \
                     /                 \
                   md51               md52
                 RAID 5              RAID 5
                /  |  \             /  |  \
               /   |   \           /   |   \
         loop10 loop11 loop12 loop13 loop14 loop15
```

Logical structure:

```text
RAID 5 + RAID 5
       ↓
    RAID 0
       ↓
    RAID 50
```

---

# 18. Setup Completion Criteria

The setup stage was considered complete when all of the following were
true:

```text
[✓] Six 1 GiB image files created

[✓] Six loop devices attached

[✓] Loop mappings verified

[✓] Existing md metadata checked

[✓] No existing md superblocks detected

[✓] Existing OS/storage disks left untouched

[✓] RAID50 component allocation defined

[✓] Laboratory workspace created

[✓] Environment ready for RAID creation
```

---

# 19. Setup Evidence Summary

Final test members:

```text
loop10
loop11
loop12
loop13
loop14
loop15
```

Component allocation:

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

Top-level architecture:

```text
md51 + md52
    ↓
md50
    ↓
RAID 50
```

This completed the RAID50 laboratory preparation phase.

The next evidence document records the actual creation of `md51`,
`md52`, and `md50`, followed by filesystem creation and validation of
the initial healthy state.

