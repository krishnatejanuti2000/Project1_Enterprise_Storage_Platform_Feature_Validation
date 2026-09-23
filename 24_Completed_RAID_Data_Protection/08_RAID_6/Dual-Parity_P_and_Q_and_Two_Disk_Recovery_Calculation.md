# RAID 6 — P/Q Parity Calculation and Two-Disk Recovery

## 1. RAID 6 Parity Overview

RAID 6 uses two independent parity values:

```text
P → XOR parity
Q → Reed–Solomon parity
```

Q parity is generated using Reed–Solomon coding, which internally
uses Galois Field arithmetic.

For the examples in this note, we use:

```text
GF(2^8)
```

which contains 256 possible 8-bit field elements.

The RAID-6 Reed–Solomon calculation uses:

```text
g = 0x02
```

In 8-bit binary:

```text
g = 00000010
```

---

# PART 1 — RAID 6 P + Q PARITY GENERATION

## 2. RAID 6 Stripe

Assume a 6-drive RAID 6 stripe:

```text
Drive 1 → A
Drive 2 → B
Drive 3 → C
Drive 4 → D
Drive 5 → P
Drive 6 → Q
```

Data:

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
```

---

# 3. Generator `g`

The generator is:

```text
g = 0x02
```

Convert hexadecimal to 8-bit binary:

```text
0x02 = 00000010
```

Therefore:

```text
g = 00000010
```

---

# 4. Calculate Powers of `g`

## 4.1 Calculate g^0

```text
g^0 = 1
```

Therefore:

```text
g^0 = 00000001
```

---

## 4.2 Calculate g^1

```text
g^1 = g
```

Therefore:

```text
g^1 = 00000010
```

---

## 4.3 Calculate g^2

```text
g^2 = g × g

    = 00000010 × 00000010
```

Convert:

```text
00000010 = x
```

Therefore:

```text
x × x = x^2
```

Convert `x^2` to 8-bit binary:

```text
x^2 = 00000100
```

Therefore:

```text
g^2 = 00000100
```

---

## 4.4 Calculate g^3

```text
g^3 = g × g × g

    = 00000010 × 00000010 × 00000010
```

Convert:

```text
00000010 = x
```

Therefore:

```text
x × x × x
= x^3
```

Convert `x^3` to 8-bit binary:

```text
x^3 = 00001000
```

Therefore:

```text
g^3 = 00001000
```

---

## 4.5 Final coefficients

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

---

# 5. Q Parity Formula

Q parity is calculated as:

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

Substitute the values:

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

---

# 6. Calculate A × g^0

```text
10110110 × 00000001
```

Polynomial representation:

```text
10110110
= x^7 + x^5 + x^4 + x^2 + x
```

And:

```text
00000001 = x^0 = 1
```

Therefore:

```text
(x^7 + x^5 + x^4 + x^2 + x) × 1
```

Result:

```text
x^7 + x^5 + x^4 + x^2 + x
```

Convert back to binary:

```text
A × g^0 = 10110110
```

---

# 7. Calculate B × g^1

```text
01011001 × 00000010
```

Polynomial representation:

```text
01011001
= x^6 + x^4 + x^3 + 1
```

And:

```text
00000010 = x
```

Therefore:

```text
(x^6 + x^4 + x^3 + 1) × x
```

Multiply each term:

```text
x^6 × x = x^7
x^4 × x = x^5
x^3 × x = x^4
1   × x = x
```

Therefore:

```text
x^7 + x^5 + x^4 + x
```

Convert back to binary:

```text
B × g^1 = 10110010
```

---

# 8. Calculate C × g^2

```text
11000111 × 00000100
```

Polynomial representation:

```text
11000111
= x^7 + x^6 + x^2 + x + 1
```

And:

```text
00000100 = x^2
```

Therefore:

```text
(x^7 + x^6 + x^2 + x + 1) × x^2
```

Multiply each term:

```text
x^7 × x^2 = x^9
x^6 × x^2 = x^8
x^2 × x^2 = x^4
x   × x^2 = x^3
1   × x^2 = x^2
```

Therefore:

```text
x^9 + x^8 + x^4 + x^3 + x^2
```

---

## 8.1 Field Reduction

The GF(2^8) field used in this calculation is defined using:

```text
x^8 + x^4 + x^3 + x^2 + 1
```

Therefore:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

Multiply both sides by `x`:

```text
x^9 = x^5 + x^4 + x^3 + x
```

Now substitute into:

```text
x^9 + x^8 + x^4 + x^3 + x^2
```

Substitute `x^9`:

```text
(x^5 + x^4 + x^3 + x)
+ x^8
+ x^4 + x^3 + x^2
```

Now substitute `x^8`:

```text
(x^5 + x^4 + x^3 + x)
+ (x^4 + x^3 + x^2 + 1)
+ x^4 + x^3 + x^2
```

Collect equal terms:

```text
x^5
+ x^4 + x^4 + x^4
+ x^3 + x^3 + x^3
+ x^2 + x^2
+ x
+ 1
```

Inside GF(2), addition is XOR:

```text
x^4 + x^4 = 0
x^3 + x^3 = 0
x^2 + x^2 = 0
```

Therefore:

```text
x^5 + x^4 + x^3 + x + 1
```

Convert to 8-bit binary:

```text
x^7 x^6 x^5 x^4 x^3 x^2 x^1 x^0
 0   0   1   1   1   0   1   1
```

Therefore:

```text
C × g^2 = 00111011
```

---

# 9. Calculate D × g^3

```text
00101101 × 00001000
```

Polynomial representation:

```text
00101101
= x^5 + x^3 + x^2 + 1
```

And:

```text
00001000 = x^3
```

Therefore:

```text
(x^5 + x^3 + x^2 + 1) × x^3
```

Multiply each term:

```text
x^5 × x^3 = x^8
x^3 × x^3 = x^6
x^2 × x^3 = x^5
1   × x^3 = x^3
```

Therefore:

```text
x^8 + x^6 + x^5 + x^3
```

Reduce `x^8`:

```text
x^8 = x^4 + x^3 + x^2 + 1
```

Substitute:

```text
(x^4 + x^3 + x^2 + 1)
+ x^6 + x^5 + x^3
```

The two `x^3` terms cancel:

```text
x^3 + x^3 = 0
```

Therefore:

```text
x^6 + x^5 + x^4 + x^2 + 1
```

Convert to binary:

```text
x^7 x^6 x^5 x^4 x^3 x^2 x^1 x^0
 0   1   1   1   0   1   0   1
```

Therefore:

```text
D × g^3 = 01110101
```

---

# 10. Calculate Q

We now have:

```text
A × g^0 = 10110110
B × g^1 = 10110010
C × g^2 = 00111011
D × g^3 = 01110101
```

Therefore:

```text
Q =
10110110
XOR 10110010
XOR 00111011
XOR 01110101
```

First:

```text
  10110110
XOR 10110010
-----------
  00000100
```

Then:

```text
  00000100
XOR 00111011
-----------
  00111111
```

Then:

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

# 11. Calculate P Parity

P parity is ordinary XOR parity:

```text
P = A XOR B XOR C XOR D
```

Substitute:

```text
P =
10110110
XOR 01011001
XOR 11000111
XOR 00101101
```

First:

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

# 12. Final Healthy RAID 6 Stripe

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101

P = 00000101
Q = 01001010
```

Complete stripe:

```text
A          B          C          D          P          Q
10110110   01011001   11000111   00101101   00000101   01001010
```

---

# PART 2 — TWO-DISK FAILURE RECOVERY

## 13. Simulate Two Failed Drives

Assume:

```text
A = FAILED
C = FAILED
```

The surviving information is:

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

# 14. Recover Using P

Original P equation:

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
= 00000101
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

Important:

```text
A and C are NOT recovered yet.
```

We only know their relationship.

---

# 15. Recover Using Q

Original Q equation:

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

We know:

```text
g^0 = 00000001
g^1 = 00000010
g^2 = 00000100
g^3 = 00001000
```

Previously calculated:

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

First:

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
A XOR (C × 00000100) = 10001101
```

---

# 16. Substitute the P Relationship

From P:

```text
A XOR C = 01110001
```

Therefore:

```text
A = C XOR 01110001
```

Substitute this into the Q equation:

```text
(C XOR 01110001)
XOR
(C × 00000100)
=
10001101
```

Remove the parentheses:

```text
C XOR 01110001 XOR (C × 00000100)
=
10001101
```

XOR both sides with `01110001`:

```text
C XOR (C × 00000100)
=
10001101 XOR 01110001
```

Calculate:

```text
  10001101
XOR 01110001
-----------
  11111100
```

Therefore:

```text
C XOR (C × 00000100) = 11111100
```

---

# 17. Factor C

Since:

```text
00000100 = x^2
```

we have:

```text
C XOR (C × x^2) = 11111100
```

Write `C` as:

```text
C × 1
```

Then:

```text
(C × 1) XOR (C × x^2)
=
11111100
```

Factor out `C`:

```text
C × (1 XOR x^2)
=
11111100
```

Convert:

```text
1     = 00000001
x^2   = 00000100
```

Therefore:

```text
00000001 XOR 00000100
=
00000101
```

So:

```text
C × 00000101 = 11111100
```

---

# 18. Find the GF Inverse

To isolate `C`:

```text
C =
11111100 × inverse(00000101)
```

For the GF(2^8) field used in this example:

```text
inverse(00000101) = 10100111
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

Therefore:

```text
Recovered C = 11000111
```

---

# 19. Recover A

From the P relationship:

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

Calculate:

```text
  01110001
XOR 11000111
-----------
  10110110
```

Therefore:

```text
Recovered A = 10110110
```

---

# 20. Final Recovery Result

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
Recovery Result = PASS
```

Both failed data blocks were reconstructed correctly.

---

# 21. Complete RAID 6 Recovery Flow

```text
Two members fail
      ↓
Two unknown data blocks
      ↓
P parity
      ↓
First equation
      ↓
Q parity
      ↓
Second independent equation
      ↓
Solve the two equations
      ↓
Recover C
      ↓
Recover A
      ↓
Both missing data blocks reconstructed
```

---

# 22. Final Mental Model

```text
RAID 6
│
├── P parity
│     └── XOR
│
└── Q parity
      └── Reed–Solomon
            └── GF(2^8)
                  └── g = 0x02
```

For normal parity generation:

```text
Data
 ↓
P calculation ─────────→ P
 ↓
Q calculation ─────────→ Q
```

For two-disk recovery:

```text
Two missing data blocks
        ↓
P → first equation
        +
Q → second independent equation
        ↓
Solve two unknowns
        ↓
Recover both blocks
```

---

# Final Values Used in Both Examples

```text
A = 10110110
B = 01011001
C = 11000111
D = 00101101
P = 00000101
Q = 01001010
```

This single note contains both the **full parity-generation calculation** and the **full two-disk recovery calculation** we performed.

