# RAID 5 — Interview Guide

## 1. What is RAID 5?

RAID 5 is a RAID level that uses **block-level striping with distributed parity** and provides **single-disk fault tolerance**.

---

## 2. Why was RAID 5 introduced?

RAID 5 was introduced to overcome the **dedicated parity-disk bottleneck of RAID 4**.

RAID 4 uses one dedicated parity disk:

```text
Data + Data + Data + Data + Dedicated Parity
````

RAID 5 distributes parity across the member disks:

```text
Stripe 1 → D D D P
Stripe 2 → D D P D
Stripe 3 → D P D D
Stripe 4 → P D D D
```

---

## 3. What is the biggest architectural difference between RAID 4 and RAID 5?

```text
RAID 4 → Dedicated parity
RAID 5 → Distributed parity
```

RAID 5 eliminates the permanently dedicated parity disk.

---

## 4. Does RAID 5 eliminate parity overhead?

**No.**

RAID 5 eliminates the dedicated parity-disk bottleneck, but parity calculation and parity-update overhead still exist.

This is particularly important for small writes.

---

## 5. What type of striping does RAID 5 use?

RAID 5 uses:

> **Block-level striping**

---

## 6. What type of parity does RAID 5 use?

RAID 5 uses:

> **XOR-based parity**

---

## 7. What is distributed parity?

Distributed parity means that parity blocks are **spread/rotated across the RAID member disks** rather than being stored permanently on one dedicated parity disk.

---

## 8. Why is distributed parity important?

It prevents the parity workload from being permanently concentrated on one disk.

Therefore:

```text
RAID 4
→ Dedicated parity workload

RAID 5
→ Distributed parity workload
```

---

## 9. Does parity rotate in RAID 5?

Yes.

The parity position changes across stripes.

Conceptually:

```text
Stripe 1 → P on Disk 4
Stripe 2 → P on Disk 3
Stripe 3 → P on Disk 2
Stripe 4 → P on Disk 1
```

The exact placement depends on the RAID implementation and layout.

---

## 10. How is RAID 5 parity calculated?

Using XOR.

For example:

```text
P = A XOR B XOR C
```

---

## 11. Why can RAID 5 reconstruct a missing block?

Because XOR has the property that the missing value can be calculated from the remaining values.

For example:

```text
P = A XOR B XOR C
```

If `B` is missing:

```text
B = A XOR C XOR P
```

---

## 12. What happens when one RAID 5 disk fails?

The array enters a **degraded state**.

The array can normally continue providing data because the missing information can be reconstructed from the surviving members and parity.

---

## 13. How many disk failures can RAID 5 tolerate?

RAID 5 can tolerate:

> **One member failure.**

A second member failure exceeds RAID 5's normal redundancy.

---

## 14. What is a degraded RAID 5 array?

A degraded RAID 5 array is an array in which one member is unavailable but the remaining members still provide enough information to continue operating.

Conceptually:

```text
[UUU_]
```

---

## 15. Can data still be read when RAID 5 is degraded?

Yes.

If the requested data belongs to a surviving member, it can normally be read directly.

If the requested data belongs to the failed member, the RAID layer reconstructs it from the surviving data and parity.

---

## 16. Does a normal RAID 5 read require reading parity?

Normally, **no**.

If the requested data block exists on an available member, the controller can read it directly.

Parity is primarily required for redundancy, degraded reads, reconstruction, and parity maintenance.

---

## 17. What is data reconstruction?

Data reconstruction means calculating a missing data block using the surviving data and parity.

Example:

```text
B = A XOR C XOR P
```

---

## 18. What is parity reconstruction?

Parity reconstruction means recalculating a missing parity block from the surviving data.

Example:

```text
P = A XOR B XOR C
```

---

## 19. What is the difference between reconstruction and rebuild?

### Reconstruction

Calculating missing information.

### Rebuild

Restoring the failed RAID member onto a replacement member.

In short:

```text
Reconstruction
→ Calculation

Rebuild
→ Restoration
```

---

## 20. What is RAID 5 RMW?

RMW means:

> **Read-Modify-Write**

It is commonly used for small partial writes.

Example:

```text
Old B → New B
```

The controller reads:

```text
Old B
Old Parity
```

Then calculates:

```text
New P = Old P XOR Old B XOR New B
```

Then writes:

```text
New B
New P
```

---

## 21. How many I/O operations are commonly associated with RAID 5 RMW?

The commonly described path is:

```text
2 Reads
+
2 Writes
=
4 I/O operations
```

This is called the:

> **RAID 5 small-write penalty**

---

## 22. Why does RAID 5 have a small-write penalty?

Because a small data update requires parity to remain consistent.

Therefore the RAID layer may need to:

```text
Read old data
Read old parity
Calculate new parity
Write new data
Write new parity
```

The application requested a small write, but the RAID layer performs additional operations to maintain redundancy.

---

## 23. What is the RMW parity formula?

For a change from `Old B` to `New B`:

```text
New P = Old P XOR Old B XOR New B
```

---

## 24. What is reconstruct-write?

Reconstruct-write is a write strategy where the RAID layer reads the unchanged data blocks and combines them with the new data to calculate new parity.

Example:

```text
New P = A XOR New B XOR C
```

It is different from reconstructing missing data after a disk failure.

---

## 25. What is a full-stripe write?

A full-stripe write provides all new data blocks belonging to the data portion of a stripe.

The RAID layer can calculate parity directly:

```text
New P = New A XOR New B XOR New C XOR New D
```

It does not need old data or old parity merely to calculate the new parity.

---

## 26. Why are full-stripe writes more efficient for parity maintenance?

Because the complete new data is already available.

Therefore:

```text
Complete New Data
        ↓
XOR
        ↓
New Parity
```

The old-data/old-parity read path required by RMW is avoided.

---

## 27. What is the RAID 5 write penalty?

The RAID 5 write penalty refers to the additional RAID operations required to maintain parity during partial writes.

For the commonly described RMW path:

```text
2 Reads + 2 Writes
```

---

## 28. Does RAID 5 have a write penalty even though parity is distributed?

Yes.

Distributed parity solves the **dedicated parity-disk bottleneck**, but it does not remove the need to calculate and update parity.

---

## 29. What happens during a RAID 5 rebuild?

After a failed member is replaced:

```text
Surviving members
      ↓
Read required blocks
      ↓
Reconstruct missing blocks
      ↓
Write reconstructed blocks
      ↓
Replacement member restored
```

---

## 30. Why does a RAID 5 rebuild affect performance?

Because the surviving members must perform additional work while also serving normal application I/O.

During rebuild:

```text
Normal Application I/O
        +
Rebuild Reads
        +
Parity Calculations
        +
Replacement Writes
```

---

## 31. What is the rebuild window?

The rebuild window is the period between member failure and restoration of full redundancy.

Conceptually:

```text
Failure
   ↓
Degraded
   ↓
Rebuild
   ↓
Healthy
```

During this period, the array has reduced redundancy.

---

## 32. Why is a second failure during rebuild dangerous?

RAID 5 has only one disk-equivalent of redundancy.

Therefore:

```text
First failure
→ Redundancy consumed

Second failure
→ Protection exceeded
```

---

## 33. What is a URE?

URE means:

> **Unrecoverable Read Error**

It means a required block cannot be successfully read after the drive's internal recovery mechanisms are exhausted.

---

## 34. Why is URE important during RAID 5 rebuild?

A rebuild requires reading surviving members.

If a required block cannot be read:

```text
Required surviving block
        ↓
URE
        ↓
Required reconstruction input unavailable
```

The affected stripe may therefore not be reconstructable.

---

## 35. Does one URE automatically mean the entire RAID 5 array is lost?

No.

The effect depends on:

* Which block encountered the error
* Whether that block is required for reconstruction
* RAID/controller behavior
* The condition of the other members

The important engineering concern is that rebuilds require substantial reads from surviving members.

---

## 36. What is a hot spare?

A hot spare is a standby disk reserved for RAID recovery.

Example:

```text
Active RAID Members
+
Hot Spare
```

When a member fails:

```text
Member Failure
      ↓
Degraded RAID
      ↓
Hot Spare
      ↓
Rebuild
```

---

## 37. Does a hot spare increase RAID 5 fault tolerance?

No.

RAID 5 still provides:

```text
One member failure tolerance
```

The hot spare is a recovery resource, not an additional parity level.

---

## 38. What is the minimum number of disks required for RAID 5?

Three disks.

---

## 39. What is the RAID 5 capacity formula?

For equal-sized members:

```text
Usable Capacity = (N - 1) × Member Capacity
```

Example:

```text
6 × 4 TB
```

Raw capacity:

```text
24 TB
```

Usable capacity:

```text
(6 - 1) × 4 TB
= 20 TB
```

One member-equivalent is used for parity.

---

## 40. What happens if RAID 5 disks have different sizes?

The effective member size is constrained by the smallest member in a simple equal-member configuration.

For example:

```text
4 TB
4 TB
6 TB
6 TB
```

The effective member size is approximately:

```text
4 TB
```

The additional capacity of the larger members is not automatically usable by that simple configuration.

---

## 41. What is a chunk?

A chunk is the amount of contiguous data assigned to one RAID member before RAID moves to the next member.

---

## 42. What is a stripe?

A stripe is the complete set of corresponding chunks across the RAID members.

Therefore:

```text
Chunk
→ Portion on one member

Stripe
→ Complete RAID stripe
```

---

## 43. Is chunk size the same as stripe size?

No.

They are different concepts.

```text
Chunk
→ Data portion assigned to one member

Stripe
→ Complete set of member chunks
```

---

## 44. How do you create RAID 5 in Linux?

Using `mdadm`.

Example:

```bash
sudo mdadm --create --verbose /dev/md0 \
  --level=5 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Here:

```text
/dev/md0
→ RAID device

--level=5
→ RAID 5

--raid-devices=4
→ Four RAID members
```

The actual device names must always be verified before execution.

---

## 45. How do you check RAID 5 status in Linux?

```bash
cat /proc/mdstat
```

---

## 46. How do you get detailed RAID 5 information?

```bash
sudo mdadm --detail /dev/md0
```

---

## 47. How do you monitor RAID 5 recovery?

```bash
watch -n 2 cat /proc/mdstat
```

---

## 48. How do you create a filesystem on a RAID 5 array?

After the RAID device is ready:

```bash
sudo mkfs.ext4 /dev/md0
```

The filesystem is created on the RAID device, not individually on each member.

---

## 49. How do you mount the RAID 5 filesystem?

```bash
sudo mkdir -p /mnt/raid5
sudo mount /dev/md0 /mnt/raid5
```

Verify:

```bash
lsblk -f
df -h /mnt/raid5
```

---

## 50. How do you simulate a RAID 5 member failure using mdadm?

First verify the exact member.

Then:

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdX
```

After that:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

The array should show a degraded condition.

---

## 51. How do you remove a failed RAID member?

After confirming that the member is failed:

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdX
```

Always verify the exact device before running the command.

---

## 52. How do you add a replacement member?

After verifying that the replacement disk is suitable:

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

Then monitor:

```bash
cat /proc/mdstat
```

---

## 53. How do you verify that rebuild completed?

Check:

```bash
cat /proc/mdstat
```

and:

```bash
sudo mdadm --detail /dev/md0
```

Verify that:

```text
All required members are active
Failed Devices = 0
```

and the array has returned to a healthy state.

---

# Scenario-Based Questions

## 54. Scenario: RAID 4 has a parity bottleneck. What architectural change does RAID 5 make?

Answer:

> RAID 5 distributes parity across the member disks instead of using a permanently dedicated parity disk. This distributes parity-related workload and eliminates the dedicated parity-disk bottleneck.

---

## 55. Scenario: A user performs a small random write to one block in RAID 5. What happens?

Answer:

> The RAID layer must maintain parity. A common approach is Read-Modify-Write: read the old data and old parity, calculate new parity using the old data, old parity, and new data, then write the new data and new parity.

---

## 56. Scenario: The application performs a full-stripe write. Does the RAID controller need old parity?

Answer:

> No. Because the complete new data stripe is available, the controller can calculate the new parity directly from the new data. It does not need the old parity merely to calculate the new parity.

---

## 57. Scenario: One RAID 5 member fails. Can the application still read data?

Answer:

> Normally yes. Data stored on surviving members can be read directly. Data that belonged to the failed member can be reconstructed using the surviving data and parity.

---

## 58. Scenario: A parity block is lost. Is user data necessarily lost?

Answer:

> No. If the data blocks are available, the parity block can be recalculated using XOR.

---

## 59. Scenario: One member fails and a hot spare is available. What happens?

Answer:

> The array becomes degraded and the hot spare can become the recovery target. The missing member's information is reconstructed and written to the spare during rebuild.

---

## 60. Scenario: A second member fails while RAID 5 is rebuilding. Why is this serious?

Answer:

> RAID 5 has only one disk-equivalent of redundancy. The first failure consumes that redundancy. A second failure can exceed the available protection and may make the array unavailable.

---

## 61. Scenario: Rebuild is very slow. What should you consider?

Answer:

Consider:

* Normal application workload
* Rebuild workload
* Disk performance
* RAID implementation
* Rebuild throttling
* Member capacity
* Disk health
* I/O contention

A slow rebuild is not automatically a RAID failure.

---

## 62. Scenario: A URE occurs during rebuild. What is the concern?

Answer:

> The rebuild requires reading surviving members. If a required block cannot be read, the RAID layer may not have enough information to reconstruct the affected stripe.

---

## 63. Scenario: RAID 5 has four disks of different sizes. What happens to capacity?

Answer:

> In a simple equal-member configuration, the effective member size is constrained by the smallest member. Capacity above that effective size on larger disks may remain unused.

---

# Rapid-Fire Questions

## 64. RAID 5 striping?

**Block-level.**

## 65. RAID 5 parity?

**Distributed XOR parity.**

## 66. Dedicated parity disk?

**No.**

## 67. RAID 5 fault tolerance?

**One member failure.**

## 68. Minimum disks?

**Three.**

## 69. Small-write technique?

**Read-Modify-Write.**

## 70. Common RMW I/O count?

**2 reads + 2 writes.**

## 71. Full-stripe write?

**Calculate parity directly from the new data.**

## 72. Missing data reconstruction?

**XOR surviving data and parity.**

## 73. Missing parity reconstruction?

**XOR surviving data.**

## 74. RAID 5 capacity formula?

```text
(N - 1) × member capacity
```

for equal-sized members.

## 75. RAID 5's main RAID 4 improvement?

**Distributed parity.**

## 76. Does RAID 5 eliminate parity overhead?

**No.**

## 77. RAID 5 rebuild?

**Restore a failed member using reconstructed information.**

## 78. Hot spare?

**Standby recovery disk.**

## 79. URE?

**Unrecoverable Read Error.**

## 80. RAID 5 second failure?

**Beyond normal single-member protection.**

---

# Final Interview Answer

If asked:

> **"Explain RAID 5."**

A strong answer is:

> RAID 5 is a block-level striped RAID architecture that uses distributed XOR parity to provide single-disk fault tolerance. Its major architectural advantage over RAID 4 is that parity is distributed across the member disks instead of being stored on one dedicated parity disk, eliminating the dedicated parity-disk bottleneck. However, parity calculation and update overhead still exist, especially for small writes where Read-Modify-Write can result in two reads and two writes. If one member fails, the array operates in degraded mode and missing data can be reconstructed from the surviving data and parity. When a replacement or hot spare is available, the failed member can be rebuilt. RAID 5 therefore provides efficient capacity and single-member protection, but rebuild time, degraded operation, small-write overhead, and URE exposure are important engineering considerations.

