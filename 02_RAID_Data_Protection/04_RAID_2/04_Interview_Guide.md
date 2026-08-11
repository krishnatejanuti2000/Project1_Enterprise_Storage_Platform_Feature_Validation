# RAID 2 – Interview Guide

## Module

Project 1 – Enterprise Storage Platform Feature Validation

Module 2 – RAID Data Protection

Topic: RAID 2

---

# 1. What is RAID 2?

RAID 2 is a historical RAID level that uses bit-level striping together with Hamming-code-based ECC to detect and correct certain bit-level errors.

---

# 2. What is the primary purpose of RAID 2?

The primary purpose of RAID 2 is bit-level error detection and correction.

---

# 3. What type of striping does RAID 2 use?

RAID 2 uses bit-level striping.

Individual data bits are distributed across multiple data disks.

---

# 4. What does ECC mean?

ECC stands for:

Error Correcting Code

ECC provides additional information that can be used to detect and correct certain data errors.

---

# 5. What type of ECC does RAID 2 use?

RAID 2 historically uses Hamming-code-based ECC.

---

# 6. What do ECC disks store?

ECC disks store error-correcting information calculated from the data.

They do not store complete mirror copies of the application data like RAID 1.

---

# 7. What is the conceptual minimum number of drives for RAID 2?

A theoretical minimum configuration can be represented as:

    1 Data Disk
    +
    2 ECC Disks
    =
    3 Disks

However, actual RAID 2 configurations require more disks depending on the data width and ECC requirements.

---

# 8. How is RAID 2 different from RAID 1?

RAID 1 uses mirroring.

    Disk A → Complete Data
    Disk B → Complete Copy

RAID 2 uses bit-level striping and ECC.

    Data Disks → Data Bits
    ECC Disks  → Error-Correcting Information

Therefore:

    RAID 1 → Data redundancy through mirroring

    RAID 2 → Error correction through ECC

---

# 9. How does RAID 2 perform a write?

During a write:

1. The RAID controller receives the application data.
2. Data is distributed at the bit level.
3. ECC information is calculated.
4. Data bits are written to the data disks.
5. ECC information is written to the ECC disks.

---

# 10. How does RAID 2 perform a read?

During a read:

1. Data is retrieved from the data disks.
2. ECC information is retrieved.
3. The ECC information is checked against the data.
4. If no error exists, the data is returned.
5. If a correctable error exists, the incorrect bit is identified and corrected.
6. Corrected data is returned to the application.

---

# 11. What happens when a bit-level error occurs?

The ECC information is used to detect the error and, when the error is within the correction capability of the ECC scheme, identify the incorrect bit.

The incorrect bit can then be corrected before the data is returned.

---

# 12. Does RAID 2 mirror data?

No.

RAID 2 does not maintain complete duplicate copies like RAID 1.

It uses:

    Data Disks → Actual Data
    ECC Disks  → Error-Correcting Information

---

# 13. Why does RAID 2 require multiple disks?

RAID 2 distributes data at the bit level and requires additional disks for ECC information.

Therefore, multiple data disks and dedicated ECC disks are required.

---

# 14. What are the major disadvantages of RAID 2?

Major disadvantages include:

- Requires multiple disks.
- Requires dedicated ECC disks.
- Bit-level striping is complex.
- Requires tightly coordinated disk operation.
- Has significant storage overhead.
- Poor practical efficiency compared with later RAID architectures.

---

# 15. Why is RAID 2 considered legacy?

RAID 2 became obsolete because modern HDDs and SSDs already implement sophisticated internal error-correction mechanisms.

Therefore, performing another dedicated Hamming-code-based ECC function at the RAID layer adds complexity and overhead without sufficient practical benefit.

---

# 16. Is RAID 2 commonly used in modern enterprise storage?

No.

RAID 2 is considered a historical RAID level and is not a mainstream modern enterprise RAID implementation.

---

# 17. Why don't we perform a RAID 2 lab?

RAID 2 is a legacy architecture and is not a mainstream modern RAID implementation.

Therefore, this project focuses on:

- Architecture
- ECC
- Hamming Code
- Read flow
- Write flow
- Error correction
- Limitations
- Historical significance

A practical RAID 2 deployment lab is not required.

---

# 18. Interview Scenario

### Question

A customer asks:

"Should we deploy RAID 2 on our modern enterprise storage system?"

### Answer

No.

RAID 2 is a historical RAID architecture. Modern HDDs and SSDs already provide internal error-correction mechanisms, while RAID 2 introduces significant complexity, dedicated ECC disks, and storage overhead.

A modern enterprise RAID design should use an appropriate current RAID architecture instead.

---

# 19. Strong Interview Answer

### Question:

"Explain RAID 2."

### Answer:

RAID 2 is a historical RAID level that uses bit-level striping and dedicated ECC information based on Hamming Code to detect and correct certain bit-level errors. Unlike RAID 1, which provides redundancy through complete mirroring, RAID 2 uses error-correcting information. It became obsolete because modern HDDs and SSDs already provide sophisticated internal ECC, while RAID 2 requires multiple disks and introduces significant architectural complexity and storage overhead.

---

# 20. Key Interview Points

Remember these points:

- Bit-level striping
- Hamming-code-based ECC
- Dedicated ECC disks
- Error detection
- Error correction
- Not mirroring
- High architectural overhead
- Historical RAID level
- Modern drives already provide ECC
- Not used in mainstream modern enterprise storage
