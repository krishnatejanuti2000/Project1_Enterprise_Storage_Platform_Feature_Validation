# Engineering Notes – RAID Fundamentals

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

Topic: RAID Fundamentals

---

# 1. Engineering Perspective

RAID is not simply a method of combining disks. In enterprise environments, RAID is a storage virtualization technology that abstracts multiple physical drives into one logical storage device while providing specific characteristics such as improved performance, redundancy, or capacity.

Storage engineers design RAID groups based on workload requirements rather than simply maximizing storage space.

---

# 2. Enterprise Design Goals

Before selecting a RAID level, storage engineers evaluate three primary objectives:

- Performance
- Data Protection
- Capacity

Improving one objective often requires compromising another.

Example:

- RAID 0 maximizes performance but provides no redundancy.
- RAID 1 provides redundancy but sacrifices 50% of usable capacity.
- RAID 5 and RAID 6 attempt to balance capacity, performance, and fault tolerance.

---

# 3. Hardware RAID vs Software RAID

## Hardware RAID

Managed by a dedicated RAID controller.

Characteristics:

- Dedicated processor
- Dedicated cache memory
- Battery-backed or flash-backed cache (depending on controller)
- Operating system sees only logical disks

Common enterprise vendors:

- Broadcom / LSI
- Dell PERC
- HPE Smart Array
- Lenovo ThinkSystem RAID

---

## Software RAID

Managed by the operating system.

Linux uses:

```
mdadm
```

Characteristics:

- No dedicated RAID controller required
- Uses CPU resources
- Flexible and widely supported
- Common in Linux servers and cloud environments

---

# 4. RAID Controller Responsibilities

A RAID controller is responsible for:

- Managing RAID groups
- Distributing data across disks
- Monitoring disk health
- Handling disk failures
- Performing rebuild operations
- Presenting logical storage to the operating system

Applications never communicate directly with individual member disks.

---

# 5. Physical Disks vs Logical Disks

Physical disks are actual HDDs or SSDs installed in the server.

After RAID creation, these disks become members of a RAID group.

The operating system interacts only with the logical RAID device.

Example:

```
Application
      │
      ▼
Logical RAID Device
      │
 ┌────┴────┐
 ▼         ▼
Disk 1   Disk 2
```

---

# 6. RAID Group

A RAID Group is a collection of physical disks configured together using a specific RAID level.

Examples:

- RAID 0 Group
- RAID 1 Group
- RAID 5 Group
- RAID 6 Group

The RAID group determines:

- Data placement
- Capacity
- Fault tolerance
- Performance

---

# 7. Raw Capacity vs Usable Capacity

## Raw Capacity

Total capacity of all physical disks.

Example:

```
4 × 1 TB

Raw Capacity = 4 TB
```

---

## Usable Capacity

Capacity available after RAID overhead.

Example:

```
RAID 1

2 × 1 TB

Raw Capacity = 2 TB

Usable Capacity = 1 TB
```

Storage engineers always calculate both values during storage planning.

---

# 8. RAID Selection Strategy

Enterprise RAID selection depends on workload.

Examples:

| Workload | Typical RAID Choice |
|----------|---------------------|
| Temporary Data | RAID 0 |
| Operating System | RAID 1 |
| Database | RAID 10 |
| File Server | RAID 5 |
| Backup Repository | RAID 6 |

There is no universal RAID level suitable for every workload.

---

# 9. Storage Engineering Best Practices

- Select RAID based on workload requirements.
- Always calculate usable capacity before deployment.
- Avoid RAID 0 for critical business data.
- Monitor disk health continuously.
- Replace failed drives promptly in redundant RAID levels.
- Validate RAID configuration after deployment.

---

# 10. Commands Used in Linux Software RAID

Common Linux commands:

```bash
lsblk
```

View block devices.

```bash
cat /proc/mdstat
```

Check RAID status.

```bash
mdadm --detail
```

Display RAID configuration.

```bash
mdadm --examine
```

Display RAID metadata stored on member disks.

These commands are frequently used during validation and troubleshooting.

---

# Key Engineering Takeaways

- RAID abstracts multiple physical disks into one logical storage device.
- RAID design always involves trade-offs between performance, capacity, and data protection.
- Hardware RAID and Software RAID provide similar functionality but differ in implementation.
- Enterprise storage engineers select RAID levels based on workload characteristics rather than disk count alone.
- Linux Software RAID is implemented using the `mdadm` utility.
