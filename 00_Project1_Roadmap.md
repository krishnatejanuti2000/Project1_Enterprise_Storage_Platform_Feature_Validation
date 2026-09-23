# TRACK 1 — ENTERPRISE STORAGE PLATFORM / FEATURE VALIDATION

## Target Profile

Storage QA Engineer
Storage Validation Engineer
Storage Product Validation Engineer
Server & Storage Validation Engineer
Storage SDET / Storage Automation Engineer

## Target Experience

3–4 years

## Track Objective

Build the capability to validate an enterprise storage platform from platform bring-up through provisioning, host access, feature behavior, performance, failure/recovery, automation and root-cause analysis.

The target is NOT to become an expert administrator of every storage vendor.

The target is to become a technically strong **system-level storage validation engineer**.

---

# PRIORITY MODEL

## P0 — MUST MASTER

These are the subjects that should receive the greatest depth.

* Enterprise Storage Architecture
* Linux Storage / System Layer
* Storage Provisioning + RAID
* Multipathing + High Availability
* Storage Feature Validation
* Fault Injection + Recovery
* Performance / FIO
* Logs / Diagnostics
* Troubleshooting + RCA
* Python Storage Automation
* Pytest / Automation Framework
* Test Strategy / Test Design

## P1 — STRONG WORKING DEPTH

* Server / Platform Architecture
* Platform Bring-Up
* Firmware / BIOS / BMC / CPLD
* SATA
* SAS
* SCSI
* Fibre Channel
* iSCSI
* Data Integrity
* Stress / Reliability
* Compatibility / Qualification
* Git / Jenkins / CI
* VMware Storage

## P2 — AWARENESS / SPECIALIZATION

* Deep vendor-specific CLI knowledge
* Advanced protocol analyzers
* Deep electrical PCIe work
* Cloud-native storage administration
* Kubernetes storage administration
* Ceph
* Deep VMware administration

---

# MODULE 01 — ENTERPRISE STORAGE ARCHITECTURE

## Priority: P0

### 1.1 Storage models

* DAS
* SAN
* NAS
* Storage arrays
* JBOD
* Storage enclosures
* All-flash arrays
* Controller-based storage

### 1.2 End-to-end data path

Understand:

```text
Application
   ↓
Filesystem
   ↓
Operating System
   ↓
Block Layer
   ↓
Driver / Multipath
   ↓
Storage Protocol
   ↓
Storage Controller
   ↓
RAID / Pool
   ↓
Volume
   ↓
LUN
   ↓
Drive
```

### 1.3 Array architecture

* Front-end ports
* Back-end ports
* Controllers
* Controller cache
* Data path
* Management path
* Active/active
* Active/passive
* Controller ownership
* Controller failover/failback

### 1.4 Physical components

* Server
* HBA
* CNA
* NIC
* PCIe
* Backplane
* Drive enclosure
* PSU
* Fans
* Storage controller

### Mastery requirement

Be able to trace a host read/write through the complete storage system and identify every important layer involved.

---

# MODULE 02 — SERVER & PLATFORM ARCHITECTURE

## Priority: P1

### Master

* x86 server architecture
* CPU
* Memory
* NUMA basics
* PCIe root complex
* PCIe topology
* DMA basics
* Interrupt basics
* HBA
* RAID/controller
* NIC/CNA
* backplane
* storage enclosure
* power supply
* thermal basics

### Understand

* host hardware vs storage hardware
* host-side vs array-side components
* hardware dependencies
* firmware dependencies

### Troubleshooting

* PCIe device missing
* HBA missing
* wrong firmware
* driver mismatch
* storage controller not detected

---

# MODULE 03 — PLATFORM BRING-UP / PLATFORM READY

## Priority: P1

### Boot

* POST
* BIOS/UEFI
* boot sequence
* hardware initialization
* device enumeration
* controller initialization
* HBA initialization

### BMC

* out-of-band management
* remote console
* remote power control
* reset
* sensors
* event logs
* IPMI/SEL basics
* BMC firmware

### CPLD

* purpose
* hardware control role
* firmware relationship

### Platform readiness

Understand:

```text
Hardware initialized
+
Firmware initialized
+
Controllers initialized
+
Devices enumerated
+
Health checks passed
=
Platform Ready
```

### Failure scenarios

* boot failure
* device enumeration failure
* BMC unavailable
* controller initialization failure
* configuration persistence failure
* reset failure
* power-cycle recovery failure

---

# MODULE 04 — LINUX STORAGE & SYSTEM LAYER

## Priority: P0

This should become one of your strongest subjects.

### Linux storage path

* userspace
* filesystem
* block layer
* device-mapper
* SCSI layer
* driver
* block device
* device node
* udev
* sysfs

### blk-mq

* purpose
* software queues
* hardware queues
* parallel I/O
* queueing relationship to modern SSDs

Linux's blk-mq documentation describes the multi-queue layer as the mechanism used to exploit parallelism in modern storage devices, with software staging queues and hardware dispatch queues.

### Discovery

* `lsblk`
* `lspci`
* `lsscsi`
* `blkid`
* `udevadm`
* `/sys`
* `/dev`

### Diagnostics

* `dmesg`
* `journalctl`
* kernel logs
* driver messages
* timeout messages
* reset messages
* I/O errors

### Storage utilities

* `smartctl`
* `storcli`
* `mdadm`
* `multipath`
* `nvme-cli`

### Mastery requirement

Be able to answer:

> The device is physically present. Which layer do I check next and why?

---

# MODULE 05 — STORAGE PROVISIONING

## Priority: P0

### Complete flow

```text
Physical Drives
     ↓
RAID
     ↓
Storage Pool
     ↓
Volume
     ↓
LUN
     ↓
Host Mapping
     ↓
Host Discovery
     ↓
Filesystem
     ↓
Application
```

### RAID

* purpose
* RAID 0
* RAID 1
* RAID 5
* RAID 6
* RAID 10
* striping
* mirroring
* parity
* usable capacity
* fault tolerance

### RAID operations

* creation
* deletion
* degraded state
* rebuild
* hot spare
* hot swap
* rebuild interruption
* rebuild failure
* consistency check
* capacity expansion
* RAID-level migration

### Pool

* creation
* capacity
* allocation
* expansion
* free capacity
* health

### Volume

* create
* resize
* delete
* snapshot
* clone

### LUN

* create
* mapping
* masking
* host group
* initiator association
* presentation

### Mastery requirement

Be able to explain exactly why:

```text
RAID → Pool → Volume → LUN → Host
```

are different layers.

---

# MODULE 06 — SCSI

## Priority: P1

### Master

* initiator
* target
* LUN
* CDB
* command model
* Inquiry
* Read
* Write
* Test Unit Ready
* status
* sense data
* Check Condition

### Understand the transport relationship

```text
SCSI command
    ↓
SAS transport
```

and:

```text
SCSI command
    ↓
iSCSI transport
```

### Troubleshooting

* command failure
* sense data
* device not ready
* transport problem
* target/path failure

---

# MODULE 07 — SAS

## Priority: P1

### Architecture

* initiator
* target
* SAS domain
* SAS HBA
* SAS expander
* ports
* dual-port concept
* discovery
* addressing

### Validation

* device discovery
* link state
* compatibility
* firmware
* hot plug
* path redundancy
* error handling

### Deep understanding

Be able to explain why enterprise systems use SAS and how a SAS I/O path differs from a SATA path.

---

# MODULE 08 — SATA

## Priority: P1

SATA should be learned as a protocol, not a comparison-table topic.

### Architecture

* why SATA exists
* host/device model
* controller
* link
* AHCI
* command transport

### Protocol concepts

* link initialization
* command issue
* command completion
* FIS
* data transfer
* status/error reporting

### Validation

* device discovery
* link validation
* command execution
* hot plug
* reset
* device disappearance
* recovery

### Troubleshooting

Be able to investigate:

```text
Device present
 ↓
Controller detection
 ↓
OS detection
 ↓
Link state
 ↓
Command execution
 ↓
I/O
```

---

# MODULE 09 — FIBRE CHANNEL + iSCSI

## Priority: P1

## Fibre Channel

### Master

* initiator
* target
* HBA
* switch
* fabric
* WWPN
* WWNN
* zoning
* LUN masking

### Login concepts

* FLOGI
* PLOGI
* PRLI

### Storage path

```text
Host
 ↓
FC HBA
 ↓
FC Fabric
 ↓
Storage Front-End
 ↓
Controller
 ↓
LUN
```

### Validation

* discovery
* zoning
* LUN presentation
* multipathing
* path failure
* failover
* failback

## iSCSI

### Master

* initiator
* target
* IQN
* discovery
* login
* session
* connection
* TCP relationship
* CHAP basics
* LUN presentation

### Validation

* discovery
* login
* connectivity
* multipath
* path failure
* recovery

---

# MODULE 10 — MULTIPATH + HIGH AVAILABILITY

## Priority: P0

### Master

* purpose of multipathing
* redundant paths
* active/active
* active/passive
* path states
* path selection
* failover
* failback
* path recovery
* controller failover
* controller failback
* continuous I/O

### Scenario

```text
Continuous I/O
     ↓
Path failure
     ↓
Failure detection
     ↓
Surviving path selected
     ↓
I/O continues
     ↓
Failed path recovers
     ↓
Path returns
```

### Required troubleshooting

Understand what happens at:

* host
* multipath layer
* transport
* controller
* array

when one path fails.

---

# MODULE 11 — STORAGE FEATURE VALIDATION

## Priority: P0

### Validation framework

For every feature:

```text
Requirement
 ↓
Preconditions
 ↓
Configuration
 ↓
Positive Tests
 ↓
Negative Tests
 ↓
Boundary Tests
 ↓
Failure Injection
 ↓
Expected Result
 ↓
Actual Result
 ↓
Evidence
 ↓
Recovery
 ↓
Regression
```

### Features

* RAID
* pools
* volumes
* LUNs
* snapshots
* clones
* replication
* remote copy
* deduplication
* tiering
* HA
* failover
* firmware features
* health monitoring

---

# MODULE 12 — TEST ENGINEERING

## Priority: P0

### Master

* requirements analysis
* acceptance criteria
* test scenarios
* test cases
* test conditions
* test data
* expected results
* positive testing
* negative testing
* boundary testing
* functional testing
* integration testing
* system testing
* regression
* re-testing
* compatibility
* stress
* recovery
* performance
* endurance
* coverage

### 3–4 year expectation

You should be able to explain:

> Given a new storage feature, how would you decide what to test?

Current Micron SSD validation work explicitly includes defining validation strategy, test cases, coverage and PASS/FAIL criteria; current SanDisk validation roles likewise emphasize test methodology and validation coverage.

---

# MODULE 13 — FAULT INJECTION + RECOVERY

## Priority: P0

### Inject

* drive failure
* controller reset
* controller reboot
* BMC reset
* BIOS reset
* system reboot
* power cycle
* path failure
* HBA failure
* link failure
* firmware failure
* hot removal
* hot insertion

### Validate

* detection
* expected behavior
* I/O behavior
* failover
* recovery
* data integrity
* final health
* regression

### Every failure follows

```text
Healthy State
 ↓
Failure Injection
 ↓
Observe
 ↓
Collect Evidence
 ↓
Expected vs Actual
 ↓
Recover
 ↓
Verify
 ↓
Regression
```

---

# MODULE 14 — PERFORMANCE ENGINEERING WITH FIO

## Priority: P0

### fio fundamentals

* job files
* target
* workload
* block size
* I/O size
* sequential/random
* read/write
* mixed workload
* iodepth
* jobs
* runtime
* direct I/O
* I/O engine
* verification

### Metrics

* IOPS
* throughput
* bandwidth
* latency
* latency percentiles
* CPU utilization
* achieved I/O depth

### Workloads

* random read
* random write
* sequential read
* sequential write
* mixed
* different block sizes
* different queue depths
* different concurrency

### Advanced working concepts

* steady state
* baseline
* repeatability
* performance regression
* bottleneck analysis

fio's current documentation explicitly exposes I/O depth, latency targeting/percentiles, steady-state assessment, verification and measurement/reporting.

---

# MODULE 15 — DATA INTEGRITY

## Priority: P1

### Master

* data patterns
* known data
* read-after-write
* checksum
* corruption detection
* persistence
* reboot integrity
* power-cycle integrity
* failover integrity
* rebuild integrity
* firmware-change integrity

### Critical distinction

```text
I/O completed
≠
Data is correct
```

---

# MODULE 16 — STRESS / RELIABILITY / ENDURANCE

## Priority: P1

### Stress

* sustained I/O
* mixed workloads
* high queue depth
* concurrent workloads
* repeated operations

### Reliability

* reboot cycles
* reset cycles
* power cycles
* path failures
* recovery cycles

### Endurance

* long duration
* repeated writes
* stability
* degradation
* performance consistency

---

# MODULE 17 — COMPATIBILITY / QUALIFICATION

## Priority: P1

### Matrix

```text
Platform
×
Controller
×
Drive
×
Firmware
×
BIOS
×
HBA
×
OS
×
Driver
×
Protocol
```

### Validate

* supported combinations
* unsupported combinations
* firmware compatibility
* driver compatibility
* OS compatibility
* hardware compatibility
* upgrade compatibility
* downgrade compatibility
* regression

---

# MODULE 18 — LOGS / DIAGNOSTICS / RCA

## Priority: P0

This is one of the defining engineering skills of the track.

### Sources

* host OS
* kernel
* controller
* RAID
* storage array
* drive
* BMC
* firmware
* HBA
* multipath
* performance tools

### Investigation sequence

```text
Symptom
 ↓
Reproduce
 ↓
Define exact failure
 ↓
Capture configuration
 ↓
Identify affected layer
 ↓
Collect logs
 ↓
Compare healthy vs failed
 ↓
Form hypothesis
 ↓
Test hypothesis
 ↓
Isolate
 ↓
Root Cause
 ↓
Fix verification
 ↓
Regression
```

### Mastery requirement

You should be able to defend **why each diagnostic command or log source was selected**.

Current validation roles repeatedly emphasize failure analysis, debugging across layers, systematic reproduction and root-cause resolution.

---

# MODULE 19 — PYTHON FOR STORAGE AUTOMATION

## Priority: P0

### Python fundamentals required for engineering

* functions
* OOP
* data structures
* exceptions
* file handling
* strings
* regular expressions
* JSON
* CSV
* modules/packages
* logging
* configuration handling

### System automation

* `subprocess`
* process management
* return codes
* stdout/stderr
* timeout
* retries

### Useful standard libraries

* `os`
* `sys`
* `subprocess`
* `argparse`
* `logging`
* `datetime`
* `json`
* `re`
* `hashlib`

### SSH / remote execution

* SSH
* Paramiko
* remote command execution
* remote logs
* error handling

### Storage automation targets

* environment checks
* device discovery
* RAID verification
* provisioning checks
* health checks
* FIO execution
* result parsing
* log collection
* firmware verification

---

# MODULE 20 — PYTEST / STORAGE AUTOMATION FRAMEWORK

## Priority: P0

### Pytest

* test discovery
* fixtures
* fixture scope
* parametrization
* markers
* assertions
* setup
* teardown
* reusable utilities
* configuration
* reporting
* failure handling

### Framework design

```text
storage_validation/
│
├── tests/
├── devices/
├── storage/
├── fio/
├── firmware/
├── utilities/
├── config/
├── logs/
└── reports/
```

### Framework principles

* reusable
* modular
* parameterized
* scalable
* configurable
* failure-safe
* log-rich

### Automation examples

#### Platform

```text
Platform Ready Check
→ RAID Check
→ Pool Check
→ LUN Check
→ Host Connectivity
→ FIO
→ Logs
→ Result
```

#### Failure

```text
Inject Failure
→ Observe
→ Collect Logs
→ Validate Recovery
→ Generate Result
```

SanDisk's current 3–5 year Bengaluru role specifically requires developing, debugging and maintaining automated test frameworks and automated validation solutions.

---

# MODULE 21 — GIT / JENKINS / CI REGRESSION

## Priority: P1

### Git

* repository
* branch
* commit
* merge
* pull
* push
* basic branching workflow

### Jenkins

* jobs
* parameters
* scheduled runs
* environment setup
* test execution
* result collection
* reporting
* failure reporting

### CI flow

```text
New Build / Firmware
 ↓
Testbed Setup
 ↓
Automated Tests
 ↓
Log Collection
 ↓
Result Analysis
 ↓
Defect
 ↓
Fix
 ↓
Regression
```

---

# MODULE 22 — VMWARE FOR STORAGE VALIDATION

## Priority: P1

Learn only the storage-relevant portion.

### Master

* ESXi
* vCenter
* VM
* datastore
* virtual disk
* physical storage relationship
* snapshots
* clones
* templates
* HA
* FT
* vMotion
* Storage vMotion
* storage visibility
* multipathing concepts

### Do NOT become

* full VMware administrator
* VMware networking specialist
* VMware security specialist

---

# TRACK 1 — DEFINITION OF DONE

You should eventually be able to receive this requirement:

> "Validate a new enterprise storage-platform firmware release."

And independently explain:

```text
Prepare Testbed
 ↓
Platform Readiness
 ↓
Storage Discovery
 ↓
RAID
 ↓
Pool
 ↓
Volume
 ↓
LUN
 ↓
Host Mapping
 ↓
FC/iSCSI Connectivity
 ↓
Functional Tests
 ↓
FIO Performance
 ↓
Fault Injection
 ↓
Recovery
 ↓
Logs
 ↓
RCA
 ↓
Python/Pytest Automation
 ↓
Jenkins Regression
 ↓
Defect Closure
 ↓
Release Qualification
```

That is the level of capability this track is intended to produce.


=====================================================================================================
=====================================================================================================
=====================================================================================================


# Project 1 – Enterprise Storage Platform Feature Validation

## Project Objective

Validate that the complete Enterprise Storage Platform functions correctly, reliably, performs as expected, and remains highly available under all supported customer deployment scenarios.

This project focuses on validating the **entire storage platform**, not individual storage drives. The goal is to ensure that enterprise storage features work correctly across hardware, firmware, operating systems, storage protocols, and customer workloads.

---

# Project Scope

## Module 1 – Storage Provisioning

### Objective
Validate storage provisioning and capacity management features.

### Topics
- Storage Pools
- Volumes
- LUNs
- Thin Provisioning
- Thick Provisioning
- Capacity Expansion
- Volume Expansion
- LUN Mapping
- LUN Masking

---

## Module 2 – RAID & Data Protection Validation

### Objective
Validate RAID functionality and ensure data protection during normal and failure scenarios.

### Topics
- RAID Creation
- RAID Deletion
- RAID Expansion
- RAID Migration
- RAID Rebuild
- Hot Spare
- Degraded Mode
- Recovery Validation
- Consistency Check
- Background Initialization

---

## Module 3 – High Availability (HA) Validation

### Objective
Validate uninterrupted storage availability during component failures.

### Topics
- Controller Failover
- Controller Failback
- Dual Controller Operation
- High Availability Validation
- Controller Reboot
- Power Cycle Validation
- Power Supply Failure
- Network Port Failure

---

## Module 4 – Host Connectivity Validation

### Objective
Validate communication between enterprise hosts and storage arrays.

### Topics
- Fibre Channel (FC)
- iSCSI
- Multipathing
- Path Failover
- Host Discovery
- Device Rescan
- Host Connectivity Validation

---

## Module 5 – Firmware & Software Lifecycle Validation

### Objective
Validate firmware/software lifecycle without impacting platform stability.

### Topics
- Firmware Upgrade
- Firmware Downgrade
- Firmware Rollback
- Compatibility Validation
- Mixed Firmware Validation
- Platform Recovery after Upgrade

---

## Module 6 – Performance Validation

### Objective
Measure storage platform performance under enterprise workloads.

### Tools
- FIO
- IOmeter

### Metrics
- IOPS
- Throughput
- Latency
- CPU Utilization
- Controller Utilization

---

## Module 7 – Stability & Regression Validation

### Objective
Ensure platform stability after feature enhancements and software changes.

### Topics
- Functional Regression
- Performance Regression
- Stress Testing
- Soak Testing
- Long Duration Testing
- Stability Validation

---

## Module 8 – Automation Framework

### Objective
Automate repetitive storage validation activities.

### Technologies
- Python
- Pytest
- Paramiko
- Jenkins
- Git

### Activities
- Test Automation
- Remote Execution
- Result Validation
- Log Collection
- Report Generation

---

## Module 9 – Diagnostics & Log Analysis

### Objective
Collect and analyze diagnostic information for failure investigation.

### Topics
- Linux Logs
- Storage Logs
- Controller Logs
- System Logs
- Event Logs
- SMART Information (Platform Perspective)

---

## Module 10 – Troubleshooting & Root Cause Analysis

### Objective
Identify, isolate, reproduce, and verify storage platform defects.

### Topics
- Failure Reproduction
- Failure Isolation
- Root Cause Analysis
- Bug Verification
- Jira Workflow
- Regression Verification
- Fix Validation

---

# Validation Methodologies (Applied Across All Modules)

- Functional Testing
- Regression Testing
- Compatibility Testing
- Performance Testing
- Stress Testing
- Endurance Testing
- Recovery Testing
- Upgrade Validation
- Negative Testing
- Sanity Testing

---

# Enterprise QA Workflow

Requirement Analysis

↓

Test Planning

↓

Test Case Design

↓

Automation Development

↓

Test Execution

↓

Failure Detection

↓

Log Collection

↓

Issue Analysis

↓

Bug Reporting (Jira)

↓

Developer Fix

↓

Fix Verification

↓

Regression Testing

↓

Release Validation

---

# Four Core Pillars

Every module in this project will be studied from four perspectives:

1. QA & Validation
2. Automation
3. Performance
4. Troubleshooting & Root Cause Analysis

---

# Standard Learning Template (Used for Every Module)

Every module will be reverse-engineered using the following structure:

1. Architecture
2. Why the Feature Exists
3. Internal Workflow
4. Customer Use Cases
5. QA Responsibilities
6. Validation Scenarios
7. Automation Strategy
8. Performance Considerations
9. Common Failure Scenarios
10. Troubleshooting Approach
11. Root Cause Analysis
12. Interview Questions
13. Advanced Cross-Questions

---

# End Goal

Develop the knowledge and troubleshooting skills required to perform as an Enterprise Storage Platform Feature Validation Engineer capable of handling real-world validation, automation, performance analysis, defect investigation, and technical interviews with confidence.
