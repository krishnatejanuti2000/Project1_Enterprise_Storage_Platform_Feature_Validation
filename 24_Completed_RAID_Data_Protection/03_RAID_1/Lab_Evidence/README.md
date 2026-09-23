# RAID 1 Validation Lab

## Objective

Validate the complete lifecycle of a RAID 1 (Mirroring) array using Linux Software RAID (`mdadm`).

---

## Validation Workflow

1. Verify available disks
2. Verify existing RAID configuration
3. Remove inactive RAID devices
4. Verify clean member disks
5. Create RAID 1
6. Verify RAID status
7. Verify RAID configuration
8. Create filesystem
9. Mount filesystem
10. Generate test data
11. Simulate disk failure
12. Verify degraded mode
13. Verify data accessibility
14. Remove failed RAID member
15. Add replacement disk
16. Verify RAID recovery
17. Clean up the environment

---

## Key Concepts Validated

- RAID 1 creation
- Mirroring
- Initial resynchronization
- RAID health monitoring
- Filesystem creation
- Mount validation
- Test data generation
- Disk failure simulation
- Degraded mode
- Data accessibility during failure
- Failed disk removal
- Replacement disk addition
- RAID recovery (rebuild)
- Environment cleanup

---

## Validation Result

**Status: PASS**

The RAID 1 array successfully demonstrated:

- Fault tolerance through mirroring.
- Continuous data availability after a single-disk failure.
- Successful removal and replacement of a failed member.
- Automatic recovery (rebuild) to restore redundancy.
- Proper cleanup with removal of RAID metadata.

This lab validates the complete operational lifecycle of a RAID 1 array in an enterprise Linux environment.
