# RAID 6

## 1. Overview

RAID 6 is a fault-tolerant RAID level that uses:

- Block-level striping
- Distributed dual parity
- P parity
- Q parity
- Two-member fault tolerance

The major architectural difference between RAID 5 and RAID 6 is
that RAID 6 maintains **two independent parity calculations**
instead of one.

```text
RAID 5
    ↓
One parity
    ↓
One-member fault tolerance

RAID 6
    ↓
Two independent parity values
    ↓
Two-member fault tolerance
```

The Linux RAID-6 design defines the two syndromes as **P and Q**.
P is the ordinary XOR parity, while Q is the Reed–Solomon code. ([Kernel.org][1])

---

# 2. Why RAID 6 Was Introduced

RAID 5 can normally tolerate one failed member.

```text
RAID 5

Disk 1  → Healthy
Disk 2  → Healthy
Disk 3  → Failed
Disk 4  → Healthy
Disk 5  → Healthy
```

The RAID 5 array becomes degraded but can reconstruct the missing
information.

The problem occurs when another member fails before the first
failure has been fully rebuilt:

```text
Disk 1  → Healthy
Disk 2  → Failed
Disk 3  → Failed
Disk 4  → Healthy
Disk 5  → Healthy
```

RAID 5 has only one parity relationship and therefore cannot
normally reconstruct two missing members.

RAID 6 addresses this limitation by maintaining **two independent
parity relationships**.

Therefore:

```text
RAID 5 → one-member fault tolerance
RAID 6 → two-member fault tolerance
```

---

# 3. RAID 6 Architecture

A conceptual RAID 6 stripe can be represented as:

```text
A    B    C    D    P    Q
│    │    │    │    │    │
Data Data Data Data P    Q
```

Where:

```text
A, B, C, D → Data
P           → First parity
Q           → Second parity
```

The two parity mechanisms are fundamentally different:

```text
P
↓
XOR parity


Q
↓
Reed–Solomon coding
↓
Galois Field arithmetic
```

Linux RAID-6 documentation describes P as the ordinary XOR parity
and Q as the Reed–Solomon code. ([Kernel.org][1])

---

# 4. P Parity

P parity is the XOR-based parity calculation.

For four data blocks:

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

Calculate:

```text
  10110110
XOR 01011001
-----------
  11101111
```

Then:

```text
  11101111
XOR 11000111
-----------
  00101000
```

Then:

```text
  00101000
XOR 00101101
-----------
  00000101
```

Therefore:

```text
P = 00000101
```

---

# 5. Q Parity

Q is the second independent parity calculation.

Q uses:

> **Reed–Solomon coding over a Galois Field.**

The Linux RAID-6 mathematical specification represents Q as:

```text
Q =
g^0 × D0
XOR
g^1 × D1
XOR
g^2 × D2
XOR
...
XOR
g^(n-1) × D(n-1)
```

where `g` is a generator of the field. Linux uses:

```text
g = 0x02
```

for its RAID-6 field representation. ([Kernel.org][1])

---

# 6. Galois Field GF(2^8)

Q parity uses Galois Field arithmetic.

```text
GF
↓
Galois Field
```

For the RAID-6 calculation discussed here:

```text
GF(2^8)
```

means:

```text
2^8 = 256
```

possible field elements.

The elements are represented as 8-bit values:

```text
00000000
00000001
00000010
...
11111111
```

The Linux RAID-6 implementation operates on bytes and performs the
field calculations on the corresponding bytes of the data blocks. ([Kernel.org][1])

---

# 7. Galois Field Addition

Inside GF(2), addition is XOR.

Example:

```text
  1011
XOR 0101
-----------
  1110
```

Therefore:

```text
GF addition = XOR
```

This is why P parity can be described as ordinary XOR parity.

---

# 8. Generator `g`

For the Linux RAID-6 field representation:

```text
g = 0x02
```

`0x02` is hexadecimal.

In 8-bit binary:

```text
g = 00000010
```

The generator is part of the mathematical definition used by the
RAID-6 Reed–Solomon calculation. It is not discovered by reading
the disks. The Linux RAID-6 specification identifies `{02}` as a
generator of its field representation. ([Kernel.org][1])

---

# 9. Powers of `g`

The Q calculation uses powers of the generator.

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

For example:

```text
g^2 = g × g

    = 00000010 × 00000010

00000010 = x

x × x = x^2

x^2 = 00000100
```

Similarly:

```text
g^3 = g × g × g

    = x × x × x

    = x^3

    = 00001000
```

The actual multiplication is Galois Field multiplication, not
ordinary integer multiplication.

---

# 10. Polynomial Representation

An 8-bit GF(2^8) value can be represented as a polynomial.

Example:

```text
10110110
```

Bit positions:

```text
7 6 5 4 3 2 1 0
1 0 1 1 0 1 1 0
```

Therefore:

```text
10110110
=
x^7 + x^5 + x^4 + x^2 + x
```

The bit position determines the power of `x`.

---

# 11. Galois Field Multiplication

GF multiplication is not normal binary multiplication.

The binary value is interpreted as a polynomial over GF(2).

For example:

```text
00000010 = x
```

Therefore:

```text
g^2
=
00000010 × 00000010

=
x × x

=
x^2

=
00000100
```

When multiplication produces a power of `x` equal to or greater
than `x^8`, field reduction is required.

---

# 12. Field Reduction

The GF(2^8) field representation used in this calculation is based
on the primitive polynomial:

```text
x^8 + x^4 + x^3 + x^2 + 1
```

This is represented as:

```text
0x11D
```

The polynomial provides the reduction rule needed when a result
contains powers greater than `x^7`.

From:

```text
x^8 + x^4 + x^3 + x^2 + 1 = 0
```

the reduction relationship is:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

Because arithmetic is over GF(2), addition/subtraction behaves as
XOR.

Multiplying the relationship by `x` gives:

```text
x^9 = x^5 + x^4 + x^3 + x
```

These reduction relationships allow higher powers to be converted
back to valid 8-bit field elements.

The Linux RAID-6 specification defines the field representation
and its generator accordingly. ([Kernel.org][1])

---

# 13. Complete Q Parity Calculation

Consider:

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
```

The Q formula is:

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

The generator powers are:

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

Therefore:

```text
Q =
(10110110 × 00000001)
XOR
(01011001 × 00000010)
XOR
(11000111 × 00000100)
XOR
(00101101 × 00001000)
```

### A × g^0

```text
10110110 × 00000001
=
10110110
```

Therefore:

```text
A × g^0 = 10110110
```

### B × g^1

```text
01011001 × 00000010
```

Polynomial form:

```text
01011001
=
x^6 + x^4 + x^3 + 1
```

Since:

```text
00000010 = x
```

we get:

```text
(x^6 + x^4 + x^3 + 1) × x

=
x^7 + x^5 + x^4 + x
```

Therefore:

```text
B × g^1 = 10110010
```

### C × g^2

```text
11000111 × 00000100
```

Polynomial form:

```text
11000111
=
x^7 + x^6 + x^2 + x + 1
```

Since:

```text
00000100 = x^2
```

we get:

```text
(x^7 + x^6 + x^2 + x + 1) × x^2

=
x^9 + x^8 + x^4 + x^3 + x^2
```

Reduce the higher powers:

```text
x^8 = x^4 + x^3 + x^2 + 1

x^9 = x^5 + x^4 + x^3 + x
```

Therefore:

```text
x^9 + x^8 + x^4 + x^3 + x^2

=
(x^5 + x^4 + x^3 + x)
+
(x^4 + x^3 + x^2 + 1)
+
x^4 + x^3 + x^2
```

Using XOR cancellation:

```text
x^4 + x^4 = 0
x^3 + x^3 = 0
x^2 + x^2 = 0
```

Therefore:

```text
=
x^5 + x^4 + x^3 + x + 1
```

So:

```text
C × g^2 = 00111011
```

### D × g^3

```text
00101101 × 00001000
```

Polynomial form:

```text
00101101
=
x^5 + x^3 + x^2 + 1
```

Since:

```text
00001000 = x^3
```

we get:

```text
(x^5 + x^3 + x^2 + 1) × x^3

=
x^8 + x^6 + x^5 + x^3
```

Reduce `x^8`:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

Therefore:

```text
=
(x^4 + x^3 + x^2 + 1)
+
x^6 + x^5 + x^3
```

The `x^3` terms cancel:

```text
=
x^6 + x^5 + x^4 + x^2 + 1
```

Therefore:

```text
D × g^3 = 01110101
```

### XOR the four Q terms

```text
Q =
10110110
XOR 10110010
XOR 00111011
XOR 01110101
```

Step 1:

```text
10110110
XOR 10110010
-----------
00000100
```

Step 2:

```text
00000100
XOR 00111011
-----------
00111111
```

Step 3:

```text
00111111
XOR 01110101
-----------
01001010
```

Therefore:

```text
Q = 01001010
```

---

# 14. Final RAID 6 Stripe

The complete stripe is:

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
P = 00000101
Q = 01001010
```

Conceptually:

```text
A          B          C          D          P          Q
10110110   01011001   11000111   00101101   00000101   01001010
```

---

# 15. Two-Disk Failure

Assume two data members fail:

```text
A = FAILED
C = FAILED
```

Surviving information:

```text
A = ?
B = 01011001
C = ?
D = 00101101
P = 00000101
Q = 01001010
```

Unknowns:

```text
A = ?
C = ?
```

---

# 16. First Recovery Equation — P

Original:

```text
P = A XOR B XOR C XOR D
```

Therefore:

```text
A XOR C = P XOR B XOR D
```

Substitute:

```text
A XOR C
=
00000101
XOR 01011001
XOR 00101101
```

Calculate:

```text
00000101
XOR 01011001
-----------
01011100
```

Then:

```text
01011100
XOR 00101101
-----------
01110001
```

Therefore:

```text
A XOR C = 01110001
```

This is a relationship between the two missing blocks.
It does not independently determine A and C.

---

# 17. Second Recovery Equation — Q

Original:

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

Known:

```text
B × g^1 = 10110010
D × g^3 = 01110101
```

Therefore:

```text
01001010
=
A
XOR 10110010
XOR (C × 00000100)
XOR 01110101
```

Remove the known contributions:

```text
A XOR (C × 00000100)
=
01001010
XOR 10110010
XOR 01110101
```

Calculate:

```text
01001010
XOR 10110010
-----------
11111000
```

Then:

```text
11111000
XOR 01110101
-----------
10001101
```

Therefore:

```text
A XOR (C × 00000100)
=
10001101
```

---

# 18. Combine the Two Equations

From P:

```text
A XOR C = 01110001
```

Therefore:

```text
A = C XOR 01110001
```

Substitute into the Q equation:

```text
(C XOR 01110001)
XOR
(C × 00000100)
=
10001101
```

Therefore:

```text
C XOR (C × 00000100)
=
11111100
```

Since:

```text
00000100 = x^2
```

we have:

```text
C XOR (C × x^2)
=
11111100
```

Factor:

```text
C × 1 XOR C × x^2
=
11111100
```

Therefore:

```text
C × (1 XOR x^2)
=
11111100
```

Now:

```text
1       = 00000001
x^2     = 00000100
```

So:

```text
00000001
XOR 00000100
-----------
00000101
```

Therefore:

```text
C × 00000101
=
11111100
```

---

# 19. Recover C Using the GF Inverse

To isolate C:

```text
C =
11111100 × inverse(00000101)
```

For this GF(2^8) field:

```text
inverse(00000101)
=
10100111
```

because:

```text
00000101 × 10100111
=
00000001
```

Therefore:

```text
C =
11111100 × 10100111
```

GF multiplication gives:

```text
C = 11000111
```

So:

```text
Recovered C = 11000111
```

---

# 20. Recover A

From the P equation:

```text
A XOR C = 01110001
```

Therefore:

```text
A = 01110001 XOR C
```

Substitute C:

```text
A =
01110001
XOR 11000111
```

Result:

```text
10110110
```

Therefore:

```text
Recovered A = 10110110
```

---

# 21. Verify Recovery

Original:

```text
A = 10110110
C = 11000111
```

Recovered:

```text
A = 10110110
C = 11000111
```

Therefore:

```text
Two-disk recovery = PASS
```

---

# 22. Reconstruction vs Rebuild

These are different operations.

## Reconstruction

Reconstruction means:

> Calculate missing information from surviving information.

Example:

```text
Missing data
    ↓
P / Q calculations
    ↓
Reconstructed data
```

Reconstruction can occur while replacement drives are not yet
present. The reconstructed result can be used to satisfy a host
read request.

## Rebuild

Rebuild means:

> Restore the missing member onto a replacement drive.

Example:

```text
Failed member
      ↓
Replacement drive
      ↓
Reconstruct missing blocks
      ↓
Write reconstructed blocks
      ↓
Replacement member restored
```

Therefore:

```text
Reconstruction
→ calculate missing information

Rebuild
→ calculate + permanently write missing information to replacement
```

---

# 23. Degraded Read

## One failed data member

Example:

```text
A    B    C❌    D    P    Q
```

For a single missing data member, P is sufficient for the basic
XOR reconstruction:

```text
C = A XOR B XOR D XOR P
```

Q is not required for this normal single-data reconstruction.

---

# 24. Two Failed Data Members

Example:

```text
A    B    C❌    D❌    P    Q
```

P alone provides only one relationship:

```text
C XOR D = known value
```

Q provides the second independent relationship:

```text
C×g^2 XOR D×g^3 = known value
```

Therefore:

```text
P equation
+
Q equation
↓
two independent equations
↓
recover two missing members
```

---

# 25. Two Failed Members Without Replacement

Even before replacement drives are installed, RAID 6 can reconstruct
missing information on demand for host reads.

Conceptually:

```text
Application
    ↓
RAID layer
    ↓
Read surviving members
    ↓
P + Q recovery calculation
    ↓
Temporary reconstructed data
    ↓
Return data to application
```

The reconstructed result does not automatically become a permanent
new copy on a replacement disk because no replacement disk exists.

---

# 26. Rebuild After Replacement

After replacement members are installed:

```text
Failed member
     ↓
Replacement member
     ↓
Read surviving information
     ↓
Calculate missing blocks
     ↓
Write reconstructed blocks
     ↓
Member rebuilt
```

For two failed members, both replacement members have to be rebuilt.

Until rebuild completes, the array remains in a degraded/recovery
state.

---

# 27. Failure Combinations

## 27.1 One Data + P Failure

Example:

```text
A    B    C❌    D    P❌    Q
```

Recovery:

```text
Use Q / Reed–Solomon
        ↓
Recover C
        ↓
A B C D available
        ↓
P = A XOR B XOR C XOR D
        ↓
Regenerate P
```

---

## 27.2 One Data + Q Failure

Example:

```text
A    B    C    D❌    P    Q❌
```

Recovery:

```text
Use P / XOR
        ↓
Recover D
        ↓
A B C D available
        ↓
Recalculate Q using Reed–Solomon
        ↓
Regenerate Q
```

---

# 28. Parity-Only Failure

## P failure

```text
A    B    C    D    P❌    Q
```

Data is unaffected.

Regenerate:

```text
P = A XOR B XOR C XOR D
```

## Q failure

```text
A    B    C    D    P    Q❌
```

Data is unaffected.

Regenerate Q using:

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

---

# 29. Capacity

For equal-sized members:

```text
Usable Capacity = (N - 2) × Member Size
```

Example:

```text
6 × 4 TB
```

Raw capacity:

```text
24 TB
```

Two disk-equivalents are used for parity:

```text
8 TB equivalent
```

Usable capacity:

```text
(6 - 2) × 4 TB
= 16 TB
```

Therefore:

```text
RAID 5 → (N - 1) × size
RAID 6 → (N - 2) × size
```

---

# 30. Minimum Number of Drives

RAID 6 requires at least:

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

Therefore:

```text
RAID 5 → minimum 3
RAID 6 → minimum 4
```

---

# 31. Write Behavior

RAID 6 must maintain both parity values.

For a partial write, the RAID implementation must update:

```text
Data
P
Q
```

Conceptually:

```text
Old data/parity
       ↓
Parity calculations
       ↓
New data
New P
New Q
```

Therefore RAID 6 has more parity-maintenance work than RAID 5.

The Linux kernel also exposes RAID-6 P+Q operations, with P as
XOR and Q as Reed–Solomon, and provides mechanisms that can be
accelerated by suitable hardware/DMA implementations. ([Kernel Documentation][2])

---

# 32. Full-Stripe Write

For a full-stripe write, the controller has all the new data.

It can calculate:

```text
New P
```

directly from the new data using XOR.

It can calculate:

```text
New Q
```

directly from the new data using the Reed–Solomon calculation.

Conceptually:

```text
New Data
   │
   ├──→ P calculation
   │
   └──→ Q calculation
             ↓
       New parity values
```

---

# 33. Rebuild Window

During rebuild:

```text
Normal application I/O
        +
Rebuild reads
        +
Parity calculations
        +
Replacement writes
```

The array remains in a recovery/reduced-redundancy state until the
failed members have been rebuilt.

With RAID 6, recovery from two failed members involves more
reconstruction work than a single-member RAID 5 rebuild.

The rebuild window is operationally important because the array has
not yet returned to its fully protected state.

---

# 34. Hot Spare

A hot spare is a standby disk that is not normally part of the
active RAID data set.

Example:

```text
Active RAID 6:
D1 D2 D3 D4 D5 D6

Hot Spare:
D7
```

After a member failure:

```text
Failure
   ↓
Hot spare
   ↓
Rebuild target
```

A hot spare can allow recovery to begin without waiting for manual
physical replacement, depending on the system configuration.

A hot spare does **not** increase RAID 6's normal fault tolerance
from two failed members to three.

---

# 35. Unequal Disk Sizes

With simple equal-sized RAID member regions, the usable member
region is constrained by the smallest member.

Example:

```text
Disk 1 = 4 TB
Disk 2 = 4 TB
Disk 3 = 6 TB
Disk 4 = 6 TB
Disk 5 = 6 TB
Disk 6 = 6 TB
```

The effective member size for that simple configuration is:

```text
4 TB
```

The additional capacity of the larger drives may therefore remain
unused by that RAID member configuration.

---

# 36. Chunk vs Stripe

These are different concepts.

## Chunk

A chunk is the amount of contiguous data assigned to one RAID
member before moving to the next member.

## Stripe

A stripe is the complete corresponding set of chunks across the
RAID members.

Therefore:

```text
Chunk
→ portion on one member

Stripe
→ complete RAID stripe
```

---

# 37. URE and Rebuild Considerations

URE means:

```text
Unrecoverable Read Error
```

During rebuild, the RAID system must read surviving information to
reconstruct missing blocks.

RAID 6 provides stronger redundancy than RAID 5 because it has two
parity syndromes, but rebuild still depends on successfully
obtaining the required surviving information.

Therefore:

```text
RAID 6
→ higher fault tolerance
→ stronger rebuild protection
→ not immune to all storage/read errors
```

---

# 38. Key RAID 6 Points

1. RAID 6 uses block-level striping.
2. RAID 6 uses dual distributed parity.
3. P parity is XOR-based.
4. Q parity uses Reed–Solomon coding.
5. Reed–Solomon coding uses Galois Field arithmetic.
6. The field used in this calculation is GF(2^8).
7. GF(2^8) contains 256 possible 8-bit elements.
8. Linux RAID-6 uses `g = 0x02` as a generator for its field
   representation. ([Kernel.org][1])
9. P provides one parity relationship.
10. Q provides a second independent parity relationship.
11. RAID 6 can normally tolerate two member failures.
12. One missing data member can normally be reconstructed using P.
13. Two missing data members require P + Q information.
14. A missing P parity member can be regenerated from data.
15. A missing Q parity member can be regenerated using the
    Reed–Solomon calculation.
16. Reconstruction means calculating missing information.
17. Rebuild means restoring missing member data onto a replacement.
18. A hot spare is a recovery resource.
19. A hot spare does not add another RAID fault-tolerance level.
20. RAID 6 has lower usable capacity than RAID 5 because it consumes
    two member-equivalents for parity.
21. RAID 6 requires at least four members.
22. RAID 6 has higher parity-maintenance overhead than RAID 5.
23. The rebuild window is an important operational period.
24. UREs remain relevant during rebuild and recovery.
25. Chunk size and stripe are different concepts.

---

# 39. Summary

The fundamental RAID 6 architecture is:

```text
                 RAID 6
                    │
          ┌─────────┴─────────┐
          │                   │
       P parity            Q parity
          │                   │
         XOR            Reed–Solomon
                              │
                        Galois Field
                              │
                           GF(2^8)
```

Fault tolerance:

```text
RAID 5
    ↓
1 member failure

RAID 6
    ↓
2 member failures
```

Two-disk recovery:

```text
Two missing members
        ↓
P relationship
        +
Q relationship
        ↓
Two independent equations
        ↓
Recover two missing members
```

Reconstruction:

```text
Calculate missing information
```

Rebuild:

```text
Calculate missing information
        ↓
Write to replacement member
```

RAID 6 therefore provides stronger fault tolerance than RAID 5 by
adding a second, mathematically independent parity mechanism.


