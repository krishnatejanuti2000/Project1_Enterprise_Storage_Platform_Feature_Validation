# RAID 50 — Lab Cleanup Evidence

## 1. Cleanup Objective

The objective of this phase was to safely dismantle the temporary RAID50
laboratory after all planned failure, recovery, rebuild, and data-integrity
tests were completed successfully.

The cleanup was performed intentionally.

The absence of RAID arrays after this phase must therefore **not** be
interpreted as a RAID failure.

The cleanup sequence was:

```text id="jmbt4g"
Final healthy RAID50
        ↓
Unmount filesystem
        ↓
Stop top-level RAID
        ↓
Stop component RAID arrays
        ↓
Detach laboratory loop devices
        ↓
Remove temporary lab workspace
        ↓
Verify no lab arrays remain
        ↓
Verify no lab loop mappings remain
```

---

# 2. Pre-Cleanup State

Before cleanup, the RAID50 environment had returned to a healthy state.

The final topology was:

```text id="2p2qtd"
                         md50
                        RAID 0
                     /         \
                    /           \
                 md51           md52
                RAID 5          RAID 5
```

Final component members:

```text id="jjow89"
md51:
loop18
loop11
loop12

md52:
loop19
loop14
loop15
```

The top-level filesystem was:

```text id="3l4b0k"
/dev/md50
```

mounted at:

```text id="8ac2bv"
/mnt/raid50
```

The final state had already been validated as healthy before cleanup.

---

# 3. Final Data Validation Before Cleanup

The test files were still present:

```text id="cznsnj"
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

The final filesystem state was:

```text id="qm0t8a"
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

Final capacity:

```text id="1rdk5h"
/dev/md50  3.9G  1.1M  3.7G  1% /mnt/raid50
```

The final SHA-256 values matched the original healthy-state baseline.

Therefore all functional RAID50 validation was complete before teardown
began.

---

# 4. Unmount RAID50 Filesystem

The RAID50 filesystem was first unmounted from:

```text id="8z8ds3"
/mnt/raid50
```

The purpose of unmounting first was to ensure that the filesystem was no
longer actively using the top-level RAID device.

Cleanup sequence:

```text id="b4p2ha"
/mnt/raid50
    ↓
unmount
    ↓
/dev/md50 no longer used by filesystem
```

This was a controlled teardown operation.

---

# 5. Stop Top-Level RAID50 Device

After the filesystem was unmounted, the top-level RAID 0 device was
stopped.

Command:

```bash
sudo mdadm --stop /dev/md50
```

This removed the top-level logical RAID device from active use.

The hierarchy at this point became:

```text id="s8f4t9"
md50
  ↓
stopped

md51
md52
  ↓
still available for controlled teardown
```

---

# 6. Stop First RAID 5 Component

The first component array was then stopped:

```bash
sudo mdadm --stop /dev/md51
```

This removed the first RAID 5 component from active md operation.

The first component had already been completely rebuilt and verified as
healthy before this operation.

---

# 7. Stop Second RAID 5 Component

The second component array was then stopped:

```bash
sudo mdadm --stop /dev/md52
```

Both RAID 5 component arrays had now been intentionally stopped.

The complete RAID50 stack was therefore dismantled:

```text id="4j8f1h"
md50 → stopped
md51 → stopped
md52 → stopped
```

---

# 8. Verify No Active md Arrays

After stopping the RAID devices, the md subsystem was checked:

```bash
cat /proc/mdstat
```

The final output showed:

```text id="5n5au0"
Personalities : [raid0] [raid4] [raid5]
unused devices: <none>
```

No RAID50 laboratory arrays remained active.

This was the expected result of the cleanup procedure.

Important:

```text id="4z04an"
No active md arrays
        ≠
RAID50 failure
```

It indicated that the temporary test environment had been intentionally
shut down.

---

# 9. Identify Laboratory Loop Devices

The RAID50 experiment had used loop devices:

```text id="n5pj2a"
loop10
loop11
loop12
loop13
loop14
loop15
loop16
loop17
loop18
loop19
```

The mapping represented:

```text id="egv8ia"
Original members:
loop10 → disk1.img
loop11 → disk2.img
loop12 → disk3.img
loop13 → disk4.img
loop14 → disk5.img
loop15 → disk6.img

Replacement members:
loop16 → replacement1.img
loop17 → replacement2.img
loop18 → replacement3.img
loop19 → replacement4.img
```

These devices belonged exclusively to the temporary RAID50 laboratory.

---

# 10. Detach Laboratory Loop Devices

The laboratory loop devices were detached after all md arrays had been
stopped.

The detachment covered:

```text id="4q8xnr"
loop10
loop11
loop12
loop13
loop14
loop15
loop16
loop17
loop18
loop19
```

The important operational rule was:

```text id="v9f9ri"
Stop RAID
      ↓
Then detach backing loop devices
```

This prevents the backing devices from being detached while still in
active RAID use.

---

# 11. Verify Loop Device Cleanup

The loop-device state was checked using:

```bash
sudo losetup -a
```

After cleanup, the RAID50 laboratory loop mappings were no longer present.

Only the normal pre-existing host loop devices remained.

This confirmed:

```text id="5e99v6"
[✓] loop10-19 detached
[✓] RAID50 loop mappings removed
[✓] Existing host loop devices preserved
```

---

# 12. Verify Block Devices

The block-device inventory was checked:

```bash
lsblk
```

The temporary RAID50 loop-backed devices were no longer part of the
active storage topology.

The normal physical storage devices and pre-existing system devices
remained unchanged.

This confirmed that cleanup did not intentionally alter the existing
host storage configuration.

---

# 13. Remove Temporary Lab Workspace

The temporary RAID50 workspace was:

```text id="e3zjkh"
~/raid50_lab
```

After the RAID and loop devices had been dismantled, the temporary
workspace was removed.

The directory and its temporary image files were no longer required
because all evidence had already been captured in the project
documentation.

---

# 14. Verify Lab Workspace Removal

The workspace was verified to ensure that the temporary RAID50 image
files were no longer present.

Expected result:

```text id="yh7b4d"
~/raid50_lab
→ removed
```

This prevented temporary test storage from being accidentally retained
on the system.

---

# 15. Final md Subsystem Verification

The md subsystem was checked again:

```bash
cat /proc/mdstat
```

Final state:

```text id="wr1xg1"
Personalities : [raid0] [raid4] [raid5]
unused devices: <none>
```

No laboratory RAID arrays were active.

---

# 16. Final Loop-Mapping Verification

Loop mappings were checked again:

```bash
sudo losetup -a
```

The temporary RAID50 laboratory mappings:

```text id="vabps8"
loop10
loop11
loop12
loop13
loop14
loop15
loop16
loop17
loop18
loop19
```

were absent.

Only normal pre-existing loop devices remained.

---

# 17. Final Host Storage Verification

The host storage topology was checked with:

```bash
lsblk
```

The final environment contained the normal storage devices and did not
contain the temporary RAID50 laboratory hierarchy.

This verified that the experiment had been isolated and cleaned up
without leaving the temporary RAID configuration active.

---

# 18. Cleanup Verification Matrix

| Cleanup Item                                   | Result |
| ---------------------------------------------- | ------ |
| `/mnt/raid50` unmounted                        | PASS   |
| `/dev/md50` stopped                            | PASS   |
| `/dev/md51` stopped                            | PASS   |
| `/dev/md52` stopped                            | PASS   |
| RAID50 md arrays removed from active operation | PASS   |
| loop10 detached                                | PASS   |
| loop11 detached                                | PASS   |
| loop12 detached                                | PASS   |
| loop13 detached                                | PASS   |
| loop14 detached                                | PASS   |
| loop15 detached                                | PASS   |
| loop16 detached                                | PASS   |
| loop17 detached                                | PASS   |
| loop18 detached                                | PASS   |
| loop19 detached                                | PASS   |
| Temporary `raid50_lab` workspace removed       | PASS   |
| Pre-existing host loop devices preserved       | PASS   |
| Final `/proc/mdstat` clean of lab arrays       | PASS   |

---

# 19. Cleanup Safety Principle

The cleanup followed the reverse order of the storage stack:

```text id="2k9j5t"
Filesystem
    ↓
Top-level RAID 0
    ↓
Component RAID 5 arrays
    ↓
Loop-backed members
    ↓
Temporary image files
```

This is the correct conceptual teardown sequence for a nested test
environment.

---

# 20. Important Distinction — Cleanup vs Failure

The final system state showed:

```text id="3g3g8x"
No active RAID50 arrays
```

This was expected because the laboratory was intentionally dismantled.

It does **not** indicate:

```text id="gcsj87"
RAID50 failure
```

The RAID50 configuration had already passed all planned validation
scenarios and had returned to a healthy state before cleanup.

Therefore:

```text id="i4f2lq"
Final RAID health before cleanup
→ HEALTHY

Final RAID state after cleanup
→ NO LAB ARRAY ACTIVE BY DESIGN
```

---

# 21. Final Cleanup Result

```text id="l2n6jp"
LAB CLEANUP RESULT: PASS
```

The temporary RAID50 environment was successfully dismantled.

The cleanup confirmed:

```text id="u3qv1h"
[✓] Filesystem unmounted

[✓] md50 stopped

[✓] md51 stopped

[✓] md52 stopped

[✓] Lab loop devices detached

[✓] Temporary lab workspace removed

[✓] No RAID50 lab arrays remain active

[✓] Existing host storage preserved
```

---

# 22. Final RAID50 Laboratory Lifecycle

The complete fresh RAID50 laboratory lifecycle was:

```text id="wz0hwg"
Create temporary storage
        ↓
Create md51 RAID 5
        ↓
Create md52 RAID 5
        ↓
Create md50 RAID 0
        ↓
Create ext4 filesystem
        ↓
Create test data
        ↓
Capture SHA-256 baseline
        ↓
Test single md51 failure
        ↓
Rebuild md51
        ↓
Test single md52 failure
        ↓
Rebuild md52
        ↓
Test distributed failure
        ↓
Rebuild md51
        ↓
Rebuild md52
        ↓
Final health validation
        ↓
Final data-integrity validation
        ↓
Unmount filesystem
        ↓
Stop RAID arrays
        ↓
Detach loop devices
        ↓
Remove temporary lab
        ↓
Cleanup PASS
```

---

# 23. Final Evidence Status

The RAID50 laboratory now has a complete documented lifecycle:

```text id="j0s3j2"
01_Lab_Setup.md
        ↓
02_RAID50_Creation_and_Healthy_State.md
        ↓
03_Single_Member_Failure_md51.md
        ↓
04_Single_Member_Failure_md52.md
        ↓
05_Distributed_Member_Failure.md
        ↓
06_Recovery_and_Rebuild.md
        ↓
07_Data_Integrity_Validation.md
        ↓
08_Final_Healthy_State.md
        ↓
09_Lab_Cleanup.md
```

The documentation records both the operational test evidence and the
intentional teardown of the test environment.

---

# 24. Final Conclusion

The RAID50 laboratory was successfully completed and cleaned up.

The fresh final run validated:

```text id="q7fb82"
RAID50 creation
+
Healthy operation
+
Single-member failure in md51
+
Single-member failure in md52
+
Distributed member failure
+
RAID 5 rebuild
+
Filesystem availability
+
Data accessibility
+
SHA-256 data integrity
+
Final healthy-state verification
+
Controlled cleanup
```

After the final validation, the temporary RAID50 arrays and loop-backed
test devices were intentionally removed.

The host system was left without the temporary RAID50 laboratory
configuration active.

