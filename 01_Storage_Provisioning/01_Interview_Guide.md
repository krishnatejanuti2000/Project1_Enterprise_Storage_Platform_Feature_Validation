# Module 01 – Interview Guide

This interview guide is designed for Storage Validation Engineers, Storage QA Engineers, and Storage Test Engineers with approximately three years of industry experience.

The questions focus on enterprise storage provisioning, validation, troubleshooting, production scenarios, and engineering decision-making.

---

# Section 1 – Frequently Asked Interview Questions

## Q1. Explain the complete storage provisioning workflow from physical disks to application access.

### What the interviewer wants to evaluate

- End-to-end understanding
- Architecture knowledge
- Ability to explain storage logically
- Communication skills

### Expected Answer

Storage provisioning is the process of converting physical storage resources into logical storage that applications can consume.

The workflow is:

```

Physical Disks
↓
RAID Creation
↓
Storage Pool Creation
↓
Volume Creation
↓
LUN Creation
↓
Host Mapping
↓
Host Discovery
↓
Filesystem Creation
↓
Application Usage

```

1. Physical disks are grouped into a RAID set to provide redundancy and performance.
2. One or more RAID groups contribute capacity to a Storage Pool.
3. A Volume is created from the Storage Pool.
4. The Volume is exposed as a LUN.
5. The LUN is mapped to the required host.
6. The host discovers the new storage.
7. The operating system creates a filesystem.
8. Applications start using the storage.

### Interview Tip

Don't simply list the layers.

Explain why each layer exists.

---

## Q2. A customer requests a new 2 TB LUN for a production Oracle database. Walk me through how you would provision it.

### What the interviewer wants to evaluate

- Real-world engineering thinking
- Planning ability
- Validation approach

### Expected Answer

Before provisioning, I would collect the following information:

- Required capacity
- Performance requirements
- Availability requirements
- RAID policy
- Target host
- Operating system
- SAN connectivity
- Backup policy

Provisioning steps:

1. Verify available RAID capacity.
2. Verify Storage Pool health.
3. Create or identify the appropriate Storage Pool.
4. Create a 2 TB Volume.
5. Create a LUN from the Volume.
6. Map the LUN to the Oracle server.
7. Perform a host rescan.
8. Verify the LUN is visible.
9. Confirm multipath configuration.
10. Validate read/write operations.
11. Update documentation.

### Interview Tip

Interviewers appreciate candidates who include validation after configuration.

---

## Q3. What validations do you perform after provisioning storage?

### What the interviewer wants to evaluate

Whether you think like a Validation Engineer instead of just an Administrator.

### Expected Answer

I validate:

Storage Side

- Volume exists
- Correct capacity
- Correct RAID level
- Correct Storage Pool
- Correct Host Mapping
- No storage alerts

Host Side

- Host discovers the LUN
- Correct LUN size
- Multipath functioning correctly
- Filesystem creation succeeds
- Read/Write I/O succeeds

Application Side

- Database or application successfully accesses storage
- No I/O errors
- Expected performance observed

### Interview Tip

Always validate from:

Storage
→ Host
→ Application

rather than stopping after successful provisioning.

---

## Q4. What information do you verify before creating a Storage Pool?

### What the interviewer wants to evaluate

Planning ability.

### Expected Answer

I verify:

- RAID health
- Available capacity
- Drive types
- RAID level
- Controller health
- Cache status
- Firmware compatibility
- Expected workload
- Future capacity growth
- Existing pool utilization

Creating a Storage Pool without verifying these parameters may lead to performance or capacity issues later.

---

## Q5. As a Storage Validation Engineer, what test cases would you design for Storage Provisioning?

### What the interviewer wants to evaluate

- Test design skills
- Validation mindset
- Coverage thinking

### Expected Answer

I would divide the test cases into multiple categories.

### Functional Test Cases

- Create Storage Pool
- Create Volume
- Create LUN
- Map LUN to Host
- Delete LUN
- Expand Volume
- Rename Storage Objects
- Unmap Host

### Positive Test Cases

- Provision supported capacities
- Provision using different RAID levels
- Provision to multiple hosts
- Provision multiple LUNs simultaneously

### Negative Test Cases

- Create Volume larger than available capacity
- Map LUN to unauthorized host
- Duplicate object names
- Invalid LUN IDs
- Provision during degraded RAID

### Boundary Test Cases

- Minimum supported capacity
- Maximum supported capacity
- Maximum LUN count
- Maximum Storage Pools
- Maximum Volumes

### Validation

- Host discovery
- Read/Write verification
- Multipath validation
- Performance validation
- Event log verification

### Interview Tip

Don't say "I'll test provisioning."

Explain **what** you'll test and **why**.

---

## Q6. How would you validate Storage Provisioning after a firmware upgrade?

### What the interviewer wants to evaluate

Regression testing knowledge.

### Expected Answer

After a firmware upgrade, I would execute regression testing.

The validation includes:

- Existing Storage Pools remain healthy
- Existing Volumes remain accessible
- Existing LUN mappings remain intact
- New provisioning works correctly
- Volume expansion works
- LUN deletion works
- Host discovery works
- Multipath remains operational
- Existing data remains intact
- No unexpected alerts appear

### Interview Tip

Firmware upgrades should never validate only "new" functionality.

Always verify that existing provisioning workflows continue to function correctly.

---

## Q7. A newly created LUN is not visible on the Linux server. How would you troubleshoot it?

### What the interviewer wants to evaluate

Troubleshooting methodology.

### Expected Answer

I would troubleshoot layer by layer.

### Step 1

Verify the LUN exists on the storage array.

---

### Step 2

Verify Host Mapping.

- Correct WWPN
- Correct IQN
- Correct Host Group

---

### Step 3

Verify SAN connectivity.

- FC Login
- Zoning
- Switch Health

or

Verify iSCSI session.

---

### Step 4

Rescan storage on Linux.

Examples:

```bash
echo "- - -" > /sys/class/scsi_host/hostX/scan

or

rescan-scsi-bus.sh
```

---

### Step 5

Check discovered devices.

```bash
lsblk

lsscsi

multipath -ll
```

---

### Step 6

Review logs.

```bash
dmesg

journalctl

/var/log/messages
```

### Interview Tip

Never jump directly to hardware.

Always troubleshoot from the layer responsible for the symptom.

---

## Q8. What logs would you collect for a provisioning failure?

### What the interviewer wants to evaluate

Real production experience.

### Expected Answer

Storage Side

- Storage Event Logs
- Controller Logs
- Provisioning Logs
- Audit Logs

Host Side

- dmesg
- journalctl
- multipath logs
- lsscsi output
- lsblk output

SAN Side

- FC Switch Logs
- Zoning Configuration
- Port Status

Application Side

- Database Logs
- Filesystem Logs
- Application Error Logs

### Interview Tip

Collect logs from every layer involved.

Never rely on a single log source.

---

## Q9. How do you verify that provisioning has not impacted existing production workloads?

### What the interviewer wants to evaluate

Production awareness.

### Expected Answer

I verify:

- Existing LUN accessibility
- Existing application availability
- RAID health
- Controller utilization
- Cache utilization
- Storage alerts
- Host connectivity
- Performance metrics
- Multipathing
- Event logs

Then I perform a regression validation of existing workloads.

### Interview Tip

Every provisioning change has the potential to impact existing workloads.

Validation should always include regression checks.

---

## Q10. What are the most common provisioning failures you have seen or expect in enterprise environments?

### What the interviewer wants to evaluate

Practical engineering understanding.

### Expected Answer

Common provisioning failures include:

- Incorrect Host Mapping
- Wrong WWPN or IQN
- Storage Pool out of capacity
- RAID degraded during provisioning
- SAN zoning issues
- Multipath configuration problems
- Host rescan not performed
- Unsupported firmware combinations
- Incorrect permissions
- Human configuration mistakes

### Interview Tip

Interviewers often ask this question to understand whether you think beyond the happy path and can anticipate real operational issues.


---

# Section 2 – Scenario-Based Interview Questions

## Q11. A customer reports that a newly provisioned LUN is not visible on the Linux server. How would you investigate the issue?

### What the interviewer wants to evaluate

- Problem-solving approach
- Layer-by-layer troubleshooting
- Communication skills

### Expected Answer

I would investigate systematically instead of making assumptions.

### Step 1 – Verify Storage Side

- Confirm the LUN exists.
- Verify the LUN status is Online.
- Verify correct capacity.
- Check Storage Pool health.

### Step 2 – Verify Host Mapping

- Correct Host Object
- Correct WWPN/IQN
- Correct LUN Assignment
- No conflicting mappings

### Step 3 – Verify SAN Connectivity

For Fibre Channel

- HBA Online
- FC Switch Port Online
- Correct Zoning
- WWPN Login

For iSCSI

- Session Established
- Target Reachable
- Network Connectivity

### Step 4 – Verify Linux Host

```bash
lsblk
lsscsi
multipath -ll
cat /proc/partitions
```

Perform a SCSI rescan if required.

### Step 5 – Review Logs

```bash
dmesg
journalctl -xe
```

### Interview Tip

Never immediately assume the storage array is the problem.

Investigate each layer in order.

---

## Q12. During provisioning, the Storage Pool suddenly reports "Out of Capacity." What would you do?

### What the interviewer wants to evaluate

- Capacity management
- Decision making

### Expected Answer

First, I would stop further provisioning to avoid inconsistent allocations.

Then I would verify:

- Total Pool Capacity
- Used Capacity
- Thin Provisioning Usage
- Snapshot Consumption
- Reserved Space
- Pool Growth Trend

Possible actions:

- Expand the Storage Pool
- Free unused capacity
- Delete obsolete snapshots
- Migrate workloads
- Provision from another pool if appropriate

### Interview Tip

Never continue provisioning when capacity warnings appear.

Always determine the root cause first.

---

## Q13. After provisioning storage, the application reports slow performance. How would you investigate?

### What the interviewer wants to evaluate

- Performance analysis
- Enterprise thinking

### Expected Answer

I would investigate multiple layers.

Storage Layer

- RAID Type
- Disk Utilization
- Cache Usage
- Controller Utilization
- Queue Depth

Host Layer

- Multipathing
- HBA Errors
- Filesystem Configuration
- CPU Usage

Application Layer

- Database Wait Events
- Application Logs
- Workload Characteristics

Network Layer

- SAN Latency
- FC Errors
- iSCSI Network Performance

### Interview Tip

Performance issues rarely have a single cause.

Investigate every layer before reaching a conclusion.

---

## Q14. A firmware upgrade was completed successfully. What provisioning validations would you perform?

### What the interviewer wants to evaluate

- Regression testing knowledge

### Expected Answer

I would validate both existing and new functionality.

Existing Configuration

- Existing LUNs Accessible
- Existing Volumes Healthy
- Existing Host Mappings Intact

New Provisioning

- Create Storage Pool
- Create Volume
- Create LUN
- Map Host
- Delete LUN
- Expand Volume

Validation

- Read/Write Test
- Multipath Verification
- Event Log Review
- Performance Comparison

### Interview Tip

Firmware validation is incomplete if you only test new provisioning.

Regression testing is equally important.

---

## Q15. You accidentally mapped a production LUN to the wrong host. What would you do?

### What the interviewer wants to evaluate

- Production awareness
- Incident handling

### Expected Answer

Immediate actions:

- Do not perform any write operations.
- Inform the team immediately.
- Verify whether the host has accessed the LUN.
- Remove incorrect mapping.
- Validate correct mapping.
- Confirm production data integrity.
- Document the incident.
- Perform root cause analysis.

### Interview Tip

Interviewers are not expecting perfection.

They want to see that you respond calmly, prioritize data protection, and follow a structured incident response process.

---

# Section 3 – Troubleshooting Interview Questions

## Q16. A RAID group becomes degraded during storage provisioning. How would you handle the situation?

### What the interviewer wants to evaluate

- Storage fundamentals
- Risk assessment
- Decision making

### Expected Answer

My first priority is to understand whether the degraded RAID impacts the provisioning request.

Investigation steps:

1. Verify which drive failed.
2. Confirm RAID level.
3. Check whether rebuild is already running.
4. Verify controller health.
5. Check Storage Pool health.
6. Identify whether the provisioning request depends on this RAID.
7. Review storage event logs.

Possible actions:

- Pause provisioning if data integrity is at risk.
- Replace the failed drive if required.
- Monitor RAID rebuild progress.
- Resume provisioning only after confirming the storage remains healthy.

### Interview Tip

Never ignore a degraded RAID during provisioning.

A degraded RAID directly affects redundancy and increases operational risk.

---

## Q17. Volume creation fails even though the Storage Pool shows free capacity. How would you troubleshoot?

### What the interviewer wants to evaluate

- Logical troubleshooting
- Storage architecture understanding

### Expected Answer

Possible causes include:

- Pool metadata limits reached
- Thin provisioning limits exceeded
- Reserved capacity exhausted
- Maximum volume count reached
- Controller software restrictions
- Permission issues
- Temporary controller synchronization problems

Investigation:

- Check pool utilization.
- Verify metadata usage.
- Review storage event logs.
- Check firmware alerts.
- Verify system limits.
- Retry after confirming controller health.

### Interview Tip

Free capacity alone does not guarantee successful provisioning.

Always investigate logical limits as well.

---

## Q18. After a server reboot, the operating system can no longer see the LUN. What would you investigate?

### What the interviewer wants to evaluate

- Linux knowledge
- SAN understanding
- Layered troubleshooting

### Expected Answer

Storage Side

- Verify LUN still exists.
- Verify Host Mapping.
- Confirm controller health.

SAN Side

- Verify FC ports.
- Check SAN zoning.
- Verify switch status.

Linux Side

```bash
multipath -ll
lsblk
lsscsi
dmesg
journalctl
```

Verify:

- Multipath service
- HBA driver
- Device discovery
- Persistent device configuration

### Interview Tip

A reboot may expose configuration issues that were hidden while the system was running.

---

## Q19. A provisioning operation completed successfully, but write operations fail. What would you investigate?

### What the interviewer wants to evaluate

- End-to-end validation
- Storage access knowledge

### Expected Answer

Storage Layer

- LUN permissions
- Read-only configuration
- Snapshot status

Host Layer

- Filesystem
- Mount options
- Device permissions

Operating System

```bash
mount
lsblk
dmesg
journalctl
```

Application Layer

- Application permissions
- User ownership
- Database logs

### Interview Tip

Provisioning success only confirms resource creation.

Read/write validation confirms usability.

---

## Q20. Multiple hosts suddenly lose access to the same Storage Pool. Where would you begin the investigation?

### What the interviewer wants to evaluate

- Critical incident response
- Failure isolation

### Expected Answer

Since multiple hosts are affected simultaneously, I would suspect a shared infrastructure issue rather than an individual host problem.

Investigation order:

1. Storage Controller Health
2. Storage Pool Status
3. RAID Health	
4. SAN Fabric
5. FC Switch Status
6. Network (iSCSI)
7. Controller Failover Events
8. Storage Event Logs

If only one host is affected:

Investigate the host first.

If many hosts are affected:

Investigate the shared infrastructure first.

### Interview Tip

Always identify the failure scope before troubleshooting.

One affected host usually indicates a host issue.

Many affected hosts usually indicate a shared infrastructure issue.

---

# Section 4 – Manager Round / Design Discussion

## Q21. A customer asks you to provision storage for a new application. What information would you collect before starting?

### What the interviewer wants to evaluate

- Requirement gathering
- Planning ability
- Communication with customers

### Expected Answer

Before provisioning storage, I would gather the following information:

**Business Requirements**

- Purpose of the application
- Production or Non-Production
- Criticality of the application
- High Availability requirements

**Capacity Requirements**

- Initial capacity
- Expected annual growth
- Thin or Thick provisioning preference

**Performance Requirements**

- Expected IOPS
- Throughput requirements
- Latency expectations
- Read/Write workload ratio

**Infrastructure Details**

- Operating System
- Number of Hosts
- Fibre Channel or iSCSI
- Multipathing requirements

**Protection Requirements**

- RAID policy
- Backup policy
- Snapshot requirements
- Disaster Recovery requirements

### Interview Tip

Never start provisioning before understanding business and technical requirements.

---

## Q22. If you have multiple RAID options available, how do you decide which one to use?

### What the interviewer wants to evaluate

- Design thinking
- Engineering decision-making

### Expected Answer

RAID selection depends on workload characteristics rather than available capacity.

I evaluate:

- Performance requirements
- Fault tolerance
- Capacity efficiency
- Rebuild time
- Cost considerations

Examples:

- RAID 10 → Databases and high-performance applications
- RAID 5 → General-purpose workloads
- RAID 6 → Large-capacity environments requiring additional fault tolerance

### Interview Tip

There is no universally "best" RAID level.

The correct RAID level depends on business requirements.

---

## Q23. How would you explain Storage Provisioning to a customer with no storage background?

### What the interviewer wants to evaluate

- Communication skills
- Ability to simplify technical concepts

### Expected Answer

I would compare storage provisioning to constructing an apartment building.

- Physical Disks are like construction materials.
- RAID is the reinforced foundation that provides protection.
- Storage Pool is the entire building.
- Volume is an individual apartment.
- LUN is the apartment key assigned to a specific resident.
- Host Mapping determines which resident can enter which apartment.

This analogy helps customers understand logical storage without introducing unnecessary technical complexity.

### Interview Tip

Good engineers can explain complex systems in simple language.

---

## Q24. How do you ensure that a provisioning change does not impact existing production workloads?

### What the interviewer wants to evaluate

- Change management
- Production awareness

### Expected Answer

Before making any provisioning change, I verify:

- Current storage health
- Available capacity
- RAID status
- Controller utilization
- Existing workload performance
- Active alerts
- Backup availability
- Change window approval

After the change, I validate:

- Existing LUN accessibility
- Existing application functionality
- Storage alerts
- Performance metrics
- Host connectivity

### Interview Tip

Every provisioning activity should include regression validation of existing production workloads.

---

## Q25. What is the difference between configuring storage and validating storage?

### What the interviewer wants to evaluate

- Understanding of your role as a Validation Engineer

### Expected Answer

Configuration focuses on creating or modifying storage resources.

Examples:

- Creating RAID groups
- Creating Storage Pools
- Creating Volumes
- Creating LUNs
- Mapping Hosts

Validation focuses on confirming that those configurations work correctly.

Examples:

- Functional verification
- Read/Write testing
- Performance validation
- Multipath verification
- Regression testing
- Failure testing

### Interview Tip

A Storage Validation Engineer is responsible not only for configuring storage but also for proving that it behaves correctly under different conditions.

---

## Q26. Suppose a customer requests a 50 TB LUN for a new application. What factors would influence your provisioning decision?

### What the interviewer wants to evaluate

- Capacity planning
- Architecture decisions
- Risk assessment

### Expected Answer

I would first understand the workload instead of immediately creating the LUN.

Information required:

- Application type
- Database or File Server
- Expected IOPS
- Read/Write ratio
- Growth rate
- Backup requirements
- Snapshot requirements
- Disaster Recovery requirements
- High Availability requirements

Then I would evaluate:

- Storage Pool capacity
- RAID suitability
- Performance impact
- Future expansion
- Existing production workload impact

Only after completing these validations would I proceed with provisioning.

### Interview Tip

Never provision storage based only on the requested capacity.

Business requirements should always drive provisioning decisions.

---

## Q27. How would you verify that a Storage Pool is suitable for a new workload?

### What the interviewer wants to evaluate

- Capacity planning
- Performance awareness

### Expected Answer

I would verify:

Capacity

- Free Capacity
- Thin Pool utilization
- Growth trends

Performance

- Current IOPS
- Throughput
- Latency
- Controller utilization
- Cache utilization

Reliability

- RAID health
- Drive health
- Storage alerts
- Firmware status

Future Planning

- Expected workload growth
- Existing workload impact

### Interview Tip

A Storage Pool should have enough performance headroom as well as enough capacity.

---

## Q28. A customer reports that storage performance became slow immediately after provisioning a new workload. How would you investigate?

### What the interviewer wants to evaluate

- Performance troubleshooting
- Logical thinking

### Expected Answer

I would compare system behavior before and after provisioning.

Investigation:

Storage

- Storage Pool utilization
- RAID utilization
- Controller CPU
- Cache usage
- Queue depth

Host

- Multipath
- Filesystem
- HBA statistics

Application

- Query workload
- Read/Write pattern
- Concurrent users

Network

- SAN latency
- FC errors
- iSCSI performance

Finally, determine whether the slowdown is caused by:

- Capacity exhaustion
- Controller bottleneck
- Application workload
- Incorrect RAID selection
- Misconfiguration

### Interview Tip

Do not assume the newly provisioned storage is the root cause.

Always collect evidence before concluding.

---

## Q29. How do you decide whether a provisioning issue is caused by storage, SAN, or the operating system?

### What the interviewer wants to evaluate

- Failure isolation
- Layer-by-layer troubleshooting

### Expected Answer

I isolate the problem by validating each layer independently.

Storage Layer

- LUN exists
- Host Mapping
- Storage health

SAN Layer

- Zoning
- FC ports
- Switch health

Operating System

- Device discovery
- Multipathing
- Kernel logs

Application

- Filesystem
- Database
- Application logs

The first layer where expected behavior fails usually identifies the area requiring further investigation.

### Interview Tip

Never troubleshoot all layers simultaneously.

Eliminate one layer at a time.

---

## Q30. What qualities make an excellent Storage Validation Engineer?

### What the interviewer wants to evaluate

- Professional maturity
- Self-awareness
- Engineering mindset

### Expected Answer

A good Storage Validation Engineer should possess:

Technical Skills

- Strong storage fundamentals
- Linux knowledge
- SAN understanding
- Automation skills
- Troubleshooting ability

Engineering Skills

- Logical thinking
- Test design
- Root Cause Analysis
- Risk assessment
- Documentation

Professional Skills

- Communication
- Ownership
- Attention to detail
- Continuous learning
- Team collaboration

Most importantly, a Validation Engineer should never assume that a successful configuration means a successful product.

Every feature must be validated under both normal and failure conditions before it can be trusted in production.

### Interview Tip

Companies hire engineers who can think systematically, communicate clearly, and make reliable technical decisions—not just engineers who can execute commands.

---

# Section 5 – Rapid Fire Questions

## Storage Provisioning

**Q31. What is Storage Provisioning?**

Converting physical storage resources into logical storage that hosts and applications can use.

---

**Q32. What is a Storage Pool?**

A logical collection of storage capacity from which Volumes are allocated.

---

**Q33. What is a Volume?**

A logical allocation of capacity from a Storage Pool.

---

**Q34. What is a LUN?**

A Logical Unit Number presented to a host for storage access.

---

**Q35. Can a Volume exist without a Storage Pool?**

No.

---

**Q36. Can a LUN exist without a Volume?**

No.

---

**Q37. Which comes first: RAID or Storage Pool?**

RAID.

---

**Q38. Which comes first: Storage Pool or Volume?**

Storage Pool.

---

**Q39. Which comes first: Volume or LUN?**

Volume.

---

**Q40. What is Host Mapping?**

The process of assigning a LUN to a specific host.

---

## Storage Validation

**Q41. What is the first validation after provisioning?**

Verify the storage objects were created successfully.

---

**Q42. What is the second validation?**

Verify the host discovers the LUN.

---

**Q43. What is the final validation?**

Verify the application can perform read/write operations.

---

**Q44. What is regression testing?**

Verifying that existing functionality continues to work after a change.

---

**Q45. What is a negative test case?**

Testing invalid or unexpected inputs to verify proper error handling.

---

## Linux

**Q46. Which command lists block devices?**

```bash
lsblk
```

---

**Q47. Which command lists SCSI devices?**

```bash
lsscsi
```

---

**Q48. Which command displays multipath devices?**

```bash
multipath -ll
```

---

**Q49. Which command shows kernel messages?**

```bash
dmesg
```

---

**Q50. Which command displays system logs?**

```bash
journalctl
```

---

## Troubleshooting

**Q51. A LUN is not visible. Where do you start?**

Host Mapping.

---

**Q52. Multiple hosts lose storage simultaneously. Where do you start?**

Shared infrastructure (Storage Controller, SAN Fabric, Storage Pool).

---

**Q53. One host loses storage. Where do you start?**

The affected host.

---

**Q54. Which layer provides redundancy?**

RAID.

---

**Q55. Which layer manages capacity?**

Storage Pool.

---

**Q56. Which layer allocates storage?**

Volume.

---

**Q57. Which layer presents storage?**

LUN.

---

**Q58. Which layer controls access?**

Host Mapping.

---

## Best Practices

**Q59. Should provisioning end after creating a LUN?**

No. Validation must be completed.

---

**Q60. What is the most important skill for a Storage Validation Engineer?**

The ability to systematically validate, troubleshoot, and verify storage behavior under both normal and failure conditions.



