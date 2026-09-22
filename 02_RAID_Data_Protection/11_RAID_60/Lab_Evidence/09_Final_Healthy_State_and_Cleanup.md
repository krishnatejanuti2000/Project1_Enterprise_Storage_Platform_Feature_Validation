# RAID 60 — Final Healthy State and Lab Cleanup Evidence

## 1. Objective

The objective of this phase was to:

```text id="r7gk7h"
1. Verify the final healthy RAID60 state
2. Verify filesystem and data integrity
3. Verify all RAID6 component members were restored
4. Capture the final operational state
5. Safely dismantle the temporary RAID60 laboratory
6. Verify no laboratory RAID or loop devices remained active
```

The cleanup was performed only after all planned RAID60 failure and
recovery scenarios had completed successfully.

---

# 2. Final Healthy RAID60 Topology Before Cleanup

After all recovery operations, the RAID60 hierarchy was:

```text id="8c7b3j"
                           md60
                         RAID 0
                       /       \
                      /         \
                   md61         md62
                  RAID 6       RAID 6
```

Final members:

```text id="b3zjgd"
md61:
loop25
loop24
loop20
loop13

md62:
loop27
loop26
loop17
loop16
```

The complete topology was:

```text id="l5x0b9"
                           RAID 60
                              |
                             md60
                           RAID 0
                         /       \
                        /         \
                     md61         md62
                    RAID 6       RAID 6
                   / | | \      / | | \
              loop25 24 20 13 loop27 26 17 16
```

---

# 3. Final `/proc/mdstat` Validation

The final RAID state was checked using:

```bash id="8i9w6n"
cat /proc/mdstat
```

The captured final state was:

```text id="q17h8h"
md60 : active raid0 md62[1] md61[0]
      4182016 blocks super 1.2 512k chunks

md62 : active raid6 loop27[5] loop26[4] loop17[3] loop16[2]
      2093056 blocks ...

md61 : active raid6 loop25[5] loop24[4] loop20[6] loop13[3]
      2093056 blocks ...

unused devices: <none>
```

The key health indicators were:

```text id="t5uz7p"
md61 → [4/4] [UUUU]
md62 → [4/4] [UUUU]
md60 → active
```

No failed member remained in either component RAID6 array.

---

# 4. Final md61 State

The first RAID6 component was:

```text id="7dcb1j"
md61
```

Final members:

```text id="1ky8d4"
loop25
loop24
loop20
loop13
```

The final state was:

```text id="tkobgy"
[4/4] [UUUU]
```

This confirmed that all four expected members were active.

---

# 5. Final md62 State

The second RAID6 component was:

```text id="r43b9h"
md62
```

Final members:

```text id="y1x8a4"
loop27
loop26
loop17
loop16
```

The final state was:

```text id="n6yrh3"
[4/4] [UUUU]
```

All four expected members were active.

---

# 6. Final md60 State

The top-level RAID device was:

```text id="26xw5f"
/dev/md60
```

RAID level:

```text id="f1h7ve"
RAID 0
```

The component arrays were:

```text id="2ofp6g"
md61
md62
```

The final top-level configuration was:

```text id="vfpj64"
md60
→ active
→ md61 active
→ md62 active
```

---

# 7. Final md61 Detailed Validation

The first component was inspected using:

```bash id="y2y6e5"
sudo mdadm --detail /dev/md61
```

Final expected conditions:

```text id="r3lq2p"
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
Spare Devices    : 0
```

Final member state:

```text id="x4o9rc"
loop25 → active
loop24 → active
loop20 → active
loop13 → active
```

---

# 8. Final md62 Detailed Validation

The second component was inspected:

```bash id="t11s8z"
sudo mdadm --detail /dev/md62
```

Final expected conditions:

```text id="8u4y9x"
State            : clean
Active Devices   : 4
Working Devices  : 4
Failed Devices   : 0
Spare Devices    : 0
```

Final member state:

```text id="p3if4c"
loop27 → active
loop26 → active
loop17 → active
loop16 → active
```

---

# 9. Final md60 Detailed Validation

The parent array was inspected:

```bash id="g9y7vr"
sudo mdadm --detail /dev/md60
```

Final conditions:

```text id="b8a0s2"
State            : clean
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 0
```

Top-level members:

```text id="8f4nq9"
md61 → role 0
md62 → role 1
```

---

# 10. Final RAID Capacity

The top-level RAID60 logical device reported:

```text id="9e8lf4"
4182016 blocks
```

which corresponded to approximately:

```text id="y2n2qp"
3.99 GiB
```

The component RAID6 arrays each reported:

```text id="w7rf1w"
2093056 blocks
```

with a:

```text id="f2z1u0"
512K chunk size
```

---

# 11. Final Filesystem State

The filesystem remained:

```text id="m2d6it"
ext4
```

on:

```text id="wd3p5w"
/dev/md60
```

mounted at:

```text id="dkjpht"
/mnt/raid60
```

Mount validation:

```bash id="y4rwzs"
mount | grep raid60
```

The final mount was:

```text id="s1ld8e"
/dev/md60 on /mnt/raid60 type ext4 (rw,relatime,stripe=256)
```

---

# 12. Final Filesystem Capacity

Filesystem capacity was verified using:

```bash id="i8p5k7"
df -h /mnt/raid60
```

Final result:

```text id="3f6cw4"
/dev/md60  3.9G  1.1M  3.7G  1% /mnt/raid60
```

The filesystem remained fully usable before cleanup.

---

# 13. Final Directory Validation

The RAID60 filesystem contents were checked:

```bash id="b7y2w8"
ls -lh /mnt/raid60
```

The expected files were present:

```text id="vxh4d8"
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

---

# 14. Final File Read Validation

The three test files were read:

```bash id="h9j9c0"
cat /mnt/raid60/testfile_01.txt
cat /mnt/raid60/testfile_02.txt
cat /mnt/raid60/testfile_03.txt
```

All test files remained readable after completion of all failure and
rebuild scenarios.

---

# 15. Final SHA-256 Validation

The final data-integrity check was performed using:

```bash id="p5p1kf"
sha256sum /mnt/raid60/testfile_*.txt
```

Final values:

```text id="l2d9q3"
testfile_01.txt
ef8b994b4eaa76583e14ada0b7b18aed7e5623a831ebd549f09745f64ca2255c

testfile_02.txt
4ba4b039185bc6fe0f670e848c1c2796d2a35741dca74f576c8179e625050689

testfile_03.txt
2923e5735562db666ae0ea364096db080e9f35957d4b07422d71f739a009f588
```

These values matched the original healthy-state baseline.

Therefore:

```text id="u7xg5k"
Healthy baseline
      =
Final post-recovery checksum
```

for all three tested files.

---

# 16. Final Failure/Recovery Coverage

Before entering cleanup, the RAID60 laboratory had completed:

```text id="8f6g5w"
[✓] Single-member failure in md61
[✓] Two-member failure in md61
[✓] Single-member failure in md62
[✓] Distributed 1+1 failure
[✓] Maximum 2+2 failure
```

Recovery had been completed for every tested scenario.

---

# 17. Final RAID Health Summary

```text id="0j1p2a"
md61
→ RAID 6
→ [4/4] [UUUU]
→ Failed Devices = 0
→ clean

md62
→ RAID 6
→ [4/4] [UUUU]
→ Failed Devices = 0
→ clean

md60
→ RAID 0
→ active
→ Failed Devices = 0
→ clean
```

Overall:

```text id="0gqk3v"
RAID60 FINAL STATE: HEALTHY
```

---

# 18. Final Acceptance Criteria Before Cleanup

The environment was accepted as fully recovered because:

```text id="q5g9xf"
[✓] md61 healthy

[✓] md62 healthy

[✓] md60 active

[✓] No failed RAID members

[✓] All expected RAID6 members restored

[✓] Filesystem mounted

[✓] Filesystem accessible

[✓] Test files present

[✓] Test files readable

[✓] Final SHA-256 matched baseline

[✓] All planned failure scenarios completed

[✓] All planned recovery scenarios completed
```

Only after these conditions were satisfied was cleanup started.

---

# 19. Cleanup Objective

The purpose of cleanup was to safely remove the temporary RAID60
laboratory.

The cleanup sequence was:

```text id="my4x4l"
Final healthy RAID60
        ↓
Unmount filesystem
        ↓
Stop md60
        ↓
Stop md61
        ↓
Stop md62
        ↓
Detach loop10-loop27
        ↓
Remove ~/raid60_lab
        ↓
Verify /proc/mdstat
        ↓
Verify loop mappings
        ↓
Verify lsblk
```

---

# 20. Unmount RAID60 Filesystem

The filesystem mounted at:

```text id="o9t6ul"
/mnt/raid60
```

was unmounted.

The purpose was to ensure that no filesystem process continued using
`/dev/md60` before stopping the RAID hierarchy.

The teardown order was therefore:

```text id="j1ozz4"
Filesystem
    ↓
md60
    ↓
md61 / md62
    ↓
Loop devices
```

---

# 21. Stop Top-Level md60

After unmounting the filesystem, the top-level RAID device was stopped:

```bash id="d3dkfv"
sudo mdadm --stop /dev/md60
```

The top-level RAID0 device was intentionally removed from active
operation.

---

# 22. Stop md61

The first RAID6 component was stopped:

```bash id="h9f2mj"
sudo mdadm --stop /dev/md61
```

The component had already been fully recovered and verified healthy.

---

# 23. Stop md62

The second RAID6 component was then stopped:

```bash id="m3j3xh"
sudo mdadm --stop /dev/md62
```

All three temporary RAID devices had now been intentionally stopped:

```text id="wjs4na"
md60 → stopped
md61 → stopped
md62 → stopped
```

---

# 24. Verify md Subsystem After RAID Stop

The md subsystem was checked:

```bash id="fylj9p"
cat /proc/mdstat
```

The resulting state showed no active laboratory arrays:

```text id="0x6ukc"
Personalities : [raid0] [raid4] [raid5] [raid6]
unused devices: <none>
```

This was the expected result after intentional teardown.

Important distinction:

```text id="4jdu8y"
No active RAID60 array
        ≠
RAID60 failure
```

The RAID60 laboratory had already completed successfully.

---

# 25. Laboratory Loop-Device Inventory

During the experiment, the laboratory used loop devices from:

```text id="6b52sp"
loop10
through
loop27
```

Initial members:

```text id="1gfq5l"
loop10 → disk1.img
loop11 → disk2.img
loop12 → disk3.img
loop13 → disk4.img
loop14 → disk5.img
loop15 → disk6.img
loop16 → disk7.img
loop17 → disk8.img
```

Replacement members used during recovery:

```text id="kby5xk"
loop18 → replacement1.img
loop19 → replacement2.img
loop20 → replacement3.img
loop21 → replacement4.img
loop22 → replacement5.img
loop23 → replacement6.img
loop24 → replacement7.img
loop25 → replacement8.img
loop26 → replacement9.img
loop27 → replacement10.img
```

---

# 26. Detach Laboratory Loop Devices

After all RAID arrays were stopped, the temporary loop devices were
detached:

```text id="wu0a2n"
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
loop20
loop21
loop22
loop23
loop24
loop25
loop26
loop27
```

The important teardown rule was:

```text id="e9wl1v"
Stop RAID
      ↓
Detach loop devices
```

This prevents the backing files from being disconnected while still in
active RAID use.

---

# 27. Verify Loop-Device Cleanup

Loop mappings were checked using:

```bash id="s0n9aw"
sudo losetup -a
```

The laboratory mappings for:

```text id="ye7j0e"
loop10
through
loop27
```

were no longer present.

Only the normal pre-existing host loop devices remained.

---

# 28. Verify Block-Device State

The block-device inventory was checked:

```bash id="j9rqgp"
lsblk
```

The temporary RAID60 loop-backed topology was no longer present.

The normal host storage devices remained.

This confirmed that the laboratory teardown did not intentionally modify
the existing host storage configuration.

---

# 29. Remove Temporary RAID60 Workspace

The temporary laboratory directory was:

```text id="l6k447"
~/raid60_lab
```

After RAID devices and loop devices had been dismantled, the temporary
workspace and test-image files were removed.

The images were no longer required because all final evidence had already
been captured in the project documentation.

---

# 30. Verify Workspace Cleanup

The temporary workspace was verified as removed:

```text id="1g2xj9"
~/raid60_lab
→ removed
```

This ensured that the temporary RAID60 image files did not remain on the
host.

---

# 31. Final `/proc/mdstat` Verification

The md subsystem was checked again:

```bash id="6k1x50"
cat /proc/mdstat
```

Final result:

```text id="1rx7w2"
Personalities : [raid0] [raid4] [raid5] [raid6]
unused devices: <none>
```

No RAID60 laboratory arrays remained active.

---

# 32. Final `losetup` Verification

The loop-device state was checked:

```bash id="hs3cs9"
sudo losetup -a
```

The temporary RAID60 mappings were absent.

Only the pre-existing host loop devices remained.

---

# 33. Final `lsblk` Verification

The host block-device layout was checked one final time:

```bash id="bc0s1a"
lsblk
```

The temporary RAID60 devices were no longer part of the active storage
topology.

The host storage environment was left without the temporary RAID60
laboratory configuration.

---

# 34. Cleanup Verification Matrix

| Cleanup Item                             | Result |
| ---------------------------------------- | ------ |
| RAID60 filesystem unmounted              | PASS   |
| md60 stopped                             | PASS   |
| md61 stopped                             | PASS   |
| md62 stopped                             | PASS   |
| loop10 detached                          | PASS   |
| loop11 detached                          | PASS   |
| loop12 detached                          | PASS   |
| loop13 detached                          | PASS   |
| loop14 detached                          | PASS   |
| loop15 detached                          | PASS   |
| loop16 detached                          | PASS   |
| loop17 detached                          | PASS   |
| loop18 detached                          | PASS   |
| loop19 detached                          | PASS   |
| loop20 detached                          | PASS   |
| loop21 detached                          | PASS   |
| loop22 detached                          | PASS   |
| loop23 detached                          | PASS   |
| loop24 detached                          | PASS   |
| loop25 detached                          | PASS   |
| loop26 detached                          | PASS   |
| loop27 detached                          | PASS   |
| Temporary `raid60_lab` workspace removed | PASS   |
| No laboratory md arrays active           | PASS   |
| Existing host loop devices preserved     | PASS   |

---

# 35. Cleanup vs RAID Failure

The final system intentionally showed:

```text id="sm5t0p"
No active RAID60 laboratory arrays
```

This occurred because:

```text id="v5h8qz"
The laboratory was intentionally dismantled.
```

The RAID60 configuration had already been validated as healthy before
cleanup.

Therefore the final no-array state must be interpreted as:

```text id="4d8n4m"
Controlled laboratory teardown
```

and not:

```text id="l2v7ed"
RAID60 failure
```

---

# 36. Complete RAID60 Laboratory Lifecycle

The fresh final laboratory lifecycle was:

```text id="m4u7tb"
Create temporary image files
        ↓
Attach loop devices
        ↓
Create md61 RAID 6
        ↓
Create md62 RAID 6
        ↓
Create md60 RAID 0
        ↓
Create ext4 filesystem
        ↓
Create test data
        ↓
Capture SHA-256 baseline
        ↓
Single md61 failure
        ↓
md61 rebuild
        ↓
Two-member md61 failure
        ↓
Sequential md61 rebuild
        ↓
Single md62 failure
        ↓
md62 rebuild
        ↓
Distributed 1+1 failure
        ↓
Recover md61
        ↓
Recover md62
        ↓
Maximum 2+2 failure
        ↓
Recover md61 member 1
        ↓
Recover md61 member 2
        ↓
Recover md62 member 1
        ↓
Recover md62 member 2
        ↓
Final healthy-state validation
        ↓
Final data-integrity validation
        ↓
Unmount filesystem
        ↓
Stop RAID arrays
        ↓
Detach loop devices
        ↓
Remove temporary workspace
        ↓
Final host verification
```

---

# 37. Overall RAID60 Laboratory Result

```text id="3l2nff"
RAID60 LAB RESULT: PASS
```

The fresh final run successfully demonstrated:

```text id="jq14rv"
RAID60 creation
+
Healthy operation
+
Single-member RAID6 failure
+
Two-member RAID6 failure
+
Distributed failure
+
Maximum 2+2 failure
+
Sequential rebuild
+
Filesystem availability
+
Data accessibility
+
SHA-256 integrity
+
Final healthy-state validation
+
Controlled cleanup
```

---

# 38. Final Engineering Conclusion

The RAID60 laboratory demonstrated the importance of analyzing nested
RAID systems by their component failure domains.

The critical validated condition was:

```text id="r8n5sx"
md61 → 2 failed members
md62 → 2 failed members
```

while:

```text id="7a8c4p"
md61 remained operational
md62 remained operational
md60 remained active
filesystem remained accessible
test files remained readable
SHA-256 remained unchanged
```

The four failed members were subsequently recovered successfully.

Final pre-cleanup condition:

```text id="r3v3is"
md61 → [4/4] [UUUU]
md62 → [4/4] [UUUU]
md60 → active
```

The temporary RAID60 laboratory was then intentionally dismantled and
removed.

The final system was left without the temporary RAID60 test environment
active.

