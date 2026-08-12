# RAID 5

## 1. Overview

RAID 5 is a fault-tolerant RAID level that uses:

- Block-level striping
- Distributed parity
- XOR-based parity
- Single-disk fault tolerance

The major architectural purpose of RAID 5 is to overcome the **dedicated parity-disk bottleneck of RAID 4**.

RAID 5 does **not eliminate parity calculation or parity-update overhead**. Instead, it distributes parity across the member disks so that no single disk is permanently responsible for all parity operations.

---

# 2. Why RAID 5 Was Introduced

## RAID 4 Architecture

RAID 4 uses:

```text
Disk 1 → Data
Disk 2 → Data
Disk 3 → Data
Disk 4 → Data
Disk 5 → Dedicated Parity
````

All parity information is stored on one dedicated parity disk.

This can create:

```text
High parity workload
        ↓
Dedicated parity disk
        ↓
Parity bottleneck
```

## RAID 5 Solution

RAID 5 removes the permanently dedicated parity disk.

Instead:

> **Parity blocks are distributed/rotated across all member disks.**

Conceptually:

```text
Stripe 1 → D D D D P
Stripe 2 → D D D P D
Stripe 3 → D D P D D
Stripe 4 → D P D D D
Stripe 5 → P D D D D
```

Therefore:

```text
RAID 4
→ Dedicated parity

RAID 5
→ Distributed parity
```

The dedicated-parity bottleneck is addressed, but parity calculation and parity-update overhead still exist.

---

# 3. RAID 5 Architecture

RAID 5 uses:

```text
Block-level striping
+
Distributed parity
```

A RAID 5 stripe contains data chunks and one parity chunk.

Example:

```text
Disk 1 → Data A
Disk 2 → Data B
Disk 3 → Data C
Disk 4 → Data D
Disk 5 → Parity P
```

The parity position rotates across different stripes.

Therefore, a disk can contain:

```text
Data in one stripe
```

and:

```text
Parity in another stripe
```

There is no permanently dedicated parity disk.

---

# 4. Distributed Parity

The fundamental architectural change from RAID 4 to RAID 5 is **distributed parity**.

Example:

```text
Stripe 1 → Parity on Disk 5
Stripe 2 → Parity on Disk 4
Stripe 3 → Parity on Disk 3
Stripe 4 → Parity on Disk 2
Stripe 5 → Parity on Disk 1
```

This prevents one disk from being permanently responsible for parity operations.

Important distinction:

```text
Parity calculation
→ How parity is calculated

Parity placement
→ Where parity is stored
```

RAID 5 changes the parity placement architecture.

It does not eliminate XOR parity calculation.

---

# 5. RAID 5 Stripe

A RAID stripe represents the corresponding set of chunks across the RAID member disks.

Example:

```text
Disk 1 → A
Disk 2 → B
Disk 3 → C
Disk 4 → D
Disk 5 → P
```

Together:

```text
A + B + C + D + P
```

represent one RAID stripe.

One member position contains parity for that stripe, while the remaining positions contain data.

The parity position rotates across stripes.

---

# 6. XOR Parity

RAID 5 uses XOR to calculate parity.

Example:

```text
A = 1011
B = 0101
C = 1100
```

Parity:

```text
P = A XOR B XOR C
```

Calculation:

```text
1011 XOR 0101 = 1110

1110 XOR 1100 = 0010
```

Therefore:

```text
P = 0010
```

---

# 7. Data Reconstruction

If one data block is lost, RAID 5 can reconstruct it using the surviving data and parity.

Example:

```text
A = 1011
B = missing
C = 1100
P = 0010
```

The missing block can be calculated as:

```text
B = A XOR C XOR P
```

Calculation:

```text
1011 XOR 1100 = 0111

0111 XOR 0010 = 0101
```

Therefore:

```text
Recovered B = 0101
```

The controller uses the surviving blocks from the same stripe to reconstruct the missing block.

---

# 8. Parity Reconstruction

If the parity block itself is lost while the data blocks are available, the controller can recalculate parity.

```text
P = A XOR B XOR C
```

Example:

```text
A = 1011
B = 0101
C = 1100

P = 0010
```

In this case:

```text
Data is still available
Parity is regenerated
```

---

# 9. Normal Read

In a healthy RAID 5 array, when the requested data block is available, the controller normally reads that data directly from the appropriate member.

Conceptually:

```text
Application
    ↓
RAID Controller
    ↓
Requested Data Member
    ↓
Data
```

The controller does not normally need to read the parity block for a healthy direct read.

---

# 10. Degraded Read

When one RAID member fails:

```text
Disk 1 → Data     ✅
Disk 2 → Failed   ❌
Disk 3 → Data     ✅
Disk 4 → Data     ✅
Disk 5 → Parity   ✅
```

The array becomes degraded.

If the requested block belonged to the failed member, the controller reconstructs it.

Conceptually:

```text
Surviving data
+
Parity
      ↓
     XOR
      ↓
Missing data
      ↓
Application
```

---

# 11. Small / Partial Write

A partial write changes only part of a stripe.

Example:

```text
Old B → New B
```

The existing parity was calculated using `Old B`.

Therefore, when `B` changes, parity must also be updated.

---

# 12. Read-Modify-Write (RMW)

For a small partial write, the controller can use:

```text
Old Data
+
Old Parity
+
New Data
```

to calculate the new parity.

For example:

```text
New P = Old P XOR Old B XOR New B
```

### RMW sequence

```text
1. Read Old B
2. Read Old Parity
3. Calculate New Parity
4. Write New B
5. Write New Parity
```

The commonly described RAID 5 small-write path is:

```text
2 Reads
+
2 Writes
=
4 I/O operations
```

This is commonly called the:

> **RAID 5 small-write penalty**

---

# 13. Why RMW Is Required

Suppose:

```text
P = A XOR B XOR C
```

If:

```text
Old B → New B
```

the old parity no longer represents the new data.

Therefore:

```text
Old Parity
+
Old Data
+
New Data
        ↓
New Parity
```

The parity must be updated whenever the corresponding data changes.

---

# 14. Reconstruct-Write

Another write strategy is reconstruct-write.

If only one data block changes, the controller can read the unchanged data blocks and combine them with the new data.

Example:

```text
A → unchanged
B → New B
C → unchanged
```

The controller calculates:

```text
New P = A XOR New B XOR C
```

Then:

```text
Write New B
Write New P
```

Important:

> **Reconstruct-write in the write path is different from reconstructing missing data after a disk failure.**

---

# 15. Full-Stripe Write

A full-stripe write provides the complete new data portion of the stripe.

Example:

```text
New A
New B
New C
New D
```

The controller already has all the new data required to calculate parity.

Therefore:

```text
New P = New A XOR New B XOR New C XOR New D
```

The controller does not need to read the old data or old parity to calculate the new parity.

Conceptually:

```text
New Data
    ↓
XOR
    ↓
New Parity
    ↓
Write New Data + New Parity
```

This can avoid the old-data/old-parity read path associated with RMW.

---

# 16. RAID 5 Write Behavior

## Partial Write

```text
Read old data
Read old parity
Calculate new parity
Write new data
Write new parity
```

## Full-Stripe Write

```text
Complete new data
      ↓
Calculate new parity
      ↓
Write new data + parity
```

Therefore, RAID 5 write behavior depends heavily on how much of the stripe is being updated.

---

# 17. RAID 5 Parity Overhead

RAID 5 does not eliminate parity overhead.

The architectural improvement is:

```text
RAID 4
→ Dedicated parity disk
→ Parity workload concentrated

RAID 5
→ Distributed parity
→ Parity workload distributed
```

But RAID 5 still has:

```text
Parity calculation
+
Parity update
+
Additional write-path I/O
```

especially for small partial writes.

---

# 18. RAID 5 Fault Tolerance

RAID 5 provides:

> **Single-member fault tolerance.**

Example:

```text
Disk 1 → Healthy
Disk 2 → Failed
Disk 3 → Healthy
Disk 4 → Healthy
Disk 5 → Healthy
```

The array can normally continue operating in degraded mode.

Therefore:

```text
1 member failure
→ Within RAID 5 protection

2 member failures
→ Beyond RAID 5 protection
```

A second failure while the array is rebuilding is particularly dangerous because redundancy has not yet been restored.

---

# 19. RAID 5 Degraded State

The normal lifecycle is:

```text
HEALTHY
   ↓
Member Failure
   ↓
DEGRADED
   ↓
Replacement
   ↓
REBUILD
   ↓
HEALTHY
```

While degraded:

```text
Data may remain accessible
Redundancy is reduced
Another member failure is dangerous
```

---

# 20. Reconstruction vs Rebuild

These concepts must be kept separate.

## Reconstruction

Reconstruction means:

> **Calculating missing information.**

Example:

```text
Missing B
   ↓
A XOR C XOR P
   ↓
Recovered B
```

## Rebuild

Rebuild means:

> **Restoring the failed RAID member onto a replacement member.**

Conceptually:

```text
Failed Member
      ↓
Replacement Disk
      ↓
Reconstruct Missing Blocks
      ↓
Write Blocks to Replacement
      ↓
Member Restored
```

Therefore:

```text
Reconstruction
→ Calculation of missing information

Rebuild
→ Restoration of a failed member
```

---

# 21. RAID 5 Rebuild

Suppose one RAID member fails and a replacement disk becomes available.

The controller rebuilds the replacement member stripe by stripe.

Conceptually:

```text
Surviving Members
       ↓
Read Required Blocks
       ↓
Reconstruct Missing Block
       ↓
Write to Replacement
       ↓
Continue Through Array
       ↓
Rebuild Complete
```

The surviving disks therefore perform additional work during the rebuild.

---

# 22. Rebuild Workload

During rebuild, the storage system may handle:

```text
Normal Application I/O
        +
Rebuild Reads
        +
Parity Calculations
        +
Replacement Writes
```

Therefore:

> **Rebuild can affect application performance.**

---

# 23. Rebuild Window

The array remains exposed to additional failure while redundancy has not yet been restored.

```text
Disk Failure
     ↓
Degraded Array
     ↓
Rebuild
     ↓
Redundancy Restored
```

A longer rebuild means a longer period of reduced redundancy.

---

# 24. Second Failure During Rebuild

Example:

```text
Disk 2 → FAILED
       ↓
Rebuild starts
       ↓
Disk 4 → FAILED
```

RAID 5 has only one disk-equivalent of redundancy.

Therefore:

```text
First failure
→ Recoverable

Second failure before redundancy is restored
→ Protection exceeded
```

---

# 25. URE — Unrecoverable Read Error

URE means:

> **Unrecoverable Read Error**

During a RAID 5 rebuild, surviving members must be read to reconstruct the failed member.

Example:

```text
A → readable
B → failed member
C → URE
P → readable
```

To reconstruct `B`:

```text
B = A XOR C XOR P
```

But if `C` cannot be read, the required information for that stripe is unavailable.

Therefore:

```text
Failed member
+
Required unreadable block
        ↓
Affected stripe may not be reconstructable
```

Important:

> A URE does not automatically mean that the entire RAID 5 array is lost.

The effect depends on the affected block and RAID/controller behavior.

---

# 26. Hot Spare

A hot spare is an additional standby disk that is not normally part of the active RAID data set.

Example:

```text
RAID 5:
D1 D2 D3 D4

Hot Spare:
D5
```

If a member fails:

```text
Member Failure
      ↓
RAID Degraded
      ↓
Hot Spare Available
      ↓
Hot Spare Used as Recovery Target
      ↓
Rebuild
```

A hot spare can reduce the time before recovery begins.

Important:

> **A hot spare does not increase RAID 5 fault tolerance from one disk to two disks.**

It is a recovery resource.

---

# 27. Capacity

For equal-sized RAID 5 members:

```text
Usable Capacity = (N - 1) × Drive Capacity
```

Example:

```text
6 × 4 TB
```

Raw capacity:

```text
6 × 4 TB = 24 TB
```

Parity overhead:

```text
1 × 4 TB = 4 TB equivalent
```

Usable capacity:

```text
(6 - 1) × 4 TB = 20 TB
```

Therefore:

```text
Raw      = 24 TB
Parity   = 4 TB equivalent
Usable   = 20 TB
```

---

# 28. Unequal Disk Sizes

For a simple RAID 5 configuration using equal-sized member regions, the effective member size is constrained by the smallest member.

Example:

```text
Disk 1 = 4 TB
Disk 2 = 4 TB
Disk 3 = 6 TB
Disk 4 = 6 TB
```

Effective member size:

```text
4 TB
```

The larger disks therefore contain capacity that is not used by that RAID member configuration.

---

# 29. Minimum Number of Drives

RAID 5 requires a minimum of:

```text
3 drives
```

Conceptually:

```text
Data
Data
Parity
```

The parity position can rotate across the members.

---

# 30. Chunk Size vs Stripe

These are different concepts.

## Chunk

A chunk is the amount of contiguous data assigned to one RAID member before moving to the next member.

## Stripe

A stripe is the complete set of corresponding chunks across the RAID members.

Therefore:

```text
Chunk
→ Portion on one member

Stripe
→ Complete RAID stripe across members
```

---

# 31. RAID 5 Performance

## Healthy Reads

Healthy direct reads can be efficient because the requested data can be read directly from the appropriate member.

## Small Random Writes

Small writes can require:

```text
Old data read
Old parity read
Parity calculation
New data write
New parity write
```

Therefore parity maintenance can increase write cost.

## Full-Stripe Writes

Full-stripe writes can calculate parity directly from the new data and avoid the old-data/old-parity read path.

---

# 32. RAID 5 vs RAID 4

| Feature                     | RAID 4      | RAID 5      |
| --------------------------- | ----------- | ----------- |
| Striping                    | Block-level | Block-level |
| Parity                      | Dedicated   | Distributed |
| Permanent parity disk       | Yes         | No          |
| Dedicated parity bottleneck | Yes         | Addressed   |
| XOR parity                  | Yes         | Yes         |
| Fault tolerance             | 1 disk      | 1 disk      |
| Parity-update overhead      | Yes         | Yes         |
| Minimum drives              | 3           | 3           |

The fundamental architectural difference:

```text
RAID 4
→ Dedicated parity

RAID 5
→ Distributed parity
```

---

# 33. Creating RAID 5 Using Linux mdadm

Linux software RAID can be created using `mdadm`.

## Step 1 — Verify Available Disks

Before creating the array:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

Identify the disks that will be used for RAID.

> **Never use the operating-system disk accidentally.**

---

## Step 2 — Verify mdadm

```bash
mdadm --version
```

---

## Step 3 — Create RAID 5

For example, using four disks:

```bash
sudo mdadm --create --verbose /dev/md0 \
  --level=5 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Meaning:

```text
/dev/md0
→ RAID device

--level=5
→ RAID 5

--raid-devices=4
→ Four active RAID members

/dev/sdb /dev/sdc /dev/sdd /dev/sde
→ RAID member devices
```

Depending on the `mdadm` version and configuration, you may be prompted to confirm creation and/or enable a write-intent bitmap.

---

# 34. Verify RAID Creation

Check:

```bash
cat /proc/mdstat
```

Detailed information:

```bash
sudo mdadm --detail /dev/md0
```

The output can show:

```text
Raid Level
Raid Devices
Active Devices
Working Devices
Failed Devices
Spare Devices
Layout
Chunk Size
Metadata Version
Rebuild / Recovery Status
```

---

# 35. Monitor RAID Recovery

Immediately after creation, the array may perform initial synchronization/recovery.

Monitor it with:

```bash
watch -n 2 cat /proc/mdstat
```

Or:

```bash
cat /proc/mdstat
```

A degraded/recovery representation can appear temporarily during initial synchronization.

Wait for synchronization/recovery to complete before treating the array as fully synchronized.

---

# 36. Create a Filesystem on RAID 5

After the RAID device is ready, a filesystem can be created on `/dev/md0`.

Example using ext4:

```bash
sudo mkfs.ext4 /dev/md0
```

> The filesystem is created on the RAID device, not individually on each RAID member.

---

# 37. Mount the RAID 5 Filesystem

Create a mount point:

```bash
sudo mkdir -p /mnt/raid5
```

Mount:

```bash
sudo mount /dev/md0 /mnt/raid5
```

Verify:

```bash
lsblk -f
```

and:

```bash
df -h /mnt/raid5
```

---

# 38. Verify RAID and Filesystem Together

Check RAID:

```bash
sudo mdadm --detail /dev/md0
```

Check filesystem:

```bash
lsblk -f
```

Check mounted capacity:

```bash
df -h
```

The storage stack is conceptually:

```text
Physical Disks
      ↓
RAID Members
      ↓
/dev/md0
      ↓
Filesystem
      ↓
Mount Point
      ↓
Application
```

---

# 39. RAID 5 Failure and Replacement Commands

If a RAID member fails, first inspect the array:

```bash
cat /proc/mdstat
```

Then:

```bash
sudo mdadm --detail /dev/md0
```

To mark a known failed member:

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdX
```

To remove a failed member:

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdX
```

After a suitable replacement disk is available, add it:

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

Monitor rebuild:

```bash
watch -n 2 cat /proc/mdstat
```

Verify final state:

```bash
sudo mdadm --detail /dev/md0
```

> Always verify the exact failed/replacement device before running `--fail`, `--remove`, or `--add`.

---

# 40. RAID 5 Cleanup

When intentionally removing a test RAID array, first unmount the filesystem:

```bash
sudo umount /mnt/raid5
```

Stop the RAID device:

```bash
sudo mdadm --stop /dev/md0
```

If the disks are being reused, RAID metadata should also be removed carefully.

For example:

```bash
sudo mdadm --zero-superblock /dev/sdb
sudo mdadm --zero-superblock /dev/sdc
sudo mdadm --zero-superblock /dev/sdd
sudo mdadm --zero-superblock /dev/sde
```

> Only run cleanup commands against disks that are confirmed to be dedicated RAID-test devices.

---

# 41. Important RAID 5 Commands — Quick Reference

## Inspect disks

```bash
lsblk
```

## Detailed disk information

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## Check mdadm

```bash
mdadm --version
```

## Create RAID 5

```bash
sudo mdadm --create --verbose /dev/md0 \
  --level=5 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

## Check RAID status

```bash
cat /proc/mdstat
```

## Detailed RAID information

```bash
sudo mdadm --detail /dev/md0
```

## Monitor recovery

```bash
watch -n 2 cat /proc/mdstat
```

## Create filesystem

```bash
sudo mkfs.ext4 /dev/md0
```

## Mount

```bash
sudo mount /dev/md0 /mnt/raid5
```

## Fail a member

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdX
```

## Remove a member

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdX
```

## Add a replacement

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

## Stop array

```bash
sudo mdadm --stop /dev/md0
```

---

# 42. RAID 5 Mental Model

```text
                 RAID 5
                    |
       +------------+------------+
       |                         |
Block-level striping      Distributed parity
       |                         |
       +------------+------------+
                    |
                  XOR
                    |
          Single-disk protection
                    |
        +-----------+-----------+
        |                       |
     Healthy                 Failure
        |                       |
   Normal read              Degraded
   Normal write                 |
        |                 Reconstruction
        |                       |
        |                  Replacement
        |                       |
        |                     Rebuild
        |                       |
        +-----------+-----------+
                    |
                 Healthy
```
# 43. Hot-Plug and Hot-Swap

Hot-plug and hot-swap are important operational concepts in
storage systems, especially when replacing failed RAID
members while the system remains online.

---

## 43.1 Hot-Plug

Hot-plug means that a storage device can be connected or
disconnected while the system is powered on and running.

Conceptually:

```text
Server Running
      ↓
Connect / Disconnect Device
      ↓
System Detects Device
```

The key idea is:

> **Hot-plug = connect or disconnect a device while the system is running.**

---

## 43.2 Hot-Swap

Hot-swap means replacing a failed component with a replacement
component while the system continues running.

For a RAID storage system:

```text
RAID Running
      ↓
Disk Fails
      ↓
Remove Failed Disk
      ↓
Insert Replacement Disk
      ↓
RAID Continues Operating
      ↓
Rebuild
```

The key idea is:

> **Hot-swap = replace a failed component while the system remains running.**

---

## 43.3 Hot-Plug vs Hot-Swap

```text
Hot-Plug
    ↓
Connect / Disconnect
while system is running


Hot-Swap
    ↓
Replace a failed component
while system is running
```

A simple way to remember the difference:

```text
PLUG → Connect / Disconnect

SWAP → Replace
```

---

## 43.4 RAID 5 Context

Consider a RAID 5 array:

```text
Disk 1 → Healthy
Disk 2 → Healthy
Disk 3 → Failed
Disk 4 → Healthy
```

If the storage hardware supports hot-swap:

```text
Disk 3 Failed
      ↓
Remove Failed Disk
      ↓
Insert Replacement Disk
      ↓
RAID Rebuild
```

The server can remain operational during the physical
replacement process.

---

## 43.5 Important Engineering Distinction

Software RAID management commands and physical hot-swap
capability are different concepts.

For example:

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdc
sudo mdadm --manage /dev/md0 --remove /dev/sdc
sudo mdadm --manage /dev/md0 --add /dev/sdc
```

These commands demonstrate the RAID software workflow for
failing, removing, and adding a RAID member.

They do **not** by themselves prove that the physical server
hardware supports hot-swapping.

Physical hot-swap capability depends on the storage hardware
and platform supporting safe online insertion and removal.

---

## 43.6 Key Point

```text
mdadm member removal/re-addition
                ≠
physical hot-swap capability
```

Therefore, when validating a storage system, distinguish
between:

```text
Software RAID operation
        and
Physical hot-plug / hot-swap capability
```

Both are related to storage operations, but they are not the
same thing.


---

# 44. Key RAID 5 Points

1. RAID 5 uses **block-level striping**.
2. RAID 5 uses **distributed parity**.
3. RAID 5 uses **XOR parity**.
4. RAID 5 has **no permanently dedicated parity disk**.
5. RAID 5 was introduced to overcome the **dedicated parity-disk bottleneck of RAID 4**.
6. Distributed parity does **not** eliminate parity calculation.
7. Distributed parity does **not** eliminate parity-update overhead.
8. Healthy direct reads normally do not require parity.
9. A missing data block can be reconstructed using surviving data and parity.
10. A missing parity block can be recalculated from surviving data.
11. Small partial writes can use **Read-Modify-Write**.
12. RMW commonly involves **2 reads + 2 writes**.
13. Full-stripe writes can calculate parity directly from the new data.
14. RAID 5 tolerates **one member failure**.
15. Reconstruction means calculating missing information.
16. Rebuild means restoring a failed member onto a replacement member.
17. A hot spare provides a ready recovery resource.
18. A hot spare does not provide an additional RAID fault-tolerance level.
19. A URE during rebuild can prevent reconstruction of an affected stripe if required information cannot be read.
20. A second member failure exceeds RAID 5's normal protection capability.
21. RAID 5 requires at least **3 drives**.
22. Equal-member usable capacity is approximately **(N - 1) × member size**.
23. Unequal member sizes can cause larger disks to have unused capacity.
24. Chunk size and stripe are different concepts.
25. Rebuild activity can affect application performance.
26. The degraded/rebuild window is an important operational risk.
27. Linux `mdadm` can be used to create, monitor, manage, and rebuild software RAID 5 arrays.


