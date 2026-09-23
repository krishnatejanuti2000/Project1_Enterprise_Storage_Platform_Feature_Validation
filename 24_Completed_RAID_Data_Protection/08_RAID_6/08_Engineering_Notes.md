# RAID 6 — Engineering Notes

## 1. Engineering Purpose of RAID 6

RAID 6 is designed to provide protection against **two member
failures**.

The primary engineering problem it addresses is the risk of a
second member failure while an array is already degraded and
rebuilding.

```text
RAID 5
    ↓
One parity relationship
    ↓
One-member fault tolerance

RAID 6
    ↓
Two independent parity relationships
    ↓
Two-member fault tolerance
```

The additional protection comes from the second parity mechanism,
not from simply duplicating the first parity.

---

# 2. RAID 6 Dual-Parity Architecture

A conceptual RAID 6 stripe contains:

```text
A    B    C    D    P    Q
```

Where:

```text
A-D → Data
P   → XOR parity
Q   → Reed–Solomon parity
```

The important engineering distinction is:

```text
P → XOR-based redundancy

Q → Reed–Solomon-based redundancy
      ↓
   Galois Field arithmetic
```

P and Q are mathematically independent redundancy relationships.

---

# 3. Why Two Independent Parity Relationships Are Required

Assume two data members are missing:

```text
A    B    C❌    D❌    P    Q
```

There are two unknowns:

```text
C = ?
D = ?
```

P provides one relationship:

```text
C XOR D = known value
```

That is not sufficient to determine C and D independently.

Q provides a second independent relationship:

```text
C×g² XOR D×g³ = known value
```

Therefore:

```text
1st equation → P
2nd equation → Q

2 unknowns + 2 independent equations
                    ↓
              recover both
```

This is the mathematical foundation of RAID 6 two-member
fault tolerance.

---

# 4. P Parity Engineering View

P parity is the simpler of the two parity calculations.

For data blocks:

```text
A
B
C
D
```

P is:

```text
P = A XOR B XOR C XOR D
```

The important engineering point is that P provides one
independent redundancy relationship.

For one missing data block, P alone is sufficient.

Example:

```text
A    B    C❌    D    P
```

Then:

```text
C = A XOR B XOR D XOR P
```

Q does not need to participate in this basic single-data-block
reconstruction.

---

# 5. Q Parity Engineering View

Q is the second parity mechanism.

Q is generated using Reed–Solomon coding over a Galois Field.

Conceptually:

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

The different data positions use different powers of the
generator.

Therefore:

```text
A → g^0
B → g^1
C → g^2
D → g^3
```

This creates a parity relationship that is mathematically
different from P.

---

# 6. GF(2^8)

The Q calculation operates in a finite field.

```text
GF(2^8)
```

represents a field with:

```text
2^8 = 256
```

possible values.

Each field element can be represented as an 8-bit value.

For example:

```text
10110110
```

can be represented as:

```text
x^7 + x^5 + x^4 + x^2 + x
```

The bit position corresponds to the polynomial exponent.

---

# 7. Galois Field Addition

Within the binary field:

```text
addition = XOR
```

Example:

```text
1011 XOR 0101 = 1110
```

There is no normal binary carry operation.

This XOR property is fundamental to both ordinary P parity and
the polynomial arithmetic used by Q.

---

# 8. Generator `g`

For the Linux RAID-6 field representation used in our
mathematical example:

```text
g = 0x02
```

In 8-bit binary:

```text
g = 00000010
```

`g` is a predefined generator element used by the Reed–Solomon
coding scheme.

It is not discovered by reading disk contents.

The RAID implementation already knows the field representation
and the generator.

---

# 9. Powers of the Generator

The Q calculation uses powers of `g`.

For the initial values:

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

For example:

```text
g^2 = g × g

    = x × x

    = x^2

    = 00000100
```

and:

```text
g^3 = g × g × g

    = x × x × x

    = x^3

    = 00001000
```

For higher powers, Galois Field multiplication and reduction become
important.

---

# 10. Galois Field Multiplication

GF multiplication is not ordinary integer multiplication.

An 8-bit value is treated as a polynomial over GF(2).

For example:

```text
00001011
```

represents:

```text
x^3 + x + 1
```

Multiplication is performed as polynomial multiplication followed
by reduction back into the field.

For multiplication by:

```text
g = 0x02 = x
```

a left shift corresponds to multiplying the polynomial by `x`,
provided field reduction is not required.

---

# 11. Field Reduction

Multiplication can generate:

```text
x^8
x^9
x^10
...
```

but a GF(2^8) field element must ultimately be represented using
powers up to:

```text
x^7
```

The field definition provides the reduction rule.

For the field representation used in our example:

```text
x^8 + x^4 + x^3 + x^2 + 1 = 0
```

Therefore:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

Multiplying by `x` gives:

```text
x^9 = x^5 + x^4 + x^3 + x
```

These relationships allow higher powers to be reduced back to
valid 8-bit field elements.

---

# 12. Worked Q-Parity Result

For:

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
```

we calculated:

```text
A × g^0 = 10110110
B × g^1 = 10110010
C × g^2 = 00111011
D × g^3 = 01110101
```

Then:

```text
Q =
10110110
XOR 10110010
XOR 00111011
XOR 01110101
```

Result:

```text
Q = 01001010
```

P for the same data was:

```text
P = 00000101
```

Therefore the complete parity state was:

```text
P = 00000101
Q = 01001010
```

---

# 13. Two-Data-Member Recovery

Consider:

```text
A❌    B    C❌    D    P    Q
```

The missing values are:

```text
A = ?
C = ?
```

Using P:

```text
A XOR C = known value
```

Using Q:

```text
A XOR (C×g^2) = known value
```

Now the system has two independent equations for two unknowns.

The recovery process can solve the two equations and reconstruct
both missing values.

---

# 14. Reconstruction vs Rebuild

These terms must remain separate.

## Reconstruction

```text
Missing information
      ↓
Calculate missing information
      ↓
Use reconstructed result
```

The reconstructed result may be used to satisfy an application
read even when replacement drives do not yet exist.

## Rebuild

```text
Failed member
      ↓
Replacement member available
      ↓
Reconstruct missing blocks
      ↓
Write blocks to replacement
      ↓
Member restored
```

Therefore:

```text
Reconstruction
→ calculation

Rebuild
→ calculation + permanent restoration
```

---

# 15. Degraded Read

## One missing data member

Example:

```text
A    B    C❌    D    P    Q
```

P is sufficient:

```text
C = A XOR B XOR D XOR P
```

The reconstructed data can be returned to the host.

No replacement drive is required for this on-demand reconstruction.

---

# 16. Two Missing Data Members

Example:

```text
A    B    C❌    D❌    P    Q
```

P alone cannot determine both missing members.

Q adds the second independent relationship.

Conceptually:

```text
P
↓
First equation

Q
↓
Second equation

Both
↓
Recover both missing members
```

This is the central functional advantage of RAID 6 over RAID 5.

---

# 17. Parity-Member Failure

## P failure

```text
A    B    C    D    P❌    Q
```

Data remains intact.

Regenerate:

```text
P = A XOR B XOR C XOR D
```

## Q failure

```text
A    B    C    D    P    Q❌
```

Data remains intact.

Regenerate Q using the Reed–Solomon calculation.

---

# 18. Mixed Failure Scenarios

## Data + P Failure

```text
A    B    C❌    D    P❌    Q
```

Use Q to reconstruct the missing data:

```text
Q
↓
Recover C
```

Then:

```text
A B C D
   ↓
XOR
   ↓
Regenerate P
```

## Data + Q Failure

```text
A    B    C    D❌    P    Q❌
```

Use P:

```text
P
↓
Recover D
```

Then:

```text
A B C D
   ↓
Reed–Solomon
   ↓
Regenerate Q
```

---

# 19. Two-Member Rebuild

If two RAID 6 members have failed:

```text
A    B    C❌    D❌    P    Q
```

and replacement members become available:

```text
A    B    C'    D'    P    Q
```

the RAID system rebuilds the replacement members.

Conceptually:

```text
Surviving information
        ↓
Reconstruction
        ↓
Write missing blocks
        ↓
Replacement C'
        +
Replacement D'
```

Until rebuild completes, redundancy has not yet been fully restored.

---

# 20. Rebuild Window

The rebuild window is the time between member failure and full
redundancy restoration.

During this period:

```text
Normal application I/O
+
Rebuild reads
+
Parity calculations
+
Replacement writes
```

may occur simultaneously.

Therefore rebuild can affect application performance.

---

# 21. RAID 6 Performance Considerations

RAID 6 generally has higher parity-maintenance cost than RAID 5.

The write path must maintain:

```text
New Data
New P
New Q
```

Small partial writes can therefore incur additional work.

Full-stripe writes are generally more efficient because the new data
for the stripe is already available for calculating both parity
values.

---

# 22. Capacity Considerations

For equal-sized members:

```text
RAID 6 usable capacity
=
(N - 2) × member size
```

Two member-equivalents are consumed by parity.

Example:

```text
6 × 4 TB
```

gives:

```text
24 TB raw

16 TB usable
```

ignoring filesystem and other implementation overhead.

---

# 23. Minimum Members

Minimum RAID 6 members:

```text
4
```

Conceptually:

```text
Data
Data
P
Q
```

Larger arrays can provide more usable capacity while retaining the
two-member fault-tolerance property.

---

# 24. Hot Spare Engineering View

A hot spare is a standby disk that can be used as a recovery target.

```text
Active RAID members
       +
Hot Spare
```

After failure:

```text
Failure
   ↓
Hot Spare
   ↓
Rebuild
```

The hot spare is a recovery resource.

It does not increase RAID 6's fundamental fault tolerance from:

```text
2 failures
```

to:

```text
3 failures
```

---

# 25. URE Engineering View

URE means:

```text
Unrecoverable Read Error
```

Rebuilds require reads from surviving members.

Therefore:

```text
Rebuild
   ↓
Read surviving information
   ↓
Reconstruct missing information
```

An unreadable required block can interfere with reconstruction of
the affected stripe.

RAID 6 provides more redundancy than RAID 5, but it does not make
the surviving storage media immune to read errors.

---

# 26. Engineering Mental Model

```text
                     RAID 6
                        │
              ┌─────────┴─────────┐
              │                   │
           P parity            Q parity
              │                   │
             XOR           Reed–Solomon
                                  │
                            Galois Field
                                  │
                              GF(2^8)
```

Fault model:

```text
0 failures
    ↓
Healthy

1 failure
    ↓
Degraded
    ↓
P can normally reconstruct one missing data block

2 failures
    ↓
P + Q
    ↓
Two independent relationships
    ↓
Two-member recovery possible
```

Restoration:

```text
Failure
   ↓
Degraded
   ↓
Replacement
   ↓
Rebuild
   ↓
Redundancy restored
```

---

# 27. Key Engineering Takeaways

1. RAID 6 adds a second independent parity relationship to RAID 5.
2. P is XOR-based parity.
3. Q is based on Reed–Solomon coding.
4. Reed–Solomon coding uses Galois Field arithmetic.
5. GF(2^8) provides the field for the byte-oriented calculation.
6. The generator is a predefined mathematical parameter.
7. P alone handles one missing data block.
8. P + Q provide the relationships required to recover two missing
   members.
9. Reconstruction and rebuild are different operations.
10. Reconstruction can occur before replacement drives are present.
11. Rebuild permanently restores missing members onto replacements.
12. Parity-only failures do not directly destroy data.
13. Mixed data/parity failures can be handled by the surviving
    parity mechanism followed by parity regeneration.
14. RAID 6 sacrifices two member-equivalents of usable capacity for
    dual parity.
15. RAID 6 has higher parity-maintenance overhead than RAID 5.
16. The rebuild window remains an important operational period.
17. Hot spares accelerate recovery but do not add another fault-
    tolerance level.
18. UREs remain relevant during rebuild.

```
