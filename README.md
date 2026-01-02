# Universal Metric Time (UMT) Specification

## 1. Abstract
**Universal Metric Time (UMT)** is a decimal time system. It is designed for variable precision, allowing timestamps to be truncated or extended based on the required accuracy without changing their value context.

## 2. The Logic of Precision
UMT uses a "Significant Digits" approach for all sub-units.
- **`12X3`** implies **300** Quanta (Decimals of Chrona).
- **`.6`** implies **600** Grana (Decimals of Quanta).

Omitted digits on the right are always treated as zeros.

## 3. Format Specification

**Canonical Pattern:**
```text
[C]CX[Q...][.G...]
```

### 3.1 Chrona (The Hour)
*   **Symbol:** `X` (Separator)
*   **Format:** Integer, 0-99.
*   **Leading Zero:** Optional.
    *   `9X...` is valid.
    *   `09X...` is valid.

### 3.2 Quanta (The Second)
*   **Position:** Immediately after `X`.
*   **Format:** Variable length (0 to 3 digits).
*   **Semantics:** Decimal fraction of a Chrona.
    *   `X1`   = 100 Quanta
    *   `X12`  = 120 Quanta
    *   `X123` = 123 Quanta

### 3.3 Grana (The Millisecond)
*   **Separator:** `.` (Dot)
*   **Position:** Immediately after Quanta.
*   **Format:** Variable length (1 to 3 digits).
*   **Semantics:** Decimal fraction of a Quanta.
    *   `.6`   = 600 Grana
    *   `.67`  = 670 Grana
    *   `.678` = 678 Grana

---

## 4. Examples & Equivalencies

All timestamps in a row represent the **exact same moment in time**:

| Short / Variable | Canonical (Full Padding) | Logic |
| :--- | :--- | :--- |
| **12X345.6** | `12X345.600` | 6 is the first decimal ($\frac{6}{10}$), so 600 Grana. |
| **12X345.67** | `12X345.670` | First two decimals are known. |
| **9X5** | `09X500.000` | No leading zero on Chrona; Quanta is 500. |
| **01X2** | `01X200.000` | 200 Quanta. |
| **01X02** | `01X020.000` | 20 Quanta. |
| **01X002** | `01X002.000` | 2 Quanta. |

---

## 5. Usage Recommendations

### 5.1 Timetables (Low Precision)
Use single-digit Quanta for clean lists.
```text
14X0  (Start)
14X5  (Middle)
15X0  (Next Chrona)
```

### 5.2 System Logs (High Precision)
Use full Grana precision.
```text
[LOG] 14X592.104 - Process started
[LOG] 14X592.109 - Process finished
```

### 5.3 Human Communication
Omit Grana unless necessary.
```text
"Let's meet at 18X5." (18X500)
"The race finished at 18X521.4."
```

---

## 6. Conversion Guide (Local Time to UMT)

Since UMT is **UTC+0**, conversion requires normalizing to UTC first, then calculating the day-fraction.

### 6.1 The Constants
*   **Seconds in Day:** 86,400
*   **Seconds per Quanta:** 0.864

### 6.2 The Algorithm
1.  **Normalize:** Convert Local Time to UTC.
2.  **Sum Seconds:** Calculate total seconds passed since UTC midnight ($T_{sec}$).
    $$T_{sec} = (Hours \times 3600) + (Minutes \times 60) + Seconds + (Milliseconds / 1000)$$
3.  **Calculate UMT Raw Value:**
    $$UMT_{raw} = \frac{T_{sec}}{0.864}$$

### 6.3 Formatting Logic
Given a result like **`42317.084`**:

1.  **Chrona:** Integer part divided by 1000.
    *   $42317 / 1000 = 42$ $\rightarrow$ `42X`
2.  **Quanta:** Integer part modulo 1000.
    *   $42317 \pmod{1000} = 317$ $\rightarrow$ `317`
3.  **Grana:** The fractional part.
    *   $.084$ $\rightarrow$ `.084`

**Result:** `42X317.084`

### 6.4 Example Calculation
**Input:** 06:00:00 UTC (Morning)

1.  $T_{sec} = 6 \times 3600 = 21,600$
2.  $UMT_{raw} = 21,600 / 0.864 = 25,000$
3.  Format: `25X000` (or simply `25X0`)

**Input:** 18:30:00 UTC (Evening)

1.  $T_{sec} = (18 \times 3600) + (30 \times 60) = 66,600$
2.  $UMT_{raw} = 66,600 / 0.864 = 77,083.333...$
3.  Format: `77X083.333`
