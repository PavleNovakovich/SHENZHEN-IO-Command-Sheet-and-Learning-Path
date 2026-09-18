# 诚尚 MicroTM MCxxxx Family
## Language Reference Card

### Instruction Set

| Basic Instructions | Arithmetic Instructions | Test Instructions |
| :--- | :--- | :--- |
| **`nop`** | **`add`** `R/I` | **`teq`** `R/I` `R/I` |
| **`mov`** `R/I` `R` | **`sub`** `R/I` | **`tgt`** `R/I` `R/I` |
| **`jmp`** `L` | **`mul`** `R/I` | **`tlt`** `R/I` `R/I` |
| **`slp`** `R/I` | **`not`** | **`tcp`** `R/I` `R/I` |
| **`slx`** `P` | **`dgt`** `R/I` | |
| | **`dst`** `R/I` `R/I` | |

---

### Registers

* **`acc`**
* **`dat`** <sup>[1]</sup>
* **`p0`**, **`p1`** <sup>[1]</sup>
* **`x0`**, **`x1`**, **`x2`**, **`x3`** <sup>[1]</sup>

---

### Notation & Operands

| Notation | Meaning |
| :--- | :--- |
| **`R`** | Register |
| **`I`** | Integer <sup>[2]</sup> |
| **`R/I`** | Register or integer <sup>[2]</sup> |
| **`P`** | Pin register (`p0`, `p1`, etc.) |
| **`L`** | Label <sup>[3]</sup> |

---

### Notes

1. Not all registers are available on all microcontrollers. Refer to the parts datasheets for pin diagrams and register information.
2. Integer values must be in the range **-999** to **999**.
3. Labels used as operands must be defined elsewhere in the program.