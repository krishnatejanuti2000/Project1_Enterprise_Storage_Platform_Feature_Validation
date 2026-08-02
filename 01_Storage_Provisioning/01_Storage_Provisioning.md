# Module 01 - Storage Provisioning

> **Project:** Enterprise Storage Platform Feature Validation 
> **Module:** 01 - Storage Provisioning 

---

# Chapter 1 - Enterprise Scenario

## Introduction

Enterprise Storage Provisioning is not the process of creating a Storage Pool, Volume, or LUN.

It is the complete engineering workflow that transforms newly installed storage hardware into production-ready storage that can be consumed by enterprise applications.

Understanding Storage Provisioning begins with understanding a real production environment.

---

## Enterprise Scenario

ABC Bank has purchased a new enterprise storage array to host its mission-critical Oracle production database.

The infrastructure team completes the following tasks:

- Rack installation
- Power connection
- Network connectivity
- Controller boot
- Firmware initialization
- Hardware discovery
- Health verification

The storage array reports:

```text
System Status : Healthy

Controllers : Online

Physical Drives : Healthy

No Hardware Faults Detected
```

At this stage, the storage platform is operational.

However, the Oracle DBA submits the following request:

> "Provision a 40 TB storage device for the Oracle Production Database."

Can the Storage Administrator immediately provide storage?

**No.**

Although the storage array is healthy, no logical storage has been provisioned.

There is:

- No RAID
- No Storage Pool
- No Volume
- No LUN
- No Host Mapping

From the operating system's perspective, no usable storage exists.

This is where Storage Provisioning begins.

---

## Engineering Perspective

A healthy storage array is **not automatically a usable storage array**.

Enterprise storage must pass through two independent readiness stages.

```text
Platform Readiness
        │
        ▼
Customer Readiness
        │
        ▼
Application Readiness
```

---

### Platform Readiness

Platform Readiness is completed automatically by the storage system.

Typical activities include:

- Hardware Initialization
- Controller Initialization
- Firmware Loading
- Drive Discovery
- Hardware Health Checks
- Cache Initialization
- Internal Metadata Initialization

At the end of Platform Readiness, the storage array is operational but cannot yet provide storage to any application.

---

### Customer Readiness

Customer Readiness is performed by the Storage Administrator.

During this stage, logical storage resources are created according to business requirements.

Typical activities include:

- RAID Creation
- Storage Pool Creation
- Volume Creation
- Provisioning Type Selection (Thin or Thick)
- LUN Creation
- Host Mapping

Only after these steps does the operating system discover usable storage.

---

### Application Readiness

Once storage has been provisioned and presented to the host:

- Linux discovers the LUN
- The administrator partitions the disk
- A filesystem is created
- The filesystem is mounted
- The application begins storing data

At this point, the storage provisioning lifecycle is complete.

---

## Key Engineering Principle

A healthy storage array does **not** imply that applications can store data.

The complete lifecycle is:

```text
Healthy Storage Array
        │
        ▼
Storage Provisioning
        │
        ▼
Host Presentation
        │
        ▼
Operating System Discovery
        │
        ▼
Filesystem Creation
        │
        ▼
Application Storage
```

Storage Provisioning is therefore the engineering process that bridges the gap between infrastructure readiness and application usability.
============================================================================================================
# Chapter 2 - Why Storage Provisioning Exists

## The Engineering Problem

When a new enterprise storage array is powered on for the first time, it contains physical hardware but no logical storage organization.

Although every drive is healthy and every controller is operational, applications cannot immediately store data.

Consider the storage array immediately after installation.

```text
Storage Array

├── Controller A (Healthy)
├── Controller B (Healthy)
├── 24 Physical SSDs (Healthy)
├── Cache Memory Initialized
├── Firmware Running
└── No Hardware Faults
```

From a hardware perspective, the storage platform is fully operational.

However, from the customer's perspective, the storage array is still unusable.

Why?

Because enterprise applications do not communicate directly with physical drives.

Applications require logical storage resources that provide:

- Data Protection
- Capacity Management
- Workload Isolation
- Controlled Host Access
- Simplified Administration

These logical resources do not exist automatically.

They must be created by the Storage Administrator.

---

## Why Raw Drives Cannot Be Used Directly

Suppose a storage array contains twenty-four enterprise SSDs.

Can an Oracle Database directly start writing data to SSD Number 7?

The answer is **No**.

Raw drives present several problems:

- No redundancy
- No fault tolerance
- No centralized capacity management
- No workload isolation
- No secure host access
- No logical abstraction

Managing hundreds of physical drives individually would become impossible in an enterprise environment.

Storage Provisioning solves this problem by introducing multiple logical abstraction layers.

---

## The Purpose of Storage Provisioning

Storage Provisioning transforms raw hardware into enterprise-ready storage through a series of logical layers.

Each layer introduces a specific capability.

```text
Physical Drives
        │
        ▼
Provides Physical Storage
        │
        ▼
RAID
        │
        ▼
Provides Data Protection
        │
        ▼
Storage Pool
        │
        ▼
Provides Capacity Management
        │
        ▼
Volume
        │
        ▼
Provides Workload Isolation
        │
        ▼
Thin / Thick Provisioning
        │
        ▼
Defines Capacity Allocation Strategy
        │
        ▼
LUN
        │
        ▼
Presents Storage to Hosts
        │
        ▼
Host Mapping
        │
        ▼
Controls Host Access
        │
        ▼
Operating System
        │
        ▼
Filesystem
        │
        ▼
Application
```

Every layer exists to solve a specific engineering problem.

No layer is optional.

Each layer builds upon the previous one to create a complete enterprise storage solution.

---

## Storage Provisioning as a Layered Architecture

Rather than exposing physical disks directly to applications, enterprise storage systems progressively abstract the hardware into logical objects.

Each abstraction introduces new functionality while hiding unnecessary hardware complexity.

| Layer | Primary Responsibility |
|---------|------------------------|
| Physical Drives | Store physical data |
| RAID | Data protection and fault tolerance |
| Storage Pool | Capacity management |
| Volume | Workload isolation |
| Thin / Thick | Capacity allocation strategy |
| LUN | Storage presentation |
| Host Mapping | Host access control |
| Operating System | Filesystem management |
| Application | Business data |

This layered architecture allows enterprise storage systems to scale from a few terabytes to multiple petabytes while remaining manageable.

---

## Engineering Perspective

Storage Provisioning is not simply the creation of a Volume or a LUN.

It is the engineering process of transforming unmanaged physical storage into secure, reliable, scalable, and application-ready enterprise storage.

Without Storage Provisioning:

- Physical drives remain isolated hardware devices.
- Applications cannot discover storage.
- Capacity cannot be managed.
- Storage cannot be securely shared among hosts.

Storage Provisioning bridges the gap between hardware infrastructure and business applications.
============================================================================================================

# Chapter 3 - Complete Enterprise Storage Provisioning Lifecycle

## From Business Requirement to Application Storage

Storage Provisioning does not begin with creating a Storage Pool or a Volume.

It begins when a business requirement is received.

Every provisioning activity performed by a Storage Administrator originates from a storage request submitted by a customer, application owner, database administrator, virtualization team, or infrastructure team.

The Storage Administrator transforms this business requirement into a sequence of logical storage objects that eventually become usable by the operating system.

This complete transformation is known as the **Storage Provisioning Lifecycle**.

---

## Enterprise Scenario

ABC Bank plans to deploy a new Oracle Production Database.

The Oracle DBA submits the following request to the Storage Team:

```text
Storage Request

Application  : Oracle Production Database

Required Capacity : 40 TB

Provisioning Type : Thick

Required Availability : High

Operating System : Red Hat Enterprise Linux
```

The Storage Administrator now begins the provisioning process.

---

# Complete Enterprise Provisioning Workflow

```text
Business Requirement
        │
        ▼
Storage Request Received
        │
        ▼
Review Available Capacity
        │
        ▼
Create RAID
        │
        ▼
Create Storage Pool
        │
        ▼
Create Volume
        │
        ▼
Select Provisioning Type
(Thin / Thick)
        │
        ▼
Create LUN
        │
        ▼
Assign LUN ID
        │
        ▼
Map LUN to Host
        │
        ▼
Linux Host Rescan
        │
        ▼
Operating System Discovers New Disk
        │
        ▼
Partition Creation
        │
        ▼
Filesystem Creation
        │
        ▼
Mount Filesystem
        │
        ▼
Application Starts Using Storage
```

This workflow represents the complete lifecycle followed in enterprise storage environments.

Every logical object created during this process performs a specific responsibility.

No step exists without purpose.

---

# Step 1 - Business Requirement

Every provisioning activity begins with a customer requirement.

Examples include:

- Oracle Database
- SAP HANA
- VMware Datastore
- SQL Server
- Backup Repository
- AI/ML Storage
- File Server

The requested workload determines:

- Required Capacity
- Performance Requirements
- Availability Requirements
- Provisioning Type
- RAID Level
- Expansion Strategy

Storage is never provisioned without understanding the workload.

---

# Step 2 - RAID Creation

The administrator creates an appropriate RAID configuration.

Purpose:

- Protect data
- Provide fault tolerance
- Improve availability

Output:

A protected logical RAID Group.

---

# Step 3 - Storage Pool Creation

The RAID Group is added to a Storage Pool.

Purpose:

- Centralized capacity management
- Flexible storage allocation
- Future expansion
- Simplified administration

Output:

A logical storage pool from which future volumes will be created.

---

# Step 4 - Volume Creation

The administrator creates a Volume inside the Storage Pool.

Purpose:

- Allocate storage for a specific workload
- Isolate applications
- Enable independent management

Example:

```text
Storage Pool

↓

Oracle_VOL

40 TB
```

At this stage the storage still cannot be seen by Linux.

The Volume exists only inside the storage array.

---

# Step 5 - Provisioning Type Selection

During Volume creation, the Storage Administrator selects the provisioning method.

Available options:

- Thin Provisioning
- Thick Provisioning

This decision determines how physical capacity will be managed throughout the lifetime of the Volume.

The Volume is now logically complete.

However, it is still invisible to the operating system.

---

# Step 6 - LUN Creation

A LUN is created from the Volume.

Purpose:

- Present storage to the host operating system
- Convert the internal Volume into a host-visible block device

Without a LUN, Linux cannot discover the Volume.

---

# Step 7 - LUN ID Assignment

Every LUN receives a unique identifier.

Example:

```text
Oracle_VOL

↓

LUN 15
```

The LUN ID allows hosts and storage controllers to uniquely identify the presented storage object.

---

# Step 8 - Host Mapping

The newly created LUN is mapped to the required host.

Example:

```text
Oracle_Linux_Server

↓

Mapped

↓

LUN 15
```

Only mapped hosts can access the storage.

Other servers remain unaware of its existence.

---

# Step 9 - Operating System Discovery

After Host Mapping, the Linux administrator rescans the storage adapters.

Linux discovers a new block device.

Example:

```text
/dev/sdb
```

At this point the storage is visible but not yet usable.

---

# Step 10 - Filesystem Preparation

The Linux administrator performs:

- Partition Creation (optional depending on design)
- Filesystem Creation
- Mount Point Creation
- Filesystem Mount

The operating system now provides a usable filesystem.

---

# Step 11 - Application Begins Using Storage

Finally, the application writes data to the mounted filesystem.

Example:

```text
Oracle Database

↓

Filesystem

↓

Linux Block Layer

↓

LUN

↓

Volume

↓

Storage Pool

↓

RAID

↓

Physical Drives
```

The provisioning lifecycle is now complete.

Business storage requirements have successfully been transformed into application-ready enterprise storage.

---

## Engineering Perspective

Storage Provisioning is not a single operation.

It is a carefully orchestrated sequence of logical storage transformations.

Each stage introduces a new abstraction layer while preserving reliability, manageability, scalability, and security.

Understanding this lifecycle is fundamental to Storage Administration, Storage Validation, Storage Automation, and Storage Troubleshooting.

============================================================================================================

# Chapter 4 - Storage Pool

## Introduction

A Storage Pool is the first logical storage layer responsible for centralized capacity management within an enterprise storage system.

Although RAID provides data protection and fault tolerance, it is not designed to allocate storage directly to applications.

Storage Pools solve this limitation by aggregating protected storage into a flexible and manageable resource from which logical storage can be provisioned.

Without Storage Pools, enterprise storage administration would become increasingly complex as storage capacity grows.

---

# Why Does a Storage Pool Exist?

Imagine an enterprise storage array containing multiple RAID Groups.

```text
RAID Group 1 : 20 TB

RAID Group 2 : 30 TB

RAID Group 3 : 50 TB
```

Suppose an application requires **40 TB** of storage.

Without a Storage Pool, the Storage Administrator must decide:

- Which RAID Group should be used?
- What happens if one RAID Group becomes full?
- How should future capacity expansion be handled?
- How can free space across multiple RAID Groups be managed?

Managing individual RAID Groups quickly becomes difficult in large enterprise environments.

Storage Pools solve this problem by combining one or more RAID Groups into a single logical storage resource.

---

# Definition

A Storage Pool is a logical abstraction that aggregates one or more RAID Groups into a single manageable storage resource from which Volumes are created.

Applications never consume storage directly from RAID Groups.

Instead, storage is allocated from the Storage Pool.

---

# Purpose of a Storage Pool

A Storage Pool provides:

- Centralized capacity management
- Logical abstraction of physical storage
- Simplified storage administration
- Flexible capacity allocation
- Future storage expansion
- Efficient utilization of protected storage

It separates physical storage organization from logical storage allocation.

---

# Internal Architecture

```text
Physical Drives
        │
        ▼
      RAID 5
        │
        ├─────────────┐
        ▼             ▼
      RAID 6       RAID 10
        │             │
        └──────┬──────┘
               ▼
        Storage Pool
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
   Volume A Volume B Volume C
```

The Storage Pool does not replace RAID.

Instead, it sits above RAID and provides a logical layer for managing protected capacity.

---

# Controller Perspective

When a Storage Pool is created, the storage controller performs several internal operations.

Typical workflow:

```text
Administrator Requests Pool Creation
                │
                ▼
Controller Validates RAID Groups
                │
                ▼
Checks Available Capacity
                │
                ▼
Creates Storage Pool Object
                │
                ▼
Allocates Pool Metadata
                │
                ▼
Updates Configuration Database
                │
                ▼
Storage Pool Ready
```

The controller now maintains information about the Storage Pool and uses this metadata whenever Volumes are created, expanded, or deleted.

---

# Metadata Maintained by the Controller

Although implementation differs between storage vendors, a Storage Pool generally maintains metadata such as:

- Pool Identifier
- Pool Name
- Member RAID Groups
- Total Capacity
- Available Capacity
- Allocated Capacity
- Pool Health Status
- Expansion Information
- Performance Statistics

This metadata enables the storage array to manage capacity efficiently and consistently.

---

# Real-Time Creation Workflow

The following sequence represents a typical enterprise provisioning workflow.

```text
Business Requirement

↓

Review Existing Capacity

↓

Open Storage Manager

↓

Storage

↓

Storage Pools

↓

Create Pool

↓

Select RAID Group(s)

↓

Assign Pool Name

↓

Review Capacity

↓

Confirm

↓

Controller Creates Pool Metadata

↓

Storage Pool Available
```

Although menu names differ between vendors such as Dell, NetApp, HPE, IBM, or Pure Storage, the overall workflow remains conceptually similar.

---

# Enterprise Example

ABC Bank has three RAID Groups available.

```text
RAID 5 : 20 TB

RAID 6 : 30 TB

RAID 10 : 50 TB
```

The Storage Administrator creates:

```text
Production_Pool

Total Capacity : 100 TB
```

The Production Pool is now used to provision storage for multiple workloads.

```text
Production_Pool

├── Oracle_VOL
├── VMware_VOL
├── SQL_VOL
└── Backup_VOL
```

Each application receives its own Volume while sharing the same centralized Storage Pool.

---

# Validation Perspective

During Storage Pool validation, a Storage Validation Engineer verifies that the feature behaves correctly.

Typical validation activities include:

- Create a Storage Pool
- Verify Pool creation succeeds
- Verify total capacity
- Verify available capacity
- Verify RAID Group membership
- Verify metadata consistency
- Expand the Storage Pool
- Verify updated capacity
- Delete the Storage Pool
- Verify resources are released correctly

Testing focuses on functionality, reliability, and consistency across normal, boundary, and failure scenarios.

---

# Key Takeaways

- A Storage Pool is created from one or more RAID Groups.
- RAID provides protected storage.
- Storage Pools provide centralized capacity management.
- Volumes are always created from Storage Pools.
- Applications never consume storage directly from RAID Groups.
- Storage Pools simplify enterprise storage administration and future expansion.

============================================================================================================
# Chapter 5 - Volume

## Introduction

A Volume is a logical storage object created from a Storage Pool and allocated for a specific application or workload.

While a Storage Pool provides centralized capacity management, it is not directly consumed by applications.

Instead, storage administrators create Volumes from the Storage Pool, allowing individual applications to receive dedicated logical storage with independent capacity, provisioning policies, and lifecycle management.

Every enterprise application ultimately stores its data inside one or more Volumes.

---

# Why Does a Volume Exist?

Suppose an enterprise Storage Pool contains 100 TB of available capacity.

```text
Production_Pool

Total Capacity : 100 TB
```

Several business teams submit storage requests:

- Oracle Database : 40 TB
- VMware : 20 TB
- SQL Server : 15 TB
- Backup Repository : 25 TB

If all applications accessed the Storage Pool directly:

- Capacity ownership would be unclear.
- Applications could interfere with each other.
- Expansion would be difficult.
- Monitoring would become complex.
- Performance isolation would not be possible.

To solve these problems, logical storage allocations called **Volumes** are created.

---

# Definition

A Volume is a logical allocation of storage capacity created from a Storage Pool for a specific workload or application.

Each Volume has its own:

- Name
- Capacity
- Provisioning Type
- Lifecycle
- Metadata
- Management Operations

The Volume exists entirely inside the storage array and is not yet visible to the host operating system.

---

# Purpose of a Volume

A Volume provides:

- Dedicated storage allocation
- Workload isolation
- Independent capacity management
- Flexible expansion
- Simplified administration
- Controlled provisioning policies

Each application receives its own logical storage instead of sharing raw pool capacity.

---

# Internal Architecture

```text
Physical Drives
        │
        ▼
      RAID
        │
        ▼
 Storage Pool
        │
        ├──────────────┬──────────────┐
        ▼              ▼              ▼
 Oracle_VOL      VMware_VOL     Backup_VOL
```

Every Volume is created from the Storage Pool but remains logically independent.

Deleting one Volume does not affect the others.

---

# Real-Time Volume Creation Workflow

In a production environment, creating a Volume typically follows this workflow:

```text
Business Requirement

↓

Review Available Capacity

↓

Open Storage Manager

↓

Storage

↓

Volumes

↓

Create Volume

↓

Enter Volume Name

↓

Select Storage Pool

↓

Specify Capacity

↓

Choose Provisioning Type

↓

Review Configuration

↓

Confirm

↓

Controller Creates Volume Object

↓

Controller Updates Metadata

↓

Volume Ready
```

Although vendor interfaces differ, the workflow remains conceptually similar across enterprise storage platforms.

---

# Controller Perspective

When the administrator clicks **Create Volume**, the storage controller performs several internal operations.

```text
Volume Creation Request

↓

Validate Storage Pool

↓

Validate Requested Capacity

↓

Validate Provisioning Policy

↓

Create Volume Object

↓

Allocate Volume Metadata

↓

Update Configuration Database

↓

Update Pool Capacity Information

↓

Return Success
```

The Volume now exists inside the storage array.

However, it is still invisible to the operating system because no LUN has been created.

---

# Metadata Maintained for a Volume

Typical metadata includes:

- Volume Identifier
- Volume Name
- Parent Storage Pool
- Capacity
- Used Capacity
- Free Capacity
- Provisioning Type
- Health Status
- Owner Controller
- Creation Timestamp
- Expansion History

This metadata enables accurate capacity tracking, monitoring, and management throughout the Volume lifecycle.

---

# Provisioning Types

Every Volume must be created using a provisioning policy.

Enterprise storage arrays commonly support:

- Thin Provisioning
- Thick Provisioning

The selected provisioning type determines how physical capacity is allocated and managed.

---

## Thin Provisioning

Thin Provisioning allocates logical capacity immediately while consuming physical storage only as application data is written.

### Internal Workflow

```text
Create 40 TB Volume

↓

Reserve Metadata

↓

No Physical Blocks Allocated

↓

Application Writes Data

↓

Controller Allocates Physical Blocks On Demand

↓

Pool Usage Increases Gradually
```

### Enterprise Use Cases

Thin Provisioning is commonly selected for:

- VMware Datastores
- Virtual Desktop Infrastructure (VDI)
- Development Environments
- Test Labs
- Backup Repositories
- General-purpose virtual workloads

These environments usually allocate more logical capacity than is immediately consumed.

---

## Thick Provisioning

Thick Provisioning reserves the requested physical capacity during Volume creation.

### Internal Workflow

```text
Create 40 TB Volume

↓

Reserve Metadata

↓

Reserve 40 TB Physical Capacity

↓

Update Storage Pool Capacity

↓

Volume Ready
```

Capacity is guaranteed regardless of actual application usage.

---

### Enterprise Use Cases

Thick Provisioning is commonly selected for:

- Oracle Production Databases
- SQL Server Production
- Banking Systems
- SAP HANA
- High-performance transactional workloads

These applications require guaranteed capacity and predictable performance.

---

# Choosing Between Thin and Thick Provisioning

Provisioning type should always be selected based on workload requirements rather than personal preference.

| Workload | Recommended Provisioning |
|----------|--------------------------|
| Oracle Production | Thick |
| SQL Server Production | Thick |
| Banking Systems | Thick |
| SAP HANA | Thick |
| VMware Datastore | Thin |
| Development | Thin |
| Test Environment | Thin |
| Backup Repository | Thin |
| VDI | Thin |

There is no universally "better" provisioning type.

The correct choice depends entirely on business and application requirements.

---

# Enterprise Example

ABC Bank requires storage for multiple workloads.

```text
Production_Pool

↓

Oracle_VOL

40 TB

Provisioning : Thick
```

A second request is received.

```text
VMware_VOL

100 TB Logical

Provisioning : Thin
```

Although VMware receives 100 TB of logical capacity, physical storage is allocated only as virtual machines consume space.

---

# Validation Perspective

During Volume validation, a Storage Validation Engineer verifies:

- Volume creation
- Thin Provisioning behaviour
- Thick Provisioning behaviour
- Capacity accounting
- Pool utilization
- Volume expansion
- Volume deletion
- Metadata consistency
- Capacity recovery after deletion
- Boundary and negative scenarios

Testing confirms that the controller behaves correctly throughout the Volume lifecycle.

---

# Key Takeaways

- A Volume is created from a Storage Pool.
- Applications receive storage through Volumes.
- Volumes isolate workloads from one another.
- Every Volume has its own metadata and lifecycle.
- Thin Provisioning allocates physical storage on demand.
- Thick Provisioning reserves physical storage immediately.
- Provisioning type must always be selected according to workload requirements.
===========================================================================================================
# Chapter 6 - LUN (Logical Unit Number)

## Introduction

A Volume is an internal logical storage object that exists only within the storage array.

However, enterprise operating systems such as Linux, Windows, or VMware ESXi cannot directly discover or access internal storage objects.

A mechanism is required to present a Volume to the host operating system as a usable block storage device.

This mechanism is provided by the **Logical Unit Number (LUN)**.

The LUN serves as the presentation layer between the storage array and the host, allowing applications to consume storage without understanding the storage array's internal architecture.

---

# Why Does a LUN Exist?

Suppose the Storage Administrator has created the following Volume:

```text
Production_Pool

↓

Oracle_VOL

40 TB
```

From the storage array's perspective, the Volume exists and is healthy.

However, on the Linux server:

```bash
lsblk
```

Output:

```text
NAME    SIZE

sda     500G
```

The newly created Volume does not appear.

Why?

Because the Volume is an internal storage object.

Linux has no visibility into the internal configuration of the storage array.

The Volume must first be presented to the host.

That presentation is performed by creating a **LUN**.

---

# Definition

A LUN (Logical Unit Number) is a logical presentation object that exposes a Volume to one or more host systems as a block storage device.

The LUN does not store data independently.

Instead, it acts as the presentation interface between the storage array and the host operating system.

---

# Purpose of a LUN

The primary responsibilities of a LUN include:

- Presenting storage to hosts
- Providing a unique identifier for storage presentation
- Enabling operating system discovery
- Supporting controlled host access
- Acting as the interface between logical storage and the host

Without a LUN, a Volume remains inaccessible to external systems.

---

# Internal Architecture

```text
Physical Drives
        │
        ▼
      RAID
        │
        ▼
 Storage Pool
        │
        ▼
     Volume
        │
        ▼
       LUN
        │
        ▼
 Host Mapping
        │
        ▼
 Linux Server
```

The LUN does not replace the Volume.

Instead, it provides a mechanism for presenting the Volume outside the storage array.

---

# Real-Time LUN Creation Workflow

A typical enterprise workflow for creating a LUN is:

```text
Volume Already Exists

↓

Open Storage Manager

↓

Storage

↓

LUNs

↓

Create LUN

↓

Select Volume

↓

Assign LUN ID

↓

Review Configuration

↓

Confirm

↓

Controller Creates LUN Object

↓

Controller Updates Metadata

↓

LUN Ready
```

Although management interfaces vary between storage vendors, the logical workflow remains similar.

---

# Controller Perspective

When the Storage Administrator creates a LUN, the storage controller performs several internal operations.

```text
Receive LUN Creation Request

↓

Validate Volume

↓

Verify Volume Health

↓

Assign Unique LUN ID

↓

Create LUN Object

↓

Update Presentation Metadata

↓

Update Configuration Database

↓

Return Success
```

The Volume is now associated with a LUN.

However, it is still invisible to the operating system until Host Mapping is completed.

---

# Metadata Maintained for a LUN

Typical LUN metadata includes:

- LUN Identifier
- Associated Volume
- LUN Capacity
- Presentation Status
- Access Permissions
- Owning Controller
- Mapping Information
- Creation Timestamp
- Health Status

This metadata enables the storage controller to manage storage presentation efficiently.

---

# Enterprise Example

ABC Bank provisions storage for its Oracle production server.

```text
Production_Pool

↓

Oracle_VOL

↓

Create LUN

↓

LUN ID : 15
```

Internally, the storage array now associates:

```text
Volume

↓

Oracle_VOL

↓

Presented As

↓

LUN 15
```

The storage is prepared for presentation but remains inaccessible until it is mapped to the correct host.

---

# Volume vs LUN

Although closely related, Volumes and LUNs perform different responsibilities.

| Volume | LUN |
|---------|-----|
| Internal storage object | Presentation object |
| Created from Storage Pool | Created from Volume |
| Stores application data | Presents storage to the host |
| Exists only inside the storage array | Can be discovered by hosts after mapping |
| Not directly visible to Linux | Appears as a block device after presentation |

The Volume owns the storage.

The LUN exposes that storage to the host.

---

# Validation Perspective

During LUN validation, a Storage Validation Engineer verifies:

- LUN creation
- Unique LUN ID assignment
- Association with the correct Volume
- Presentation metadata
- LUN deletion
- Recreation with new identifiers
- Boundary conditions
- Duplicate LUN ID handling
- Error handling for invalid configurations

Testing ensures that storage presentation behaves correctly under normal and failure conditions.

---

# Key Takeaways

- A LUN is created from a Volume.
- A LUN presents storage to the host operating system.
- Linux cannot discover a Volume directly.
- The storage controller assigns a unique LUN ID during creation.
- A LUN becomes usable only after it is mapped to a host.
- Volume and LUN are separate logical objects with different responsibilities.

===========================================================================================================
# Chapter 7 - Host Mapping

## Introduction

Creating a Volume and a LUN does not automatically make storage available to a server.

Enterprise storage arrays support multiple hosts simultaneously, and each host should access only the storage explicitly assigned to it.

Host Mapping is the process of granting a specific host permission to access a particular LUN.

Only after Host Mapping is completed can the operating system discover and use the storage.

---

# Why Does Host Mapping Exist?

Consider an enterprise storage array connected to multiple servers.

```text
Storage Array

│

├── Oracle Server

├── VMware Cluster

├── SQL Server

├── Backup Server

└── Development Server
```

Suppose the Storage Administrator creates:

```text
Oracle_VOL

↓

LUN 15
```

Should every connected server automatically see this storage?

The answer is **No.**

If all hosts could access every LUN:

- Applications could overwrite each other's data.
- Sensitive business data would be exposed.
- Filesystems could become corrupted.
- Unauthorized systems could access production storage.

Host Mapping prevents these problems by ensuring that only authorized hosts can access specific storage resources.

---

# Definition

Host Mapping is the process of associating a LUN with one or more authorized hosts, allowing only those hosts to discover and access the presented storage.

Host Mapping acts as the access-control mechanism between the storage array and connected servers.

---

# Purpose of Host Mapping

Host Mapping provides:

- Controlled storage access
- Host isolation
- Secure storage presentation
- Multi-host storage management
- Enterprise access control

Without Host Mapping, enterprise storage environments would be insecure and difficult to manage.

---

# Internal Architecture

```text
Physical Drives
        │
        ▼
      RAID
        │
        ▼
 Storage Pool
        │
        ▼
     Volume
        │
        ▼
       LUN
        │
        ▼
   Host Mapping
        │
        ▼
  Authorized Host
        │
        ▼
 Linux Operating System
```

Host Mapping is the final storage-array operation before the operating system can discover the storage.

---

# Real-Time Host Mapping Workflow

A typical enterprise workflow is:

```text
LUN Already Exists

↓

Open Storage Manager

↓

Hosts

↓

Host Mapping

↓

Select Host

↓

Select LUN

↓

Choose Access Mode

↓

Review Configuration

↓

Confirm

↓

Controller Updates Mapping Table

↓

Host Mapping Complete
```

The exact interface differs between storage vendors, but the logical workflow remains similar.

---

# Controller Perspective

Internally, the storage controller performs several operations.

```text
Receive Host Mapping Request

↓

Validate Host

↓

Validate LUN

↓

Verify Existing Access

↓

Create Mapping Entry

↓

Update Access-Control Metadata

↓

Update Configuration Database

↓

Return Success
```

At this point, the storage array is ready to present the LUN to the host.

---

# Metadata Maintained

Typical Host Mapping metadata includes:

- Host Identifier
- Host Name
- WWPN / IQN / NQN
- Associated LUN
- Access Mode
- Mapping Status
- Controller Ownership
- Creation Timestamp

This metadata allows the storage array to determine exactly which hosts are permitted to access each LUN.

---

# Enterprise Example

ABC Bank provisions storage for its Oracle database.

```text
Oracle_VOL

↓

LUN 15

↓

Mapped To

↓

Oracle_Linux_Server
```

The storage array now records:

```text
Host

Oracle_Linux_Server

↓

Authorized

↓

LUN 15
```

Other connected servers remain unable to discover this LUN.

---

# Operating System Discovery

After Host Mapping is completed, the Linux administrator rescans the storage adapters.

Example:

```bash
echo "- - -" > /sys/class/scsi_host/host0/scan
```

or

```bash
rescan-scsi-bus.sh
```

The operating system discovers a new block device.

Example:

```bash
lsblk
```

Output:

```text
NAME    SIZE

sda     500G

sdb      40T
```

The storage is now visible to Linux.

However, it still cannot store data until it is prepared with a filesystem.

---

# Final Storage Preparation

The Linux administrator typically performs:

```bash
fdisk /dev/sdb
```

or

```bash
parted /dev/sdb
```

Create a filesystem:

```bash
mkfs.xfs /dev/sdb1
```

Create a mount point:

```bash
mkdir /oracle_data
```

Mount the filesystem:

```bash
mount /dev/sdb1 /oracle_data
```

Verify:

```bash
df -h
```

The storage is now ready for application use.

---

# Validation Perspective

During Host Mapping validation, a Storage Validation Engineer verifies:

- Host registration
- Host Mapping creation
- Correct LUN visibility
- Unauthorized host isolation
- Multiple host mappings
- Mapping deletion
- Access permission changes
- Linux device discovery
- Boundary and negative scenarios

Testing confirms that storage is presented only to intended hosts.

---

# Key Takeaways

- Host Mapping is the final storage-array operation before the operating system can discover storage.
- A LUN without Host Mapping remains invisible to connected hosts.
- Host Mapping enforces secure and controlled storage access.
- Only authorized hosts can discover mapped LUNs.
- After Host Mapping, Linux rescans storage adapters and discovers the new block device.
- Filesystem creation and mounting are required before applications can begin using the storage.

===========================================================================================================
# Chapter 8 - Complete End-to-End Enterprise Storage Provisioning Walkthrough

## Introduction

The previous chapters explained each provisioning component individually.

This chapter combines every concept into a complete enterprise provisioning workflow that mirrors real-world storage administration.

It demonstrates how a business requirement is transformed into production-ready storage through coordinated actions performed by the customer, storage administrator, storage controller, and operating system.

Understanding this complete lifecycle is essential for Storage Administrators, Storage Validation Engineers, Storage Automation Engineers, and Storage Support Engineers.

---

# Enterprise Scenario

ABC Bank is deploying a new Oracle Production Database.

The Oracle DBA submits the following storage request.

```text
Application          : Oracle Production Database
Required Capacity    : 40 TB
Provisioning Type    : Thick
Availability         : High
Host                 : oracle-rhel01
Operating System     : Red Hat Enterprise Linux
```

The Storage Team begins the provisioning process.

---

# Phase 1 - Business Requirement

Business Requirement

↓

Oracle DBA submits storage request

↓

Storage Team reviews request

↓

Capacity planning

↓

Performance review

↓

Availability review

↓

Provisioning approved

---

# Phase 2 - RAID Creation

The Storage Administrator creates an appropriate RAID Group.

Example

```text
12 Enterprise SSDs

↓

RAID 10

↓

Protected Capacity Available
```

Controller Operations

```text
Validate Drives

↓

Create RAID Metadata

↓

Initialize RAID

↓

Verify Health

↓

RAID Ready
```

Result

Protected storage is available.

---

# Phase 3 - Storage Pool Creation

The administrator creates a Storage Pool.

Administrator Workflow

```text
Storage

↓

Pools

↓

Create

↓

Select RAID Group

↓

Production_Pool

↓

Confirm
```

Controller Workflow

```text
Validate RAID

↓

Create Pool Object

↓

Allocate Metadata

↓

Update Capacity Database

↓

Pool Ready
```

Result

Centralized capacity management is available.

---

# Phase 4 - Volume Creation

Administrator Workflow

```text
Storage

↓

Volumes

↓

Create

↓

Oracle_VOL

↓

40 TB

↓

Select Production_Pool
```

Controller Workflow

```text
Validate Pool

↓

Create Volume Object

↓

Allocate Metadata

↓

Update Capacity Information

↓

Volume Ready
```

Result

A dedicated logical storage object exists.

The operating system still cannot see it.

---

# Phase 5 - Provisioning Policy

Business Requirement

Oracle Production Database

Decision

```text
Provisioning Type

↓

Thick Provisioning
```

Controller

```text
Reserve Physical Capacity

↓

Reserve Metadata

↓

Update Pool Usage

↓

Complete
```

Result

Guaranteed capacity is reserved for Oracle.

---

# Phase 6 - LUN Creation

Administrator Workflow

```text
Volumes

↓

Oracle_VOL

↓

Create LUN

↓

LUN ID 15

↓

Confirm
```

Controller Workflow

```text
Validate Volume

↓

Assign LUN ID

↓

Create Presentation Object

↓

Update Metadata

↓

Complete
```

Result

The Volume can now be presented to hosts.

---

# Phase 7 - Host Mapping

Administrator Workflow

```text
Hosts

↓

oracle-rhel01

↓

Map LUN 15

↓

Read / Write

↓

Confirm
```

Controller Workflow

```text
Validate Host

↓

Validate LUN

↓

Create Mapping

↓

Update Access Database

↓

Complete
```

Result

The storage array authorizes oracle-rhel01 to access LUN 15.

---

# Phase 8 - Linux Storage Discovery

Linux Administrator

Rescan SCSI Bus

```bash
echo "- - -" > /sys/class/scsi_host/host0/scan
```

Verify

```bash
lsblk
```

Example

```text
NAME     SIZE

sda      500G

sdb       40T
```

The operating system now detects the newly presented storage.

---

# Phase 9 - Storage Preparation

Create a partition.

```bash
parted /dev/sdb
```

Create a filesystem.

```bash
mkfs.xfs /dev/sdb1
```

Create a mount point.

```bash
mkdir /oracle_data
```

Mount the filesystem.

```bash
mount /dev/sdb1 /oracle_data
```

Verify.

```bash
df -h
```

Example

```text
Filesystem      Size   Used   Avail

/dev/sdb1        40T      0     40T
```

Result

Linux storage is ready.

---

# Phase 10 - Application Begins Using Storage

Oracle Database starts writing data.

The complete data path becomes

```text
Oracle Database

↓

Filesystem

↓

Linux Block Layer

↓

Multipath (if configured)

↓

SCSI

↓

Fibre Channel / iSCSI / NVMe-oF

↓

LUN

↓

Volume

↓

Storage Pool

↓

RAID

↓

Enterprise SSDs
```

Every application write follows this logical path until it reaches the physical storage media.

---

# Complete Enterprise Workflow

```text
Business Requirement

↓

Capacity Planning

↓

Create RAID

↓

Create Storage Pool

↓

Create Volume

↓

Select Thin / Thick

↓

Create LUN

↓

Map LUN

↓

Linux Rescan

↓

Discover Storage

↓

Partition

↓

Filesystem

↓

Mount

↓

Application Starts Using Storage
```

---

# Engineering Responsibilities

| Component | Responsibility |
|-----------|----------------|
| RAID | Data protection and fault tolerance |
| Storage Pool | Capacity management |
| Volume | Workload allocation |
| Thin / Thick | Capacity allocation policy |
| LUN | Storage presentation |
| Host Mapping | Access control |
| Linux | Storage discovery |
| Filesystem | Data organization |
| Application | Business workload |

---

# Validation Perspective

A Storage Validation Engineer validates every stage of this lifecycle.

Typical validation sequence

- Verify RAID creation
- Verify Storage Pool creation
- Verify Volume creation
- Verify Thin and Thick behavior
- Verify LUN creation
- Verify Host Mapping
- Verify Linux discovery
- Verify filesystem creation
- Verify application I/O
- Verify cleanup and capacity recovery

The objective is to ensure that every provisioning stage functions correctly both independently and as part of the complete storage provisioning workflow.

---

# Key Takeaways

- Storage Provisioning begins with a business requirement, not with technical configuration.
- Every provisioning object has a specific engineering responsibility.
- Storage becomes usable only after the complete provisioning lifecycle is finished.
- Storage administration, Linux administration, and application deployment are tightly integrated.
- Understanding the complete workflow is essential for enterprise storage engineering, validation, automation, and troubleshooting.

===========================================================================================================
# Chapter 9 - Enterprise Best Practices

## Introduction

Creating Storage Pools, Volumes, LUNs, and Host Mappings is only one part of enterprise storage administration.

A well-designed storage environment must also be:

- Consistent
- Scalable
- Secure
- Easy to maintain
- Easy to troubleshoot
- Easy to automate

Enterprise best practices ensure that storage remains manageable throughout its lifecycle, from initial deployment to future expansion, maintenance, and retirement.

---

# Capacity Planning

Storage provisioning should always begin with capacity planning rather than immediately creating storage objects.

Capacity planning considers:

- Current application requirements
- Expected data growth
- Performance requirements
- Future expansion
- Business continuity requirements

Example

```text
Current Database Size : 25 TB

Expected Annual Growth : 8 TB

Retention Requirement : 5 Years

Required Capacity

25 + (8 × 5)

= 65 TB

Recommended Provisioned Capacity

80 TB
```

Planning for future growth reduces the need for frequent capacity expansions.

---

# Choose the Correct Provisioning Type

Provisioning policies should always match application requirements.

| Workload | Recommended Provisioning |
|----------|--------------------------|
| Oracle Production | Thick |
| SQL Server | Thick |
| Banking Applications | Thick |
| SAP HANA | Thick |
| VMware | Thin |
| Development | Thin |
| Test | Thin |
| Backup Repository | Thin |
| Virtual Desktop Infrastructure (VDI) | Thin |

Choosing the wrong provisioning policy may lead to wasted capacity or unexpected storage shortages.

---

# Use Meaningful Naming Conventions

Enterprise environments may contain thousands of storage objects.

Clear and consistent naming conventions improve administration and troubleshooting.

Examples

```text
Storage Pools

PROD_POOL
DEV_POOL
BACKUP_POOL
```

```text
Volumes

Oracle_PROD_VOL01
SQL_PROD_VOL01
VMWARE_DS01
BACKUP_VOL01
```

```text
LUNs

LUN_001
LUN_002
LUN_003
```

Avoid generic names such as:

```text
Volume1

Pool2

Test

NewVolume
```

---

# Monitor Capacity Utilization

Capacity should be monitored continuously.

Typical metrics include:

- Total Capacity
- Allocated Capacity
- Used Capacity
- Available Capacity
- Thin Provisioning Utilization
- Storage Pool Growth Rate

Running out of capacity in production can result in application outages.

---

# Prevent Thin Provisioning Overcommitment

Thin Provisioning improves storage utilization but must be monitored carefully.

Example

```text
Physical Capacity

100 TB
```

Allocated

```text
Oracle

40 TB

VMware

80 TB

Backup

60 TB
```

Logical Allocation

```text
180 TB
```

Physical Capacity

```text
100 TB
```

Although oversubscription is expected with Thin Provisioning, administrators must ensure physical capacity is expanded before exhaustion occurs.

---

# Separate Workloads

Applications with different performance or availability requirements should not always share the same Storage Pool.

Example

```text
Production_Pool

Oracle

SQL Server
```

```text
Development_Pool

Development

Testing
```

```text
Backup_Pool

Backup Repository
```

Separating workloads improves performance management, maintenance, and troubleshooting.

---

# Validate Before Production

Every provisioning activity should be verified before handing storage to application teams.

Typical validation includes:

- RAID Health
- Pool Health
- Volume Health
- Provisioning Policy
- LUN Presentation
- Host Mapping
- Operating System Discovery
- Filesystem Creation
- Read/Write Testing

Validation reduces production incidents.

---

# Maintain Accurate Documentation

Enterprise documentation should record:

- RAID Layout
- Storage Pool Configuration
- Volume Inventory
- LUN Inventory
- Host Mapping Information
- Capacity Reports
- Expansion History
- Change History

Accurate documentation simplifies troubleshooting, audits, and future capacity planning.

---

# Automate Repetitive Tasks

Large enterprise environments often provision hundreds of storage objects.

Automation improves:

- Consistency
- Speed
- Repeatability
- Accuracy

Typical automation tasks include:

- Volume Creation
- LUN Creation
- Host Mapping
- Capacity Reporting
- Health Monitoring
- Inventory Collection

Automation reduces manual configuration errors.

---

# Common Mistakes

Examples of common provisioning mistakes include:

- Creating storage without capacity planning.
- Selecting the wrong provisioning policy.
- Ignoring Thin Provisioning utilization.
- Using inconsistent naming conventions.
- Mapping LUNs to incorrect hosts.
- Forgetting to validate Linux storage discovery.
- Failing to document storage changes.
- Deleting storage objects without verifying dependencies.

These mistakes can result in service interruptions and data availability issues.

---

# Engineering Principles

A successful Storage Administrator should always remember:

- Storage follows business requirements.
- Capacity must be planned before provisioning.
- Security is enforced through controlled Host Mapping.
- Every storage object should have a defined purpose.
- Storage should remain scalable throughout its lifecycle.
- Documentation is as important as implementation.
- Automation improves consistency but never replaces engineering validation.

---

# Key Takeaways

Enterprise Storage Provisioning is not simply creating storage objects.

It is the disciplined engineering practice of designing, provisioning, validating, securing, documenting, and maintaining storage infrastructure that supports critical business applications safely and efficiently.
===========================================================================================================
# Chapter 10 - Module Summary

## Module Overview

Storage Provisioning is the engineering process of transforming physical storage resources into secure, organized, and application-ready enterprise storage.

Throughout this module, we followed the complete lifecycle of enterprise storage provisioning—from the moment a business requirement is received until an application successfully begins storing data.

Rather than viewing Storage Provisioning as a collection of independent features, this module presented it as a coordinated engineering workflow in which every logical object has a clearly defined responsibility.

---

# What We Learned

We began by understanding why Storage Provisioning exists.

A healthy storage array is not automatically usable by applications.

Although the hardware, controllers, firmware, and physical drives may all be operational, applications cannot access storage until logical storage resources have been created and presented.

Storage Provisioning bridges this gap by transforming raw storage into application-ready storage.

---

## Enterprise Provisioning Lifecycle

Throughout this module, we built the complete enterprise provisioning workflow.

```text
Business Requirement
        │
        ▼
Capacity Planning
        │
        ▼
RAID Creation
        │
        ▼
Storage Pool
        │
        ▼
Volume
        │
        ▼
Thin / Thick Provisioning
        │
        ▼
LUN
        │
        ▼
Host Mapping
        │
        ▼
Operating System Discovery
        │
        ▼
Filesystem Creation
        │
        ▼
Application Storage
```

Each stage introduces a new logical abstraction while solving a specific engineering problem.

Together, these stages transform physical hardware into reliable enterprise storage.

---

# Engineering Responsibilities

During this module we studied the responsibility of every provisioning component.

| Component | Engineering Responsibility |
|-----------|----------------------------|
| RAID | Provides redundancy and fault tolerance |
| Storage Pool | Centralized capacity management |
| Volume | Logical storage allocation for workloads |
| Thin / Thick Provisioning | Capacity allocation strategy |
| LUN | Storage presentation to hosts |
| Host Mapping | Secure host access control |
| Operating System | Storage discovery and filesystem management |
| Application | Business data storage |

Understanding the responsibility of each layer is essential for effective storage administration and troubleshooting.

---

# Engineering Mindset

Enterprise Storage Provisioning is more than creating storage objects.

A Storage Engineer must consider:

- Business requirements
- Capacity planning
- Performance requirements
- Data protection
- Scalability
- Security
- Future expansion
- Operational simplicity
- Validation
- Documentation

Successful provisioning balances technical implementation with long-term operational reliability.

---

# Validation Perspective

From a Storage Validation Engineer's perspective, every provisioning stage represents a feature that must be tested.

Validation activities include:

- RAID validation
- Storage Pool validation
- Volume validation
- Thin and Thick Provisioning validation
- LUN validation
- Host Mapping validation
- Linux storage discovery
- End-to-end functional testing
- Capacity accounting
- Error handling
- Boundary testing
- Regression testing

The objective is to ensure that every provisioning feature functions correctly both independently and as part of the complete storage lifecycle.

---

# Key Engineering Principles

This module established several fundamental engineering principles that apply throughout enterprise storage.

- A healthy storage array is not automatically application-ready.
- Every logical storage object exists to solve a specific engineering problem.
- Storage is provisioned according to business requirements rather than hardware availability.
- Provisioning policies should always match workload characteristics.
- Storage must be validated before being released to production.
- Documentation and automation are essential components of enterprise storage management.

These principles form the foundation for every advanced storage technology discussed in later modules.

---

# Transition to Module 02

Throughout this module, we repeatedly created RAID Groups before creating Storage Pools.

A natural question now arises.

**Why is RAID always created first?**

The answer is simple.

Everything built in this module depends on RAID.

Storage Pools depend on RAID.

Volumes depend on Storage Pools.

LUNs depend on Volumes.

Host Mapping depends on LUNs.

If RAID cannot protect data, every layer above it also becomes unreliable.

The next module therefore focuses on the foundation of enterprise storage reliability.

In **Module 02 – RAID and Data Protection**, we will study:

- Why RAID was invented
- Problems with standalone disks
- RAID architecture
- Striping
- Mirroring
- Parity
- RAID Levels (0, 1, 5, 6, 10, 50, 60)
- Hot Spares
- RAID Rebuild Operations
- Failure Scenarios
- Enterprise RAID Design
- Validation and Troubleshooting

Understanding RAID is essential because every logical storage object created in this module ultimately depends on the protection provided by RAID.

---

# Final Takeaway

Storage Provisioning is not the act of creating a Storage Pool, Volume, or LUN.

It is the complete engineering workflow that transforms physical storage infrastructure into secure, manageable, scalable, and application-ready enterprise storage.

Every provisioning decision—from RAID selection to Host Mapping—directly influences the reliability, performance, security, and availability of enterprise applications.

A strong understanding of Storage Provisioning provides the foundation upon which all advanced enterprise storage technologies are built.


