# RAID 6 — Interview Guide

## 1. What is RAID 6?

RAID 6 is a fault-tolerant RAID level that uses block-level
striping and two independent parity calculations.

It can normally tolerate up to two member failures.

```text
RAID 6
    ↓
Data + P + Q
    ↓
Two independent parity relationships
    ↓
Two-member fault tolerance
```

---

# 2. Why was RAID 6 introduced?

RAID 5 provides only one-member fault tolerance.

During a RAID 5 rebuild, another member failure can exceed the
available redundancy.

RAID 6 addresses this problem by adding a second independent parity
calculation.

```text
RAID 5 → one-member fault tolerance
RAID 6 → two-member fault tolerance
```

---

# 3. What is the main difference between RAID 5 and RAID 6?

| Feature               | RAID 5           | RAID 6            |
| --------------------- | ---------------- | ----------------- |
| Parity relationships  | 1                | 2                 |
| P parity              | Yes              | Yes               |
| Q parity              | No               | Yes               |
| Data protection       | 1 member failure | 2 member failures |
| Minimum members       | 3                | 4                 |
| Usable capacity       | `(N - 1) × size` | `(N - 2) × size`  |
| Write/parity overhead | Lower            | Higher            |

---

# 4. What are P and Q in RAID 6?

RAID 6 uses two parity values:

```text
P → XOR parity
Q → Reed–Solomon parity
```

P provides one parity relationship.

Q provides a second independent parity relationship.

Together they provide the information required to recover from
two failed members.

---

# 5. How is P parity calculated?

P uses XOR.

For data blocks A, B, C and D:

```text
P = A XOR B XOR C XOR D
```

Example:

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
```

Result:

```text
P = 00000101
```

---

# 6. How is Q parity calculated?

Q uses Reed–Solomon coding over a Galois Field.

For four data blocks:

```text
Q =
(A × g^0)
XOR
(B × g^1)
XOR
(C × g^2)
XOR
(D × g^3)
```

The data positions use different powers of the generator.

---

# 7. What is Reed–Solomon coding doing in RAID 6?

Reed–Solomon coding provides the second independent parity
relationship.

It is used for Q parity.

Conceptually:

```text
Q
↓
Reed–Solomon coding
↓
Galois Field arithmetic
```

This makes Q mathematically different from P.

---

# 8. What is GF(2^8)?

`GF` means Galois Field.

```text
GF(2^8)
```

contains:

```text
2^8 = 256
```

possible field elements.

The elements are represented as 8-bit values.

Example:

```text
10110110
```

can be represented as:

```text
x^7 + x^5 + x^4 + x^2 + x
```

---

# 9. What does addition mean inside GF(2)?

Inside the binary Galois Field:

```text
addition = XOR
```

Example:

```text
1011 XOR 0101
= 1110
```

There is no normal binary carry.

---

# 10. What is the generator used for RAID 6 Q parity?

For the Linux RAID-6 field representation used in our calculation:

```text
g = 0x02
```

In 8-bit binary:

```text
g = 00000010
```

The generator is a predefined mathematical parameter of the
coding scheme.

The RAID implementation does not discover it from the disks.

---

# 11. What are the powers of g?

For the initial coefficients:

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

These powers are used as coefficients for the corresponding data
positions in the Q calculation.

---

# 12. How is GF multiplication different from normal multiplication?

GF multiplication uses polynomial arithmetic over GF(2).

An 8-bit value represents a polynomial.

For example:

```text
00001011
```

represents:

```text
x^3 + x + 1
```

The multiplication is performed using the field's arithmetic rules
and then reduced back into the GF(2^8) field when necessary.

It is not ordinary integer multiplication.

---

# 13. What is field reduction?

During GF multiplication, powers such as:

```text
x^8
x^9
```

can appear.

GF(2^8) requires the final result to be represented within the
8-bit field.

The field-defining polynomial used in our calculation is:

```text
x^8 + x^4 + x^3 + x^2 + 1
```

Therefore:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

and:

```text
x^9 = x^5 + x^4 + x^3 + x
```

These relationships are used to reduce higher powers back to valid
field elements.

---

# 14. Why is Q independent from P?

P is:

```text
P = A XOR B XOR C XOR D
```

Q applies different coefficients to the data positions:

```text
Q =
A×g^0
XOR
B×g^1
XOR
C×g^2
XOR
D×g^3
```

Therefore Q contains a different mathematical relationship from P.

This independence is what allows RAID 6 to recover two unknown members.

---

# 15. Can P alone recover two failed data members?

No.

Suppose:

```text
A    B    C❌    D❌    P    Q
```

P provides only one relationship between the two unknowns.

Conceptually:

```text
P
↓
C XOR D = known value
```

That does not uniquely determine both C and D.

Q provides the second independent relationship.

---

# 16. How does RAID 6 recover two failed members?

With two missing members:

```text
A    B    C❌    D❌    P    Q
```

the RAID system uses:

```text
P → first equation
Q → second independent equation
```

Therefore:

```text
2 unknowns
+
2 independent equations
=
recover both
```

---

# 17. What happens when one data member fails?

Example:

```text
A    B    C❌    D    P    Q
```

P is sufficient for the basic reconstruction:

```text
C = A XOR B XOR D XOR P
```

The reconstructed data can be returned to the host.

Q is not required for this basic single-data-member reconstruction.

---

# 18. What happens when P parity fails?

Example:

```text
A    B    C    D    P❌    Q
```

The data blocks are still available.

Regenerate P:

```text
P = A XOR B XOR C XOR D
```

Data itself is not lost because a parity member failed.

---

# 19. What happens when Q parity fails?

Example:

```text
A    B    C    D    P    Q❌
```

The data blocks remain available.

Q can be regenerated using the Reed–Solomon calculation:

```text
Q =
(A × g^0)
XOR
(B × g^1)
XOR
(C × g^2)
XOR
(D × g^3)
```

---

# 20. What happens if one data member and P fail?

Example:

```text
A    B    C❌    D    P❌    Q
```

Use the surviving Q relationship to recover C.

Then regenerate P:

```text
C → reconstructed
        ↓
A B C D
        ↓
P = A XOR B XOR C XOR D
```

Recovery sequence:

```text
Q → recover data
↓
regenerate P
```

---

# 21. What happens if one data member and Q fail?

Example:

```text
A    B    C    D❌    P    Q❌
```

Use P to recover the missing data:

```text
D = A XOR B XOR C XOR P
```

Then regenerate Q using Reed–Solomon coding.

Recovery sequence:

```text
P → recover data
↓
regenerate Q
```

---

# 22. What is reconstruction?

Reconstruction means:

> Calculating missing information from the surviving information.

Example:

```text
Missing C
   ↓
P/Q recovery calculation
   ↓
Reconstructed C
```

Reconstruction can happen before replacement drives are installed.

---

# 23. What is rebuild?

Rebuild means:

> Restoring a failed member onto a replacement member.

Conceptually:

```text
Failed member
      ↓
Replacement drive
      ↓
Reconstruct missing blocks
      ↓
Write blocks to replacement
      ↓
Member restored
```

Therefore:

```text
Reconstruction → calculation
Rebuild        → calculation + permanent restoration
```

---

# 24. Can data be reconstructed before a replacement drive exists?

Yes.

Suppose:

```text
A    B    C❌    D❌    P    Q
```

A host may request data belonging to a missing member.

The RAID system can use the surviving information and P/Q
relationships to reconstruct the required data and return it to the
host.

The reconstructed result does not automatically become a permanent
copy on a replacement disk.

---

# 25. What happens after replacement drives are installed?

The RAID system can rebuild the failed members.

Conceptually:

```text
Failed member
      ↓
Replacement member
      ↓
Read surviving information
      ↓
Reconstruct missing blocks
      ↓
Write blocks
      ↓
Replacement rebuilt
```

With two failed members, both replacement members must be rebuilt
to restore full redundancy.

---

# 26. What is the rebuild window?

The rebuild window is the period during which the failed members
have not yet been fully restored.

During this period, the storage system may perform:

```text
Normal application I/O
+
Rebuild reads
+
Parity calculations
+
Replacement writes
```

Rebuild activity can affect application performance.

---

# 27. Why is RAID 6 rebuild protection better than RAID 5?

RAID 5 has one-member fault tolerance.

RAID 6 has two-member fault tolerance.

Therefore RAID 6 can survive a second member failure that would
exceed RAID 5's normal protection level.

However, RAID 6 is not unlimited fault tolerance.

---

# 28. What is the RAID 6 capacity formula?

For equal-sized members:

```text
Usable Capacity = (N - 2) × Member Size
```

Example:

```text
6 × 4 TB
```

Usable:

```text
(6 - 2) × 4 TB
= 16 TB
```

Two member-equivalents are consumed by parity.

---

# 29. What is the minimum number of drives for RAID 6?

The minimum is:

```text
4 drives
```

Conceptually:

```text
Data
Data
P
Q
```

---

# 30. What is a hot spare in RAID 6?

A hot spare is a standby drive that is not normally part of the
active RAID data set.

After a failure:

```text
Member failure
      ↓
Hot spare
      ↓
Rebuild
```

A hot spare can reduce the time before recovery begins.

It does **not** increase RAID 6's normal fault tolerance from two
members to three.

---

# 31. What is a URE?

URE means:

```text
Unrecoverable Read Error
```

Rebuild requires reads from surviving storage.

Therefore a URE on required surviving information can interfere
with reconstruction of the affected stripe.

RAID 6 provides stronger protection than RAID 5, but UREs still
matter during rebuild.

---

# 32. What is the difference between RAID 5 and RAID 6 during two failures?

### RAID 5

```text
First member failure
→ degraded
→ rebuild

Second member failure
→ exceeds normal protection
```

### RAID 6

```text
First member failure
→ degraded

Second member failure
→ still within normal protection

P + Q
→ recover both missing members
```

---

# 33. Interview Scenario

### Question

Two drives fail in a RAID 6 array. Can the array still provide data?

### Answer

Yes, RAID 6 is designed to tolerate two member failures. The RAID
system uses the surviving data together with the independent P and
Q parity relationships to reconstruct missing information.

---

# 34. Interview Scenario

### Question

Why can't RAID 5 do the same thing?

### Answer

RAID 5 has only one parity relationship. With two unknown failed
members, one parity relationship does not provide enough independent
information to solve for both missing values.

RAID 6 adds the independent Q parity relationship.

---

# 35. Interview Scenario

### Question

Why isn't Q simply another copy of P?

### Answer

Because a second identical parity relationship would not provide
new independent information.

P uses XOR parity.

Q uses Reed–Solomon coding over a Galois Field, producing a
mathematically different relationship.

---

# 36. Interview Scenario

### Question

Why is RAID 6 slower for writes than RAID 5?

### Answer

RAID 6 must maintain two parity values instead of one.

For writes, the RAID implementation must maintain:

```text
New Data
New P
New Q
```

Therefore RAID 6 has additional parity-maintenance work.

---

# 37. Interview Scenario

### Question

What is the difference between reconstruction and rebuild?

### Answer

Reconstruction means calculating missing information.

Rebuild means restoring the failed RAID member onto a replacement
member.

Reconstruction can happen without a replacement drive, while rebuild
requires a replacement target.

---

# 38. Interview Scenario

### Question

What happens if P fails but all data remains available?

### Answer

Data is not lost. P can be regenerated using:

```text
P = A XOR B XOR C XOR D
```

---

# 39. Interview Scenario

### Question

What happens if Q fails but all data remains available?

### Answer

Q can be regenerated using the Reed–Solomon calculation over the
Galois Field.

---

# 40. Interview Scenario

### Question

What happens if one data member and P fail?

### Answer

Use the surviving Q parity relationship to reconstruct the missing
data member. Once all data is available, regenerate P using XOR.

---

# 41. Interview Scenario

### Question

What happens if one data member and Q fail?

### Answer

Use P parity to reconstruct the missing data member. Once all data
is available, regenerate Q using the Reed–Solomon calculation.

---

# 42. Interview Scenario

### Question

Does a hot spare give RAID 6 three-disk fault tolerance?

### Answer

No.

A hot spare is a standby recovery resource. RAID 6's normal
fault-tolerance level remains two member failures.

---

# 43. Interview Scenario

### Question

What happens during a RAID 6 rebuild?

### Answer

The system reads surviving information, reconstructs missing blocks
using the available parity relationships, and writes the reconstructed
blocks to the replacement members.

The surviving members experience additional I/O and computation
during the rebuild.

---

# 44. Quick Interview Revision

```text
RAID 6
→ Block-level striping + dual parity

P
→ XOR parity

Q
→ Reed–Solomon parity

GF(2^8)
→ Galois Field with 256 elements

g
→ predefined generator used by the Q calculation

1 failed member
→ P can normally reconstruct missing data

2 failed members
→ P + Q provide two independent relationships

P failed
→ regenerate P from data

Q failed
→ regenerate Q from data

Reconstruction
→ calculate missing information

Rebuild
→ restore missing member onto replacement

Hot spare
→ standby recovery resource

Capacity
→ (N - 2) × member size

Minimum members
→ 4

Two-failure tolerance
→ main RAID 6 advantage
```

## 45. Strong Interview Answer — "Explain RAID 6"

> **RAID 6 is a block-level striped RAID level that uses dual
> distributed parity. The first parity, P, is XOR-based, while the
> second parity, Q, uses Reed–Solomon coding over a Galois Field.
> Because P and Q are mathematically independent, RAID 6 can normally
> tolerate two member failures. For one missing data member, P is
> normally sufficient for reconstruction. For two missing members,
> the controller uses both P and Q relationships to solve for the
> missing data. After replacement members are installed, the missing
> members are rebuilt and full redundancy is restored.**

---

# 46. RAID 6 Interview Preparation Checklist

```text
[ ] Explain why RAID 6 exists
[ ] Compare RAID 5 and RAID 6
[ ] Explain P parity
[ ] Explain Q parity
[ ] Explain Reed–Solomon
[ ] Explain GF(2^8)
[ ] Explain generator g
[ ] Explain GF multiplication
[ ] Explain field reduction
[ ] Calculate P
[ ] Calculate Q
[ ] Explain one-member recovery
[ ] Explain two-member recovery
[ ] Explain parity-only failures
[ ] Explain mixed data/parity failures
[ ] Explain reconstruction vs rebuild
[ ] Explain rebuild window
[ ] Explain hot spare
[ ] Explain URE impact
[ ] Calculate usable capacity
[ ] Explain RAID 6 write overhead
[ ] Explain full-stripe writes
```

