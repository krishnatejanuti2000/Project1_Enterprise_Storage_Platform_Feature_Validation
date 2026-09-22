# RAID 50 — Final Healthy State Evidence

## 1. Objective

The objective of this phase was to verify that the RAID50 environment had
returned to a fully healthy and redundant state after completing all
failure and recovery tests.

This validation was performed **before laboratory cleanup**.

The final hierarchy was:

```text
                         RAID 50
                            |
                           md50
                         RAID 0
                       /       \
                      /         \
                   md51         md52
                  RAID 5       RAID 5
```

Expected final condition:

```text
md51 → healthy
md52 → healthy
md50 → healthy
Filesystem → mounted
Test data → readable
SHA-256 → matches baseline
Failed members → 0
```

---

# 2. Final Recovered RAID50 Topology

After all failure and rebuild operations, the final component membership
was:

```text
md51
 ├── /dev/loop18
 ├── /dev/loop11
 └── /dev/loop12
```

and:

```text
md52
 ├── /dev/loop19
 ├── /dev/loop14
 └── /dev/loop15
```

The top-level RAID 0 contained:

```text
md50
 ├── md51
 └── md52
```

Complete topology:

```text
                           md50
                         RAID 0
                       /       \
                      /         \
                   md51         md52
                  RAID 5       RAID 5
                 /  |  \      /  |  \
                /   |   \    /   |   \
           loop18 loop11 loop12 loop19 loop14 loop15
```

---

# 3. Final `/proc/mdstat` Validation

The final RAID status was checked using:

```bash
cat /proc/mdstat
```

The final output was:

```text
md60 : active raid0 md52[1] md51[0]
      4182016 blocks super 1.2 512k chunks

md52 : active raid5 loop19[5] loop14[4] loop15[3]
      2093056 blocks ...

md51 : active raid5 loop18[5] loop11[1] loop12[2]
      2093056 blocks ...

unused devices: <none>
```

For the RAID50 final state, the important component conditions were:

```text
md51 → [3/3] [UUU]
md52 → [3/3] [UUU]
md50 → active
```

No failed member remained in either RAID 5 component.

---

# 4. md51 Final State

The first RAID 5 component was:

```text
md51
```

Final members:

```text
loop18
loop11
loop12
```

Final state:

```text
[3/3] [UUU]
```

The component had returned to full operational capacity.

---

# 5. md52 Final State

The second RAID 5 component was:

```text
md52
```

Final members:

```text
loop19
loop14
loop15
```

Final state:

```text
[3/3] [UUU]
```

The second component had also returned to full operational capacity.

---

# 6. md50 Final State

The top-level RAID device was:

```text
/dev/md50
```

RAID level:

```text
RAID 0
```

The two component arrays were:

```text
md51
md52
```

The final top-level state was clean and active.

The RAID50 hierarchy was therefore fully restored:

```text
md50
 ├── md51 → clean
 └── md52 → clean
```

---

# 7. md50 Detailed Validation

The top-level array was inspected using:

```bash
sudo mdadm --detail /dev/md50
```

The important final conditions were:

```text
State            : clean
Active Devices   : 2
Working Devices  : 2
Failed Devices   : 0
```

The two active members were:

```text
md51 → role 0
md52 → role 1
```

The final array size remained:

```text
4182016 blocks
```

with:

```text
512K chunk size
```

---

# 8. Final md51 Detailed Validation

The first component was inspected:

```bash
sudo mdadm --detail /dev/md51
```

Final condition:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

Final members:

```text
loop18
loop11
loop12
```

This confirmed that the first RAID 5 failure domain had completely
recovered.

---

# 9. Final md52 Detailed Validation

The second component was inspected:

```bash
sudo mdadm --detail /dev/md52
```

Final condition:

```text
State            : clean
Active Devices   : 3
Working Devices  : 3
Failed Devices   : 0
Spare Devices    : 0
```

Final members:

```text
loop19
loop14
loop15
```

This confirmed that the second RAID 5 failure domain had completely
recovered.

---

# 10. Final Filesystem State

The RAID50 filesystem remained mounted at:

```text
/mnt/raid50
```

Mount validation:

```bash
mount | grep raid50
```

Final mount:

```text
/dev/md50 on /mnt/raid50 type ext4 (rw,relatime,stripe=256)
```

This confirmed that the logical RAID50 filesystem remained accessible
after all failure and recovery operations.

---

# 11. Final Filesystem Capacity

Capacity was verified using:

```bash
df -h /mnt/raid50
```

Final result:

```text
/dev/md50  3.9G  1.1M  3.7G  1% /mnt/raid50
```

The filesystem therefore remained usable with the expected logical
capacity.

---

# 12. Final Directory Validation

The filesystem contents were checked:

```bash
ls -lh /mnt/raid50
```

The expected test files remained present:

```text
testfile_01.txt
testfile_02.txt
testfile_03.txt
```

The files remained accessible after all RAID failure and recovery tests.

---

# 13. Final File Read Validation

Each test file was read:

```bash
cat /mnt/raid50/testfile_01.txt
```

```bash
cat /mnt/raid50/testfile_02.txt
```

```bash
cat /mnt/raid50/testfile_03.txt
```

The expected file contents remained intact.

Therefore:

```text
Filesystem access
        +
File access
        +
File content validation
```

all passed.

---

# 14. Final SHA-256 Validation

The final integrity check was performed using:

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

These values matched the healthy-state baseline.

Therefore:

```text
Healthy baseline
        =
Final post-recovery values
```

for all three test files.

---

# 15. Final RAID Health Summary

```text
md51
→ RAID 5
→ 3/3 active
→ 0 failed
→ clean

md52
→ RAID 5
→ 3/3 active
→ 0 failed
→ clean

md50
→ RAID 0
→ 2/2 active
→ 0 failed
→ clean
```

Overall:

```text
RAID50 → HEALTHY
```

---

# 16. Final Storage Stack

The final validated storage stack was:

```text
Application / User
        ↓
Filesystem
        ↓
ext4
        ↓
/dev/md50
        ↓
RAID 0
        ↓
┌───────────────┐
│               │
md51           md52
RAID 5         RAID 5
│               │
├─ loop18       ├─ loop19
├─ loop11       ├─ loop14
└─ loop12       └─ loop15
```

Every layer required for the tested filesystem was operational.

---

# 17. Failure Testing Completed Before Final State

The final healthy state was reached only after completing the planned
fresh-run failure scenarios.

These included:

```text
1. md51 single-member failure
2. md52 single-member failure
3. Distributed failure:
   one member in md51
   +
   one member in md52
```

Each scenario was followed by:

```text
Replacement
+
Rebuild
+
RAID validation
+
Filesystem validation
+
Data-integrity validation
```

---

# 18. Replacement Member History

The final recovered members were the result of the following replacement
sequence:

| Failure             | Component | Failed Member | Replacement |
| ------------------- | --------- | ------------- | ----------- |
| Single failure      | md51      | loop10        | loop16      |
| Single failure      | md52      | loop13        | loop17      |
| Distributed failure | md51      | loop16        | loop18      |
| Distributed failure | md52      | loop17        | loop19      |

Final active membership:

```text
md51:
loop18
loop11
loop12

md52:
loop19
loop14
loop15
```

---

# 19. Final Validation Matrix

| Validation Area           | Result |
| ------------------------- | ------ |
| md51 health               | PASS   |
| md52 health               | PASS   |
| md50 health               | PASS   |
| Failed devices            | 0      |
| Active component members  | 3 + 3  |
| Top-level members         | 2      |
| Filesystem mounted        | PASS   |
| Filesystem accessible     | PASS   |
| Test files present        | PASS   |
| Test files readable       | PASS   |
| SHA-256 integrity         | PASS   |
| Final redundancy restored | PASS   |

---

# 20. Final Acceptance Criteria

The RAID50 final healthy state was accepted because:

```text
[✓] md51 returned to [3/3] [UUU]

[✓] md52 returned to [3/3] [UUU]

[✓] md51 Failed Devices = 0

[✓] md52 Failed Devices = 0

[✓] md50 remained active

[✓] md50 Failed Devices = 0

[✓] Filesystem remained mounted

[✓] Filesystem remained accessible

[✓] All test files remained present

[✓] All test files remained readable

[✓] Final SHA-256 values matched baseline

[✓] Full RAID50 redundancy restored
```

---

# 21. Final Test Result

```text
FINAL RAID50 HEALTH RESULT: PASS
```

The RAID50 environment had successfully returned to the intended healthy
configuration after completion of all planned failure and recovery tests.

---

# 22. Final Healthy-State Snapshot

```text
                         RAID 50
                            |
                           md50
                         RAID 0
                     ____/     \____
                    /               \
                   /                 \
                md51                 md52
               RAID 5              RAID 5
               [UUU]               [UUU]
              / |  \              / |  \
             /  |   \            /  |   \
         loop18 11  12       loop19 14  15
```

Logical state:

```text
md51 → clean
md52 → clean
md50 → clean
```

Filesystem:

```text
/dev/md50 → /mnt/raid50
```

Data:

```text
Readable
```

Integrity:

```text
SHA-256 → MATCH
```

---

# 23. Evidence Conclusion

The final healthy-state validation demonstrated that the fresh RAID50
configuration successfully completed its complete failure and recovery
lifecycle.

The final condition was:

```text
RAID members healthy
        +
Component RAID5 arrays healthy
        +
Top-level RAID0 active
        +
Filesystem mounted
        +
Test files accessible
        +
SHA-256 values unchanged
        ↓
FINAL RAID50 STATE: HEALTHY
```

This state represents the final operational condition immediately before
the temporary RAID50 laboratory environment was intentionally cleaned up.

