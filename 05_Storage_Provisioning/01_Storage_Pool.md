# Storage Pool Validation

## Project

**Project 1 — Enterprise Storage Platform & Feature Validation**

## Module

**05 — Storage Provisioning**

## Submodule

**01 — Storage Pool**

---

# 1. Objective

Validate the Storage Pool layer after RAID and demonstrate:

- Storage Pool creation
- Pool capacity visibility
- Capacity allocation
- Storage Pool expansion
- Free-capacity accounting
- Post-expansion health/state validation

The lab uses **Linux LVM** to reproduce the storage-pool and capacity-management concepts in a controlled environment.

> **Important:** LVM is a lab implementation of the pool/capacity-management layer. It does not imply that enterprise storage arrays generally use LVM internally.

---

# 2. Lab Architecture

## Initial Storage Path

```text
16 Virtual Drives
        ↓
3 × RAID6
        ↓
RAID0 across the 3 RAID6 groups
        ↓
/dev/md124
        ↓
LVM Physical Volume (PV)
        ↓
LVM Volume Group (VG)
        ↓
storage_pool
```

## Pool Expansion Path

```text
Existing RAID60
        ↓
/dev/md124
        ↓
PV1
        ↓
storage_pool
        ↑
        │
PV2
        ↑
/dev/md123
        ↑
RAID6 group4
        ↑
4 New Virtual Drives
```

---

# 3. Storage Layer Responsibilities

The provisioning layers have separate engineering responsibilities:

| Layer | Responsibility |
|---|---|
| RAID | Data protection and fault tolerance |
| Storage Pool | Centralized capacity management |
| Volume | Logical storage allocation |
| LUN | Storage presentation to a host |

The Project 1 provisioning model keeps these layers distinct:

```text
RAID
  ↓
Storage Pool
  ↓
Volume
  ↓
LUN
```

---

# 4. Why LVM Is Used in This Lab

RAID provides a protected block-storage resource.

For this lab, we need another layer that can:

- manage usable capacity
- track allocated capacity
- track free capacity
- support capacity expansion
- allocate storage to logical Volumes

LVM provides that capability.

The lab mapping is:

```text
/dev/md124
    ↓
LVM PV
    ↓
LVM VG
    ↓
Storage Pool
```

Within LVM:

```text
PV = Physical Volume
VG = Volume Group
LV = Logical Volume
```

For this project:

```text
PV → prepared storage resource
VG → Storage Pool
LV → Volume
```

---

# 5. Initial RAID-Backed Capacity

The existing RAID60 device used for the Storage Pool was:

```text
/dev/md124
```

Capacity visible to LVM:

```text
<11.97 GiB
```

At the beginning of the Storage Pool exercise, all of this capacity was free.

---

# 6. Create the Physical Volume

## Command

```bash
sudo pvcreate /dev/md124
```

## Purpose

Initialize `/dev/md124` as an LVM Physical Volume.

Before:

```text
/dev/md124
   ↓
RAID block device
```

After:

```text
/dev/md124
   ↓
LVM Physical Volume
```

## Observed Result

```text
Physical volume "/dev/md124" successfully created.
```

---

# 7. Verify the Physical Volume

## Command

```bash
sudo pvs
```

## Observed Result

```text
PV         VG Fmt  Attr PSize   PFree
/dev/md124    lvm2 ---  11.97g  11.97g
/dev/sda2  rl lvm2 a--  <29.00g     0
```

## Validation

`/dev/md124` was successfully recognized as an LVM PV.

Validation points:

- PV exists
- PV size is approximately 11.97 GiB
- Entire PV capacity is free
- PV is not yet assigned to a VG

The existing `/dev/sda2` belongs to the Rocky Linux OS Volume Group `rl` and was not modified.

---

# 8. Create the Storage Pool

## Command

```bash
sudo vgcreate storage_pool /dev/md124
```

## Purpose

Create an LVM Volume Group named:

```text
storage_pool
```

For this lab, this VG represents the **Storage Pool** layer.

Architecture:

```text
/dev/md124
    ↓
PV
    ↓
storage_pool (VG)
```

## Observed Result

```text
Volume group "storage_pool" successfully created
```

---

# 9. Verify Storage Pool Capacity

## Command

```bash
sudo vgs
```

## Observed Result

```text
VG           #PV #LV #SN Attr   VSize   VFree
rl             1   2   0 wz--n- <29.00g      0
storage_pool   1   0   0 wz--n- <11.97g <11.97g
```

## Validation

The Storage Pool was created correctly.

Initial pool state:

```text
Total capacity  ≈ 11.97 GiB
Allocated       = 0 GiB
Free capacity   ≈ 11.97 GiB
PV count        = 1
LV count        = 0
```

---

# 10. Detailed Storage Pool Verification

## Command

```bash
sudo vgdisplay storage_pool
```

## Important Observed Values

```text
VG Name               storage_pool
VG Access             read/write
VG Status             resizable
Cur LV                0
Cur PV                1
Act PV                1
VG Size               <11.97 GiB
PE Size               4.00 MiB
Total PE              3064
Alloc PE / Size       0 / 0
Free  PE / Size       3064 / <11.97 GiB
```

## Validation

The Storage Pool was:

- accessible as read/write
- resizable
- backed by one active PV
- completely unallocated

LVM uses 4 MiB Physical Extents in this Volume Group.

---

# 11. Allocate Capacity to a Volume

The next objective was to verify that the Storage Pool can allocate capacity to a logical Volume.

## Command

```bash
sudo lvcreate -L 2G -n volume01 storage_pool
```

## Observed Result

```text
Logical volume "volume01" created.
```

The resulting structure:

```text
storage_pool
      ↓
volume01
      ↓
2 GiB
```

---

# 12. Verify the Volume Allocation

## Command

```bash
sudo lvs
```

## Observed Result

```text
LV       VG           Attr       LSize
root     rl           -wi-ao---- <26.00g
swap     rl           -wi-ao----   3.00g
volume01 storage_pool -wi-a-----   2.00g
```

## Validation

The new Volume:

- exists
- belongs to `storage_pool`
- has a size of 2 GiB

The existing `root` and `swap` LVs belong to the operating-system VG `rl` and were not modified.

---

# 13. Detailed Volume Verification

## Command

```bash
sudo lvdisplay /dev/storage_pool/volume01
```

## Important Observed Values

```text
LV Path                /dev/storage_pool/volume01
LV Name                volume01
VG Name                storage_pool
LV Write Access        read/write
LV Status              available
# open                 0
LV Size                2.00 GiB
Current LE             512
Segments               1
Allocation             inherit
```

## Validation

The Volume was:

- available
- read/write
- 2 GiB in size
- not currently opened by another consumer

The Volume creation and allocation were therefore verified successfully.

---

# 14. Verify Pool Capacity After Volume Allocation

The Storage Pool must correctly account for capacity consumed by the new Volume.

## Command

```bash
sudo vgs storage_pool
```

## Observed Result

```text
VG           #PV #LV #SN Attr   VSize   VFree
storage_pool   1   1   0 wz--n- <11.97g <9.97g
```

## Validation

Before Volume creation:

```text
Pool Free ≈ 11.97 GiB
```

After allocating a 2 GiB Volume:

```text
Pool Free ≈ 9.97 GiB
```

This confirms that the Storage Pool correctly accounted for the 2 GiB allocation.

---

# 15. Storage Pool Expansion

The next validation objective was to increase the capacity of the existing Storage Pool without changing the size of the existing Volume.

## Design

The original 16 virtual drives were already committed:

```text
12 drives → RAID60
4 drives  → Hot Spares
```

Those drives were left untouched.

Four additional virtual drives were created:

```text
disk17.img → 2 GiB
disk18.img → 2 GiB
disk19.img → 2 GiB
disk20.img → 2 GiB
```

These were used to create a new RAID-backed capacity source.

---

# 16. Create Additional Virtual Drives

The new sparse images were created using:

```bash
sudo truncate -s 2G /opt/storage-lab/virtual-disks/disk17.img
sudo truncate -s 2G /opt/storage-lab/virtual-disks/disk18.img
sudo truncate -s 2G /opt/storage-lab/virtual-disks/disk19.img
sudo truncate -s 2G /opt/storage-lab/virtual-disks/disk20.img
```

Each image was individually verified to have a 2.0G logical size.

---

# 17. Attach the New Images as Block Devices

The new images were attached to loop devices:

```text
disk17.img → /dev/loop16
disk18.img → /dev/loop17
disk19.img → /dev/loop18
disk20.img → /dev/loop19
```

Each loop device was verified as a 2 GiB block device.

Example verification:

```bash
lsblk -o NAME,SIZE,TYPE /dev/loop16
```

Observed:

```text
NAME   SIZE TYPE
loop16   2G loop
```

---

# 18. Create the Additional RAID6 Group

The four new block devices were combined into a new RAID6 group:

```text
/dev/loop16
/dev/loop17
/dev/loop18
/dev/loop19
        ↓
RAID6
        ↓
/dev/md123
```

## Command

```bash
sudo mdadm --create /dev/md/raid6_group4 --level=6 --raid-devices=4 --bitmap=internal --metadata=1.2 /dev/loop16 /dev/loop17 /dev/loop18 /dev/loop19
```

## Observed Result

```text
mdadm: array /dev/md/raid6_group4 started.
```

---

# 19. Verify the Additional RAID Resource

## Command

```bash
cat /proc/mdstat
```

## Observed State

```text
md123 : active raid6 loop19[3] loop18[2] loop17[1] loop16[0]
        4188160 blocks super 1.2 level 6, 512k chunk, algorithm 2 [4/4] [UUUU]
```

## Validation

The new RAID6 group was healthy:

```text
RAID members expected = 4
RAID members active   = 4
State                  = [UUUU]
```

Therefore:

```text
/dev/md123
```

was available as a protected RAID-backed capacity source.

---

# 20. Prepare the New RAID Capacity as PV2

## Command

```bash
sudo pvcreate /dev/md123
```

## Observed Result

```text
Physical volume "/dev/md123" successfully created.
```

The new RAID-backed device was now an LVM PV.

---

# 21. Verify the New PV

## Command

```bash
sudo pvs
```

## Observed Result

```text
PV         VG           Fmt  Attr PSize   PFree
/dev/md123              lvm2 ---    3.99g  3.99g
/dev/md124 storage_pool lvm2 a--  <11.97g <9.97g
/dev/sda2  rl           lvm2 a--  <29.00g     0
```

## Validation

`/dev/md123` was verified as:

```text
PV2
Size ≈ 3.99 GiB
Free ≈ 3.99 GiB
VG   = not assigned yet
```

This is important because `pvcreate` prepared the capacity but did **not** yet expand the Storage Pool.

---

# 22. Expand the Existing Storage Pool

## Command

```bash
sudo vgextend storage_pool /dev/md123
```

## Purpose

Add the new PV to the existing Storage Pool.

Before:

```text
/dev/md124 → PV1
                 ↓
            storage_pool
```

After:

```text
/dev/md124 → PV1 ──┐
                   ├──→ storage_pool
/dev/md123 → PV2 ──┘
```

## Observed Result

```text
Volume group "storage_pool" successfully extended
```

---

# 23. Verify Pool Expansion

## Command

```bash
sudo vgs storage_pool
```

## Observed Result

```text
VG           #PV #LV #SN Attr   VSize  VFree
storage_pool   2   1   0 wz--n- 15.96g 13.96g
```

## Validation

Before expansion:

```text
#PV   = 1
VSize ≈ 11.97 GiB
VFree ≈ 9.97 GiB
```

After expansion:

```text
#PV   = 2
VSize ≈ 15.96 GiB
VFree ≈ 13.96 GiB
```

This proves that the Storage Pool successfully consumed the additional RAID-backed capacity.

---

# 24. Verify Both PVs Belong to the Storage Pool

## Command

```bash
sudo pvs
```

## Observed Result

```text
PV         VG           Fmt  Attr PSize   PFree
/dev/md123 storage_pool lvm2 a--    3.99g  3.99g
/dev/md124 storage_pool lvm2 a--  <11.97g <9.97g
/dev/sda2  rl           lvm2 a--  <29.00g     0
```

## Validation

The Storage Pool now contains:

```text
PV1 → /dev/md124
PV2 → /dev/md123
```

Both are active members of:

```text
storage_pool
```

---

# 25. Post-Expansion Storage Pool Health

## Command

```bash
sudo vgdisplay storage_pool
```

## Important Observed Values

```text
VG Name               storage_pool
VG Access             read/write
VG Status             resizable
Cur LV                1
Cur PV                2
Act PV                2
VG Size               15.96 GiB
PE Size               4.00 MiB
Total PE              4086
Alloc PE / Size       512 / 2.00 GiB
Free  PE / Size       3574 / 13.96 GiB
```

## Validation

The Storage Pool remained healthy after expansion:

- read/write access
- resizable
- two active PVs
- one existing Volume
- approximately 2 GiB allocated
- approximately 13.96 GiB free

---

# 26. Volume Expansion

After validating Pool expansion, the existing Volume was independently expanded.

Initial state:

```text
volume01 = 2 GiB
```

Available pool capacity was sufficient to support expansion.

## Command

```bash
sudo lvextend -L +2G /dev/storage_pool/volume01
```

## Observed Result

```text
Size of logical volume storage_pool/volume01 changed from 2.00 GiB (512 extents) to 4.00 GiB (1024 extents).
Logical volume storage_pool/volume01 successfully resized.
```

---

# 27. Verify Volume Expansion

## Command

```bash
sudo lvs storage_pool/volume01
```

## Observed Result

```text
LV       VG           Attr       LSize
volume01 storage_pool -wi-a----- 4.00g
```

## Validation

The Volume size changed:

```text
2 GiB
 ↓
4 GiB
```

The Storage Pool total capacity itself did not change.

---

# 28. Verify Pool Capacity After Volume Expansion

## Command

```bash
sudo vgs storage_pool
```

## Observed Result

```text
VG           #PV #LV #SN Attr   VSize  VFree
storage_pool   2   1   0 wz--n- 15.96g 11.96g
```

## Validation

Before Volume expansion:

```text
Pool total = 15.96 GiB
Used       = 2 GiB
Free       = 13.96 GiB
```

After adding another 2 GiB to `volume01`:

```text
Pool total = 15.96 GiB
Used       = 4 GiB
Free       = 11.96 GiB
```

This proves that:

- Volume expansion worked.
- Pool total capacity stayed unchanged.
- Pool free capacity decreased by the amount allocated to the Volume.

---

# 29. Pool Expansion vs Volume Expansion

These two operations were intentionally validated separately.

## Pool Expansion

```text
Add new storage capacity
        ↓
Storage Pool becomes larger
        ↓
Existing Volume stays same size
```

Example:

```text
Pool       = 12 GiB → 16 GiB
Volume01   = 2 GiB → 2 GiB
Free space = 10 GiB → 14 GiB
```

## Volume Expansion

```text
Allocate additional pool capacity
        ↓
Existing Volume becomes larger
        ↓
Pool total stays the same
        ↓
Pool free capacity decreases
```

Example:

```text
Pool       = 16 GiB → 16 GiB
Volume01   = 2 GiB → 4 GiB
Free space = 14 GiB → 12 GiB
```

---

# 30. Current Storage State

The current validated architecture is:

```text
Existing RAID60
/dev/md124
      ↓
PV1
      │
      ├──────────────┐
                     ↓
                 storage_pool
                     ↑
      └──────────────┘
      │
New RAID6
/dev/md123
      ↓
PV2

storage_pool
      ↓
volume01
      ↓
4 GiB
```

Current capacity:

```text
Storage Pool = 15.96 GiB
Allocated    = 4.00 GiB
Free         = 11.96 GiB
```

Current PVs:

```text
/dev/md124 → storage_pool
/dev/md123 → storage_pool
```

---

# 31. Validation Summary

| Validation | Result |
|---|---|
| Create PV from RAID60 | PASS |
| Verify PV | PASS |
| Create Storage Pool | PASS |
| Verify Pool capacity | PASS |
| Detailed Pool health | PASS |
| Create Volume | PASS |
| Verify Volume | PASS |
| Verify capacity allocation | PASS |
| Create additional RAID-backed capacity | PASS |
| Create second PV | PASS |
| Expand Storage Pool | PASS |
| Verify Pool expansion | PASS |
| Verify both PVs | PASS |
| Post-expansion Pool health | PASS |
| Expand Volume | PASS |
| Verify Volume expansion | PASS |
| Verify Pool capacity accounting | PASS |

---

# 32. Engineering Conclusions

This lab demonstrated the following storage-engineering relationships:

```text
RAID
 ↓
Protected block capacity

Storage Pool
 ↓
Capacity management

Volume
 ↓
Logical allocation from the pool
```

The most important distinctions validated were:

```text
RAID Expansion
≠
Pool Expansion
≠
Volume Expansion
```

and:

```text
Pool Expansion
→ increases available pool capacity

Volume Expansion
→ consumes pool capacity to increase an existing Volume
```

The exercise also demonstrated the validation mindset of checking the storage state **before and after each configuration change**, rather than relying only on command success messages.

---

# 33. Remaining Storage Pool / Volume Validation

The basic provisioning and capacity-management path is complete.

Broader Project 1 validation still needs to cover scenarios such as:

- Thin Provisioning behavior
- Thick Provisioning behavior
- Boundary conditions
- Allocation failure
- Insufficient pool capacity
- Volume deletion
- Capacity recovery after deletion
- Metadata consistency
- Negative provisioning scenarios
- Failure/recovery behavior

These will be covered as part of the broader **Storage Feature Validation, Negative Testing, Fault Injection, Recovery and RCA** work rather than blocking the main provisioning path.

---

# 34. Next Layer

The next provisioning layer is:

```text
RAID
   ↓
Storage Pool ✅
   ↓
Volume ✅
   ↓
LUN ← NEXT
   ↓
Host Mapping
   ↓
Host Discovery
   ↓
Filesystem
   ↓
Application I/O
```

The Storage Pool and basic Volume lifecycle are therefore established well enough to proceed to **LUN creation and presentation**.
