# Universal Metric Time (UMT) Specification

**Status:** Draft Specification
**Version:** 1.0

---

## 1. Abstract

**Universal Metric Time (UMT)** is a decimal time-of-day system that divides a nominal 24-hour day into decimal units.

A UMT day contains:

* **100 Chrona**
* **1,000 Quanta per Chrona**
* **1,000 Grana per Quanta**

The canonical UMT representation is:

```text
CCXQQQ.GGG
```

Example:

```text
42X317.084
```

UMT is based on the UTC day and is independent of local time zones.

The notation also supports a compact representation in which trailing decimal positions may be omitted:

```text
42X3
42X31
42X317
42X317.08
42X317.084
```

Omitted positions are interpreted as zero.

---

## 2. Normative Language

The terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirements.

---

## 3. Time Model

### 3.1 UMT Day

One UMT day corresponds to one nominal civil day of:

```text
86,400 SI seconds
```

The day is divided into exactly:

```text
100 Chrona
```

Therefore:

```text
1 Chrona = 864 seconds
          = 14 minutes 24 seconds
```

Each Chrona contains:

```text
1,000 Quanta
```

Therefore:

```text
1 Quanta = 0.864 seconds
```

Each Quanta contains:

```text
1,000 Grana
```

Therefore:

```text
1 Grana = 0.000864 seconds
        = 0.864 milliseconds
        = 864 microseconds
```

The unit relationships are therefore:

```text
1 day     = 100 Chrona
1 Chrona  = 1,000 Quanta
1 Quanta  = 1,000 Grana
```

or:

```text
1 day = 100,000 Quanta
      = 100,000,000 Grana
```

---

## 4. Units

### 4.1 Chrona

The **Chrona** is the largest UMT time-of-day component.

Range:

```text
0–99
```

Examples:

```text
0X
9X
42X
99X
```

In canonical form, Chrona MUST contain exactly two digits:

```text
00X
09X
42X
99X
```

Compact representations MAY omit a leading zero:

```text
9X
```

is equivalent to:

```text
09X
```

---

### 4.2 Quanta

The **Quanta** component expresses the decimal subdivision of a Chrona.

Range:

```text
000–999
```

Canonical form:

```text
X000
X001
X100
X999
```

In compact form, one or two trailing positions MAY be omitted.

Omitted positions are filled with zeros on the right.

Therefore:

```text
X1   = X100
X12  = X120
X123 = X123
```

Likewise:

```text
X01  = X010
X001 = X001
```

The position of a digit is therefore significant.

---

### 4.3 Grana

The **Grana** component is the smallest unit defined by the base UMT representation.

It follows a decimal point:

```text
.
```

Range:

```text
000–999
```

Canonical form:

```text
.000
.001
.100
.999
```

Compact form MAY omit trailing positions:

```text
.1   = .100
.12  = .120
.123 = .123
```

A Grana MUST NOT be described as exactly one millisecond.

Its exact duration is:

```text
0.864 ms
```

---

## 5. Syntax

### 5.1 Canonical Form

The canonical UMT representation is:

```text
CCXQQQ.GGG
```

where:

```text
CC  = Chrona, exactly 2 decimal digits
X   = literal uppercase character "X"
QQQ = Quanta, exactly 3 decimal digits
.   = literal decimal point
GGG = Grana, exactly 3 decimal digits
```

Example:

```text
42X317.084
```

Canonical values MUST satisfy:

```text
00 <= CC <= 99
000 <= QQQ <= 999
000 <= GGG <= 999
```

---

## 6. Compact Form

A human-readable compact representation MAY omit redundant trailing zero positions.

Examples:

```text
09X500.000 -> 9X5
12X300.000 -> 12X3
12X340.000 -> 12X34
12X345.000 -> 12X345
12X345.600 -> 12X345.6
12X345.670 -> 12X345.67
12X345.678 -> 12X345.678
```

A parser MUST reconstruct omitted positions by **right-padding with zeros**.

Thus:

```text
12X3
```

MUST be interpreted as:

```text
12X300.000
```

and:

```text
12X03
```

MUST be interpreted as:

```text
12X030.000
```

These are different times.

---

## 7. Value Semantics

Let:

```text
C = Chrona
Q = Quanta
G = Grana
```

The total number of Quanta elapsed since the start of the day is:

```text
U = (C × 1000) + Q + (G / 1000)
```

The number of SI seconds elapsed since the start of the day is:

```text
S = U × 0.864
```

Equivalently:

```text
S = (C × 864)
  + (Q × 0.864)
  + (G × 0.000864)
```

---

## 8. Day Fraction

UMT can also be interpreted directly as a decimal fraction of a day.

For:

```text
CCXQQQ.GGG
```

the fraction of the day elapsed is:

```text
F = U / 100000
```

where:

```text
U = (CC × 1000) + QQQ + (GGG / 1000)
```

Example:

```text
25X000.000
```

represents:

```text
25,000 / 100,000
= 0.25
```

of the day.

Therefore:

```text
25X000.000 = 06:00:00 UTC
50X000.000 = 12:00:00 UTC
75X000.000 = 18:00:00 UTC
```

---

## 9. UTC Relationship

UMT is defined relative to the UTC day.

A local civil time MUST first be converted to UTC before it is converted to UMT.

For example:

```text
Local Time
    ↓
UTC
    ↓
UMT
```

Time-zone offsets are therefore not part of the UMT clock notation itself.

---

## 10. Conversion from UTC to UMT

Given UTC time:

```text
hour
minute
second
fraction
```

calculate the number of elapsed SI seconds since UTC midnight:

```text
S =
    hour × 3600
  + minute × 60
  + second
  + fraction
```

Then calculate:

```text
U = S / 0.864
```

### 10.1 Chrona

```text
C = floor(U / 1000)
```

### 10.2 Quanta

```text
Q = floor(U) mod 1000
```

### 10.3 Grana

```text
G = floor((U - floor(U)) × 1000)
```

The default conversion therefore truncates values below one Grana.

---

## 11. Conversion from UMT to UTC

Given:

```text
CCXQQQ.GGG
```

calculate:

```text
U = (CC × 1000)
  + QQQ
  + (GGG / 1000)
```

Then:

```text
S = U × 0.864
```

`S` is the number of elapsed SI seconds since UTC midnight.

From this value:

```text
hours   = floor(S / 3600)

minutes = floor(
            (S mod 3600) / 60
          )

seconds = S mod 60
```

---

## 12. Conversion Examples

### 12.1 Midnight

UTC:

```text
00:00:00
```

Elapsed seconds:

```text
0
```

UMT:

```text
00X000.000
```

Compact:

```text
0X0
```

---

### 12.2 06:00 UTC

Elapsed seconds:

```text
21,600
```

Quanta:

```text
21,600 / 0.864
= 25,000
```

UMT:

```text
25X000.000
```

Compact:

```text
25X0
```

---

### 12.3 12:00 UTC

Elapsed seconds:

```text
43,200
```

Quanta:

```text
43,200 / 0.864
= 50,000
```

UMT:

```text
50X000.000
```

Compact:

```text
50X0
```

---

### 12.4 18:00 UTC

Elapsed seconds:

```text
64,800
```

UMT:

```text
75X000.000
```

Compact:

```text
75X0
```

---

### 12.5 18:30 UTC

Elapsed seconds:

```text
66,600
```

Quanta:

```text
66,600 / 0.864
= 77,083.333333...
```

At base UMT precision this becomes:

```text
77X083.333
```

---

## 13. Compact Representation and Precision

Compact notation MUST NOT be interpreted as an uncertainty interval.

For example:

```text
18X5
```

means exactly:

```text
18X500.000
```

It does **not** mean:

```text
18X500.000 through 18X599.999
```

Similarly:

```text
18X52
```

means:

```text
18X520.000
```

This distinction is important for software implementations.

Removing a non-zero digit therefore changes the represented time.

For example:

```text
18X521
```

cannot be shortened to:

```text
18X52
```

without changing its value.

Only positions whose removal is equivalent to replacing them with zero may be omitted without changing the represented instant.

---

## 14. Canonicalization

A conforming parser SHOULD be able to convert any valid compact representation into canonical form.

Examples:

```text
9X5
        -> 09X500.000

01X2
        -> 01X200.000

01X02
        -> 01X020.000

01X002
        -> 01X002.000

12X345.6
        -> 12X345.600

12X345.67
        -> 12X345.670
```

Canonical form SHOULD be used for:

* storage
* database keys
* machine-to-machine protocols
* sorting
* serialization
* test vectors

Compact form is primarily intended for human-facing display.

---

## 15. Ordering

Canonical UMT representations have fixed width.

Therefore canonical strings can be lexicographically ordered within the same day.

For example:

```text
09X999.999
10X000.000
10X001.000
42X500.000
```

are both lexicographically and chronologically ordered.

Compact representations MUST NOT be assumed to preserve lexical chronological ordering.

---

## 16. Range

The first representable instant of a UMT day is:

```text
00X000.000
```

The final representable base-resolution value is:

```text
99X999.999
```

The next value would be:

```text
100X000.000
```

which is not a valid time-of-day value.

Instead it represents:

```text
00X000.000
```

of the next day.

Implementations performing arithmetic MUST carry overflow into the date.

---

## 17. Base Resolution

The canonical three-digit Grana field provides a resolution of:

```text
1 Grana
```

or:

```text
864 microseconds
```

Applications requiring finer resolution MAY define additional decimal digits after the Grana field.

For example:

```text
42X317.0845
```

may be interpreted as a further decimal subdivision of one Grana.

Such extensions MUST preserve the decimal hierarchy.

The first three digits after the decimal point remain the base Grana field.

---

## 18. Leap Seconds

The core UMT model assumes a nominal day of exactly:

```text
86,400 SI seconds
```

UTC leap-second labels such as:

```text
23:59:60
```

therefore have no distinct representation in the base UMT clock.

Implementations requiring leap-second-aware astronomical timing MUST define an additional conversion policy.

Such a policy MUST NOT silently change the duration of a Chrona, Quanta, or Grana.

Possible external policies include:

* UTC leap-second normalization
* UTC smearing
* conversion through TAI
* conversion through another monotonic timescale

The selected policy SHOULD be documented by the implementation.

---

## 19. Dates

UMT itself defines a **time of day**, not a calendar system.

When a date is required, an ISO 8601 calendar date SHOULD be associated with the UMT value.

A recommended textual form is:

```text
YYYY-MM-DDTCCXQQQ.GGG
```

Example:

```text
2026-09-19T42X317.084
```

Because UMT is defined relative to UTC, this value inherently refers to the corresponding UTC date.

An implementation MAY append:

```text
Z
```

when explicit UTC association is desirable:

```text
2026-09-19T42X317.084Z
```

---

## 20. Parsing Rules

A conforming parser MUST:

1. identify the `X` separator;
2. parse the Chrona component;
3. reject Chrona values outside `0–99`;
4. parse zero to three Quanta digits;
5. right-pad Quanta to three digits;
6. if a decimal point exists, parse the fractional digits;
7. right-pad the base Grana component as required;
8. reject malformed or out-of-range representations.

A parser MUST NOT left-pad Quanta when interpreting compact precision.

For example:

```text
X2
```

means:

```text
X200
```

not:

```text
X002
```

---

## 21. Reference Parsing Examples

```text
Input           Canonical

0X0             00X000.000
0X1             00X100.000
0X01            00X010.000
0X001           00X001.000

9X5             09X500.000
9X05            09X050.000
9X005           09X005.000

12X3            12X300.000
12X34           12X340.000
12X345          12X345.000

12X345.6        12X345.600
12X345.06       12X345.060
12X345.006      12X345.006
```

---

## 22. Invalid Examples

The following are invalid:

```text
100X000.000
```

Reason:

```text
Chrona exceeds 99.
```

---

```text
12X1000.000
```

Reason:

```text
Quanta contains more than three base digits.
```

---

```text
12Y500.000
```

Reason:

```text
Invalid separator.
```

---

```text
-1X500.000
```

Reason:

```text
Negative time-of-day components are not valid.
```

---

## 23. Recommended Usage

### Human communication

Compact notation is recommended:

```text
18X5
```

---

### Timetables

Moderate precision may be sufficient:

```text
14X0
14X25
14X5
15X0
```

---

### Logs

Canonical or high-precision notation SHOULD be used:

```text
14X592.104
14X592.109
```

---

### Persistent storage

Canonical notation SHOULD be used:

```text
CCXQQQ.GGG
```

or the corresponding integer number of Grana since midnight.

---

## 24. Integer Representation

For efficient software implementations, a UMT time MAY be represented as the total number of Grana elapsed since midnight.

Given:

```text
C = Chrona
Q = Quanta
G = Grana
```

calculate:

```text
N = C × 1,000,000
  + Q × 1,000
  + G
```

The valid base range is:

```text
0 <= N < 100,000,000
```

This representation is exact for all base-resolution UMT values and avoids floating-point arithmetic.

Implementations SHOULD prefer integer arithmetic where practical.

---

## 25. Design Properties

UMT has the following properties:

* decimal subdivision throughout the clock;
* exactly 100 Chrona per nominal day;
* exactly 1000 Quanta per Chrona;
* exactly 1000 Grana per Quanta;
* fixed canonical representation;
* compact human representation;
* direct mapping to day fractions;
* deterministic integer representation;
* no AM/PM distinction;
* no time-zone component in the clock itself.

---

## 26. Canonical Summary

```text
Canonical format:
CCXQQQ.GGG

1 day:
100 Chrona

1 Chrona:
1000 Quanta
864 seconds

1 Quanta:
1000 Grana
0.864 seconds

1 Grana:
0.000864 seconds
0.864 milliseconds
864 microseconds

Valid time-of-day range:
00X000.000
through
99X999.999
```

---

## 27. Example

```text
UTC: 06:00:00
UMT: 25X000.000
Short: 25X0

UTC: 12:00:00
UMT: 50X000.000
Short: 50X0

UTC: 18:00:00
UMT: 75X000.000
Short: 75X0

UTC: 00:00:00
UMT: 00X000.000
Short: 0X0
```

---

# End of Specification

