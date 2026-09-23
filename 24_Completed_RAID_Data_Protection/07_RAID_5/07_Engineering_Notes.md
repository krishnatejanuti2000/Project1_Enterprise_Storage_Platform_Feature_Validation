# RAID 5 — Engineering Notes

## 1. Engineering Purpose of RAID 5

RAID 5 was designed to address the major architectural limitation of RAID 4:

```text
RAID 4
    ↓
Dedicated parity disk
    ↓
Concentrated parity workload
    ↓
Potential parity bottleneck
````

RAID 5 changes the parity placement architecture:

```text
RAID 5
    ↓
Distributed parity
    ↓
Parity workload distributed across members
    ↓
No permanently dedicated parity disk
```

The important engineering point is:

> RAID 5 solves the dedicated parity-disk bottleneck of RAID 4, but it does not eliminate the cost of maintaining parity.

---

# 2. Distributed Parity — Engineering View

In RAID 5, parity is distributed across the member disks.

Conceptually:

```text
Stripe 1 → D D D P
Stripe 2 → D D P D
Stripe 3 → D P D D
Stripe 4 → P D D D
```

The parity position rotates.

Therefore, over multiple stripes, each disk participates in both:

* Data storage
* Parity storage

This distributes parity-related workload instead of concentrating it permanently on one disk.

---

# 3. Parity Calculation

RAID 5 uses XOR parity.

For a stripe:

```text
P = A XOR B XOR C XOR D
```

XOR has an important reconstruction property:

```text
A XOR B XOR C XOR D XOR P = 0
```

Therefore, if one component is missing, it can be recovered by XORing the remaining components.

For example:

```text
B = A XOR C XOR D XOR P
```

This mathematical property is the foundation of RAID 5 single-member recovery.

---

# 4. Data Reconstruction

When a data member is unavailable, the RAID layer reconstructs the missing block from the surviving blocks in the same stripe.

Conceptually:

```text
Surviving Data
      +
Parity
      ↓
     XOR
      ↓
Missing Data
```

This reconstruction is performed logically for the requested data during degraded operation and physically during rebuild when the replacement member is populated.

---

# 5. Reconstruction vs Rebuild

These terms must be kept separate.

## Reconstruction

```text
Question:
"What should the missing block contain?"
```

The RAID layer calculates the missing information.

Example:

```text
Missing B

B = A XOR C XOR D XOR P
```

## Rebuild

```text
Question:
"How do we restore the failed member?"
```

The RAID layer:

```text
Reads surviving information
        ↓
Reconstructs missing blocks
        ↓
Writes reconstructed blocks
        ↓
Restores replacement member
```

Therefore:

```text
Reconstruction
→ Calculation

Rebuild
→ Restoration
```

---

# 6. Healthy Read Path

When the requested data exists on an available member:

```text
Application
    ↓
Filesystem
    ↓
RAID Layer
    ↓
Correct RAID Member
    ↓
Data
```

Parity does not normally need to participate in a healthy direct read.

This is important when comparing RAID 5 read behavior with degraded operation.

---

# 7. Degraded Read Path

When one member has failed:

```text
Application
    ↓
Filesystem
    ↓
RAID Layer
    ↓
Requested block belongs to failed member
    ↓
Read surviving stripe members
    ↓
XOR reconstruction
    ↓
Return reconstructed data
```

The RAID layer therefore performs additional work for data that was located on the failed member.

This is one reason degraded operation can have different performance characteristics from healthy operation.

---

# 8. Small Write — Read-Modify-Write

A partial stripe update creates a parity-maintenance problem.

Suppose:

```text
Old B → New B
```

The existing parity was calculated using `Old B`.

The controller can calculate:

```text
New P = Old P XOR Old B XOR New B
```

The conceptual operation is:

```text
Read Old B
Read Old P
Calculate New P
Write New B
Write New P
```

This is the commonly described:

```text
2 Reads + 2 Writes
```

RAID 5 small-write path.

---

# 9. Why RMW Creates Overhead

The application may only want to change a small amount of data.

However, the RAID layer must maintain redundancy.

Therefore:

```text
Small Application Write
        ↓
Parity must remain correct
        ↓
Additional RAID operations
```

The additional operations can increase:

* I/O activity
* Latency
* Write amplification at the RAID layer
* Controller workload

This is why small random write workloads are an important RAID 5 performance consideration.

---

# 10. Reconstruct-Write

Instead of reading the old data and old parity, the RAID layer can reconstruct the new parity from the unchanged data and new data.

Example:

```text
A → unchanged
B → New B
C → unchanged
```

Then:

```text
New P = A XOR New B XOR C
```

The conceptual path becomes:

```text
Read unchanged data
        ↓
Combine with new data
        ↓
Calculate new parity
        ↓
Write new data
        ↓
Write new parity
```

The appropriate write strategy depends on the amount and pattern of data being modified.

---

# 11. Full-Stripe Write

A full-stripe write provides all new data blocks for the data portion of the stripe.

Example:

```text
New A
New B
New C
New D
```

The RAID layer can directly calculate:

```text
New P = New A XOR New B XOR New C XOR New D
```

There is no requirement to read the old data or old parity merely to calculate the new parity.

Conceptually:

```text
Complete New Data
        ↓
Parity Calculation
        ↓
New Data + New Parity
        ↓
RAID Members
```

This makes full-stripe writes more efficient from a parity-maintenance perspective than small partial writes.

---

# 12. RAID 5 Write Performance

The important engineering distinction is:

```text
Small Partial Write
→ Additional parity-related work


Full-Stripe Write
→ Parity can be calculated from new data
```

Therefore RAID 5 workload characteristics matter.

A workload dominated by small random writes can experience more parity-related overhead than a workload that produces efficient full-stripe writes.

---

# 13. RAID 5 Fault Tolerance

RAID 5 provides:

```text
One disk-equivalent of redundancy
```

Therefore:

```text
One member failure
        ↓
Array can operate degraded
```

But:

```text
Second member failure
        ↓
Redundancy requirement exceeded
```

RAID 5 should therefore not be interpreted as protection against arbitrary numbers of disk failures.

---

# 14. Degraded Operation

After one member failure:

```text
Healthy
   ↓
Member Failure
   ↓
Degraded
```

The RAID layer can continue providing data.

For data located on the failed member:

```text
Surviving members
        ↓
Reconstruction
        ↓
Requested data
```

However, the array no longer has its original redundancy.

---

# 15. Rebuild Engineering

Once a replacement member is available:

```text
Failed Member
      ↓
Replacement Member
      ↓
Rebuild
```

The RAID layer processes the array and reconstructs the information that belonged to the failed member.

Conceptually:

```text
Stripe 1 → reconstruct → write
Stripe 2 → reconstruct → write
Stripe 3 → reconstruct → write
...
Stripe N → reconstruct → write
```

The process continues until the replacement member is restored.

---

# 16. Rebuild Workload

A rebuild creates additional workload on the surviving members.

During rebuild:

```text
Application I/O
      +
Rebuild reads
      +
Parity calculations
      +
Replacement writes
```

Therefore rebuild can affect:

* Read latency
* Write latency
* Throughput
* Overall application performance

The exact impact depends on the RAID implementation, storage devices, workload, and rebuild controls.

---

# 17. Rebuild Window

The rebuild window is an important reliability consideration.

Lifecycle:

```text
HEALTHY
   ↓
FAILURE
   ↓
DEGRADED
   ↓
REBUILD
   ↓
HEALTHY
```

During:

```text
DEGRADED
+
REBUILD
```

the system has reduced redundancy.

Therefore:

> The objective is to restore redundancy as safely and efficiently as possible.

---

# 18. Second Failure During Rebuild

Consider:

```text
Disk A → Failed
        ↓
Rebuild starts
        ↓
Disk B → Failed
```

RAID 5 has only one disk-equivalent of redundancy.

The second failure can therefore exceed the available redundancy.

Engineering principle:

> RAID 5 is particularly vulnerable to an additional member failure while already degraded.

---

# 19. URE and Rebuild Risk

URE means:

```text
Unrecoverable Read Error
```

A RAID 5 rebuild requires reading surviving members.

Conceptually:

```text
Failed Member
      ↓
Need surviving data
      ↓
Read surviving members
      ↓
Reconstruct missing blocks
```

If a required surviving block cannot be read:

```text
Required Block
      ↓
URE
      ↓
Reconstruction input unavailable
      ↓
Affected stripe may not be reconstructable
```

Therefore URE is an important consideration during large RAID 5 rebuilds.

---

# 20. Why Rebuild Size Matters

Larger member capacity generally means more information must be processed during rebuild.

Conceptually:

```text
Larger Member
      ↓
More blocks to reconstruct
      ↓
More reads
      ↓
More reconstruction work
      ↓
Longer rebuild
      ↓
Longer reduced-redundancy window
```

Therefore storage engineers should consider rebuild characteristics when selecting RAID levels for large-capacity storage.

---

# 21. Hot Spare Engineering

A hot spare is a standby disk reserved for recovery.

Architecture:

```text
Active RAID Members
        +
Standby Hot Spare
```

After a member failure:

```text
Member Failure
      ↓
RAID Degraded
      ↓
Hot Spare Available
      ↓
Rebuild Target
      ↓
Rebuild
```

The hot spare can reduce the delay between member failure and rebuild initiation.

However:

```text
RAID 5 fault tolerance = 1 member
```

remains unchanged.

---

# 22. Hot Spare Does Not Equal Additional Redundancy

Consider:

```text
RAID 5:
D1 D2 D3 D4

Hot Spare:
D5
```

The hot spare is not normally part of the active RAID data/parity layout.

Therefore it should not be counted as:

```text
"RAID 5 can tolerate two failures."
```

Instead:

```text
One active member can fail
        ↓
Hot spare can become recovery target
        ↓
Rebuild
```

---

# 23. Capacity Engineering

For equal-sized members:

```text
Usable Capacity = (N - 1) × Member Capacity
```

The one-member equivalent is used for parity.

Example:

```text
6 × 4 TB
```

Raw:

```text
24 TB
```

Parity:

```text
4 TB equivalent
```

Usable:

```text
20 TB
```

Therefore RAID 5 has approximately one member's capacity as parity overhead.

---

# 24. Unequal Member Sizes

For a simple RAID 5 arrangement using equal member regions, the smallest member constrains the usable member size.

Example:

```text
4 TB
4 TB
6 TB
6 TB
```

The RAID configuration effectively uses approximately:

```text
4 TB per member
```

The additional capacity on the larger members is not automatically usable by that simple RAID configuration.

Engineering consideration:

> Matching member sizes avoids wasting capacity and simplifies capacity planning.

---

# 25. Chunk Size

Chunk size determines how much contiguous data is assigned to a member before RAID moves to the next member.

Example:

```text
Chunk = 512 KiB
```

Conceptually:

```text
Chunk 1 → Disk 1
Chunk 2 → Disk 2
Chunk 3 → Disk 3
...
```

Chunk size influences the relationship between:

* Sequential access
* Random access
* Stripe utilization
* I/O pattern

Chunk size and stripe are not the same concept.

---

# 26. Stripe

A stripe is the complete set of corresponding chunks across the RAID members.

For a four-member RAID 5:

```text
Data
Data
Data
Parity
```

together form one stripe.

The parity position changes across stripes.

Therefore:

```text
Chunk
→ Portion assigned to one member

Stripe
→ Complete RAID stripe
```

---

# 27. RAID 5 and Workload Characteristics

RAID 5 is influenced strongly by workload type.

### Read-oriented workload

Healthy direct reads can be efficient because data can be read directly from the member containing it.

### Small random writes

Can incur parity-related overhead.

### Full-stripe writes

Can calculate parity directly from new data.

### Degraded workload

Can require reconstruction for requests targeting the failed member.

### Rebuild workload

Adds additional reads, calculations, and writes.

Therefore RAID selection should consider workload characteristics rather than capacity alone.

---

# 28. RAID 4 vs RAID 5 — Engineering Perspective

```text
RAID 4

Data → Data → Data → Data → Dedicated Parity
                                  ↑
                            Concentrated load
```

versus:

```text
RAID 5

Stripe 1 → D D D P
Stripe 2 → D D P D
Stripe 3 → D P D D
Stripe 4 → P D D D
```

Engineering change:

```text
Dedicated parity
        ↓
Distributed parity
```

Result:

```text
Parity workload is distributed
```

But:

```text
Parity maintenance still exists
```

---

# 29. RAID 5 Reliability Model

The basic reliability model is:

```text
Normal
  ↓
One member fails
  ↓
Degraded
  ↓
Rebuild
  ↓
Redundancy restored
```

The risk increases when:

```text
Rebuild takes longer
        ↓
Longer exposure to another failure
```

This is why RAID level selection must consider:

* Member capacity
* Number of members
* Workload
* Rebuild time
* Failure probability
* URE behavior
* Availability requirements

---

# 30. RAID 5 Creation — Engineering Command Reference

Linux software RAID 5 can be created with `mdadm`.

Example:

```bash
sudo mdadm --create --verbose /dev/md0 \
  --level=5 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

The important parameters are:

```text
--create
→ Create the RAID array

--verbose
→ Show creation details

/dev/md0
→ RAID device name

--level=5
→ RAID 5

--raid-devices=4
→ Four active RAID members

/dev/sdb /dev/sdc /dev/sdd /dev/sde
→ Member devices
```

The exact device names must always be verified before execution.

---

# 31. RAID 5 Verification Commands

Check the RAID state:

```bash
cat /proc/mdstat
```

Detailed RAID information:

```bash
sudo mdadm --detail /dev/md0
```

Inspect block devices:

```bash
lsblk
```

More detailed block-device information:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

Monitor recovery:

```bash
watch -n 2 cat /proc/mdstat
```

---

# 32. RAID 5 Filesystem Layer

After the RAID array is ready:

```text
Physical Disks
      ↓
RAID 5
      ↓
/dev/md0
      ↓
Filesystem
      ↓
Mount Point
      ↓
Application
```

For example:

```bash
sudo mkfs.ext4 /dev/md0
```

Then:

```bash
sudo mkdir -p /mnt/raid5
sudo mount /dev/md0 /mnt/raid5
```

The filesystem is created on the RAID device rather than separately on each RAID member.

---

# 33. Failure and Replacement Engineering

When a member fails:

```text
Healthy
   ↓
Member Failure
   ↓
Degraded
```

The RAID state should be inspected before changing anything:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

A known failed member can be marked:

```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdX
```

A failed member can be removed:

```bash
sudo mdadm --manage /dev/md0 --remove /dev/sdX
```

A suitable replacement can be added:

```bash
sudo mdadm --manage /dev/md0 --add /dev/sdX
```

Then monitor:

```bash
watch -n 2 cat /proc/mdstat
```

Finally verify:

```bash
sudo mdadm --detail /dev/md0
```

Device identity must always be verified before using `--fail`, `--remove`, or `--add`.

---

# 34. Engineering Mental Model

The complete RAID 5 engineering model:

```text
                 RAID 5
                    |
        +-----------+-----------+
        |                       |
 Block-Level Striping    Distributed Parity
        |                       |
        +-----------+-----------+
                    |
                   XOR
                    |
          Single-Member Protection
                    |
       +------------+------------+
       |                         |
    Healthy                    Failure
       |                         |
   Direct Read              Degraded Read
       |                         |
   Normal Write           Reconstruction
       |                         |
       |                    Replacement
       |                         |
       |                      Rebuild
       |                         |
       +------------+------------+
                    |
             Redundancy Restored
```

---

# 35. Engineering Takeaways

1. RAID 5's major architectural change from RAID 4 is **distributed parity**.
2. Distributed parity removes the permanently dedicated parity disk.
3. Distributed parity does not eliminate parity calculations.
4. Distributed parity does not eliminate parity-update overhead.
5. XOR provides the mathematical basis for parity and single-member reconstruction.
6. Healthy direct reads normally do not require parity.
7. Degraded reads may require reconstruction.
8. Small writes can require Read-Modify-Write.
9. RMW commonly involves two reads and two writes.
10. Full-stripe writes can calculate parity directly from new data.
11. RAID 5 provides one disk-equivalent of redundancy.
12. Reconstruction is calculation of missing information.
13. Rebuild is restoration of a failed member.
14. A rebuild adds workload to surviving members.
15. A longer rebuild creates a longer reduced-redundancy window.
16. UREs are an important consideration during rebuild.
17. A hot spare can reduce recovery delay but does not increase RAID 5 fault tolerance.
18. Capacity is approximately `(N - 1) × member size` for equal-sized members.
19. Unequal member sizes can waste capacity in a simple RAID configuration.
20. Chunk size and stripe are different concepts.
21. RAID 5 suitability depends on workload, capacity, rebuild characteristics, and availability requirements.

