
SHENZHEN I/O — Command Sheet & Learning Path
Compiled from SHENZHEN IO Manual (English).pdf (the in-box 诚尚Micro / MCxxxx binder).
Page references point at that PDF.

---

## PART 1 — THE COMMAND SHEET

### 1.1 Operand notation (manual p.4, p.14)

| Notation | Meaning |
|---|---|
| `R` | Register |
| `I` | Integer, **must be −999 to 999** |
| `R/I` | Either a register or an integer |
| `P` | Pin register only (`p0`, `p1`, `x0`…) |
| `L` | Label (must be defined somewhere in the program) |

### 1.2 Registers (p.14)

| Register | What it is |
|---|---|
| `acc` | Primary general-purpose accumulator. **Every arithmetic instruction implicitly reads and writes `acc`.** Initialised to 0. |
| `dat` | Second general-purpose register, only on some chips (e.g. MC6000). Usable almost anywhere `acc` is. Initialised to 0. |
| `p0`, `p1` | Simple I/O pin registers. |
| `x0`–`x3` | XBus pin registers. |
| `null` | Pseudo-register. **Reading gives 0; writing is discarded.** Great for throwing away an XBus packet you don't need. |

Using a register the chip doesn't physically have is a compile error — check the datasheet pinout first.

### 1.3 Basic instructions (p.15)

| Command | Signature | What it does |
|---|---|---|
| `nop` | `nop` | Does nothing. Burns 1 instruction of power/time. Useful as a placeholder or padding. |
| `mov` | `mov R/I R` | Copy first operand **into** second operand. This is also how you read/write pins: `mov 100 p1` (write), `mov p0 acc` (read). |
| `jmp` | `jmp L` | Jump to the instruction **after** the named label. |
| `slp` | `slp R/I` | Sleep for N **time units**. Consumes no power while asleep. |
| `slx` | `slx P` | Sleep until data is available to read on the given **XBus** pin. Only works on XBus pins. |

### 1.4 Arithmetic instructions (p.15)

All results land in `acc`. Values clamp to the −999…999 range — `acc=800` then `add 400` gives **999**, not 1200.

| Command | Signature | What it does |
|---|---|---|
| `add` | `add R/I` | `acc = acc + operand` |
| `sub` | `sub R/I` | `acc = acc − operand` |
| `mul` | `mul R/I` | `acc = acc × operand` |
| `not` | `not` | Logical NOT: if `acc == 0` → `acc = 100`; **any other value** → `acc = 0`. |
| `dgt` | `dgt R/I` | Isolate one decimal digit of `acc` (0 = ones, 1 = tens, 2 = hundreds) and leave just that digit in `acc`. |
| `dst` | `dst R/I R/I` | Set digit *#first* of `acc` to value *second*. |

**`dgt` / `dst` worked examples (p.15):**

| `acc` | Instruction | Result |
|---|---|---|
| 596 | `dgt 0` | 6 |
| 596 | `dgt 1` | 9 |
| 596 | `dgt 2` | 5 |
| 596 | `dst 0 7` | 597 |
| 596 | `dst 1 7` | 576 |
| 596 | `dst 2 7` | 796 |

There is **no divide instruction** — division needs the MC4010 co-processor, or you build it out of loops/`sub`.

### 1.5 Test instructions & conditional execution (p.14, p.16)

A test instruction sets a hidden flag. That flag enables/disables every following instruction that is prefixed with `+` or `-`. Instructions with **no** prefix always run.

- **All conditional instructions start disabled** — a test must run first.
- A disabled instruction is skipped and **consumes no power**.
- The flag persists until the next test instruction, so one test can gate several lines.

| Command | Signature | `+` runs when… | `-` runs when… |
|---|---|---|---|
| `teq` | `teq R/I R/I` | A **==** B | A **!=** B |
| `tgt` | `tgt R/I R/I` | A **>** B | A **<=** B |
| `tlt` | `tlt R/I R/I` | A **<** B | A **>=** B |
| `tcp` | `tcp R/I R/I` | A **>** B | A **<** B |

`tcp` is the three-way compare: if **A == B**, *both* `+` and `-` lines are disabled — that's how you get an "equal" branch for free.

### 1.6 Program structure & syntax (p.13)

Each line is, strictly in this order — **all parts optional**:

```
LABEL   CONDITION   INSTRUCTION   COMMENT
```

```asm
# This line is a comment.
loop:            # until ACC is ten
  teq acc 10
+ jmp end
  mov 50 x2
  add 1
  jmp loop
end:
  mov 0 acc      # reset counter
```

- Comments: everything after `#` to end of line.
- Labels: first on the line, end with `:`, start with a letter, may contain letters/digits/underscores.
- Operands are separated by **spaces only** — no commas, no brackets.

### 1.7 Pins — simple I/O vs XBus (p.9)

The single most important mechanical concept in the game. **The two types are not interoperable** — a pin can only connect to a pin of the same type.

| | **Simple I/O** | **XBus** |
|---|---|---|
| Marking on chip | unmarked | **yellow dot** |
| Value range | 0 to 100, continuous signal level | −999 to 999, discrete data **packets** |
| Timing | Read/write **any time**, ignores what's connected | **Synchronised** — transfer only happens when a reader *and* a writer are both attempting it |
| If the other side isn't ready | no problem, you just read/write a level | **your chip blocks** (stalls) until the other side acts |
| Typical use | buttons, switches, mics, LEDs, speakers, motors | chip-to-chip data, keypads, numeric displays |
| Registers | `p0`, `p1` | `x0`–`x3` |

Extra gotcha (p.18/19): a simple I/O pin is in **either** input or output mode at any moment. **Writing** to `pN` puts it in output mode at that value; **reading** from `pN` puts it in input mode and *clears the output value you had set*.

### 1.8 Time units and sleep (Application Note 393, p.10)

- The CPU is far faster than the signals it drives; it can run a very large number of instructions inside one time unit.
- To advance to the **start of the next time unit**, go to sleep: `slp`.
- Square wave on `p1`, 3 units on / 3 units off:

```asm
  mov 100 p1
  slp 3
  mov 0 p1
  slp 3
```

Practical rule: if your output needs to be "one value per time unit," your loop must contain exactly one `slp 1` (or one blocking XBus op) per iteration.

---

## PART 2 — PARTS QUICK REFERENCE

### 2.1 Microcontrollers (p.18–19)

| Chip | Program lines | Registers | XBus pins | Simple I/O pins |
|---|---|---|---|---|
| **MC4000** | 9 | `acc` | 2 (`x0`,`x1`) | 2 (`p0`,`p1`) |
| **MC4000X** | — | `acc` | XBus-only variant | — |
| **MC6000** | 14 | `acc`, `dat` | 4 (`x0`–`x3`) | 2 (`p0`,`p1`) |

### 2.2 MC4010 Math Co-processor (p.29)

Write a **command sequence** (op code, then 1–2 values) to any pin; read any pin to get `result` back. `result` can be re-read any number of times. Range-limited to −999…999.

| Operation | Sequence | `result` |
|---|---|---|
| Set | `10 A` | A |
| Add | `20 A B` | A + B |
| Subtract | `30 A B` | A − B |
| Multiply | `40 A B` | A × B |
| **Divide** | `50 A B` | A / B |
| Remainder | `51 A B` | remainder of A/B (negative if A was negative) |
| Modulus | `60 A B` | A mod B (negative if B was negative) |
| Exponent | `70 A B` | A^B |
| Square root | `80 A` | √A, rounded down |
| Min | `90 A B` | smaller of A, B |
| Max | `91 A B` | larger of A, B |

```asm
# x0 is connected to an MC4010 input pin
mov 50 x0   # op code for division
mov 20 x0   # first value
mov 4  x0   # second value
mov x0 acc  # read back result (5)
```

### 2.3 DX300 Digital I/O Expander (p.20–21)

Turns one XBus packet into three simple I/O lines (or vice versa). Three XBus pins, three simple I/O pins (`p0`,`p1`,`p2`).

- **Write** a 3-digit number → each digit sets a pin on (`1` → 100) or off (`0` → 0).
- **Read** → get a 3-digit number encoding the current pin states.
- Digit mapping: **ones = `p0`, tens = `p1`, hundreds = `p2`.**
- Like a simple I/O pin, the whole chip is input-mode *or* output-mode: writing puts it in output mode, reading puts it in input mode and clears set outputs.

| XBus value | p0 | p1 | p2 |
|---|---|---|---|
| 100 | 0 | 0 | 100 |
| 011 | 100 | 100 | 0 |
| 000 | 0 | 0 | 0 |

### 2.4 Memory (p.22–23)

| Chip | Type | Cells | Pins |
|---|---|---|---|
| **100P-14** | RAM (read/write) | 14, all init to 0 | `a0`,`a1` = pointers; `d0`,`d1` = data |
| **200P-14** | ROM (read-only) | 14, values set by you at design time | `a0`,`a1` = pointers (read); `d0`,`d1` = data (read) |

- Both pointers start at address 0.
- Write a pointer pin (`aN`) to seek; read/write a data pin (`dN`) to access the cell.
- **After every data-pin access the matching pointer auto-increments.** Two independent pointers = read and write streams that don't interfere.

### 2.5 LC70Gxx Logic Gates (p.24)

Simple I/O only. Inputs: **< 50 = off, ≥ 50 = on.**

| Part | Gate |
|---|---|
| **LC70G04** | Inverter (1 in, 1 out) |
| **LC70G08** | AND (2 in) |
| **LC70G32** | OR (2 in) |
| **LC70G86** | XOR (2 in) |

Note the LC70G04 exposes both a normal and an inverted output pin.

### 2.6 Input/output peripherals

| Part | Page | Interface | Behaviour |
|---|---|---|---|
| **N4PB-8000** push-button controller | p.28 | 4 non-blocking XBus | Read `[button]` = down event, `-[button]` = up event, `-999` = no new events. Up to 8 buttons. |
| **LX700** 7-segment display | p.28 | XBus | Write −199…199 to display; write `-999` to blank all segments. 2.5 digits + minus sign. |
| **LX910C** custom LCD w/ touch | p.28 | XBus `cN`/`tN`/`qN` | `cN` write `[seg]` = on, `-[seg]` = off, `999` = all on, `-999` = all off. `tN` read `[seg]` = touch, `-[seg]` = release, `-999` = nothing new. `qN` write a segment to query it, then read `1`/`0`. |
| **C2S-RF901** RF transceiver | p.26 | 2 XBus (transmit/receive) | Wireless link. **Non-blocking buffer: reading with no data yields `-999`** instead of blocking. |
| **NLP2** speech recogniser | p.33 | XBus keywords + simple I/O audio | Keywords arrive as **pairs of 3-digit values**; non-blocking, yields `-999` when empty. Also passes raw audio through. |
| **DT2415** incremental clock | p.25 | simple I/O | Emits the number of **15-minute increments since midnight**: 0 at 00:00–00:14, up to 95 at 23:45–23:59. |
| **D80C010-F** security chip | p.30 | 2 read-only XBus | Stores a unique ID value you read over XBus. |
| **KUJI-EK1 "Oracle Engine"** | p.31 | 2 simple I/O (button, oracle) | On use, emits an I Ching hexagram as **6 values over 6 time units**, lowest line first; `100` = solid line, `0` = broken. |
| **FM/iX FM Blaster** | p.27 | — | 1-voice FM tone generator, 10 preset instruments (00 Harpsiclav … 09 Bass Drum), MIDI-ish note numbers with 60 = middle C. New note or instrument change cuts the current note. |
| **PGA33X6** logic array | p.32 | 3 buffered in / 3 buffered out | Sum-of-products programmable logic, 6 product columns, 1 set/reset flip-flop with feedback. **Not suited to low-power applications.** (Datasheet is in Chinese only — by design.) |

---

## PART 3 — HOW TO LEARN THE GAME WITH THIS MANUAL

### 3.1 First, set the manual up the way it's meant to be used (p.1)

The manual is deliberately built as a **reference binder**, not a tutorial you read front-to-back. Its intended physical form:

1. Cover page in front.
2. **Reference Card** (p.4) folded into quarters — kept loose, next to you.
3. Three story documents (two emails p.5–6, visa form p.7) in the front pocket.
4. Five tabbed sections: **Application Notes → Language Reference → Parts Datasheets → Supplemental Data → Engineering Notes**.

If you're not printing it: at minimum keep **page 4 (the Reference Card)** open on a second monitor/phone, and bookmark pages 13–16 (Language Reference) and 18–33 (Datasheets). That single choice is most of the difference between the game feeling opaque and feeling fair.

### 3.2 The reading order that actually works

| Step | Read | Why |
|---|---|---|
| 1 | **p.9 — App Note 268** (simple I/O vs XBus) | Everything else depends on this distinction. Learn it before touching a puzzle. |
| 2 | **p.13 — Language Reference, "Program Structure"** | Line layout, labels, comments. Five minutes. |
| 3 | **p.15 — Basic + Arithmetic instructions** | Skip `dgt`/`dst` on first pass; you won't need them for hours. |
| 4 | **p.10 — App Note 393** (sleep/time units) | The mental model for *when* things happen, not just what. |
| 5 | **p.14 — Conditional Execution** + **p.16 — Test instructions** | This is the game's branching system and it's unlike normal assembly. Re-read it. |
| 6 | **p.11 — App Note 650** (touch-activated light) | A complete two-chip reference design combining XBus, `slx`, conditionals and state in `acc`. It is the manual's worked example — study it line by line. |
| 7 | Datasheets **on demand only** (p.18–33) | Read the datasheet for a part *when a puzzle hands you that part*. Don't front-load. |
| 8 | Supplemental Data (p.35–46) **as puzzle input** | These aren't lore — they're the actual specs for individual puzzles (colour tables, meat patterns, sector maps, sound-effect value lists, keyword codes). When a puzzle briefing seems to be missing information, the missing information is in this section. |

### 3.3 The per-puzzle loop

1. **Read the email brief.** Note the required output format and, critically, the **timing**: one value per time unit? Only while a trigger is high? Only on a rising edge?
2. **Find the matching Supplemental Data page.** If the brief references a table, diagram, colour, or product list, the real numbers are in the manual, not the game UI.
3. **Pull the datasheet for every unfamiliar part** *before* wiring anything — the pin type (yellow dot or not) determines the whole layout.
4. **Sketch the dataflow on paper** (that's what the Engineering Notes tab is for): which chip owns which decision, what travels over which bus.
5. **Write the simplest version that works.** Cost/power/lines optimisation comes after correctness.
6. **Simulate and step.** Watch a single time unit at a time. Most bugs are timing, not logic.
7. **Then optimise**, in this order of payoff: remove chips → shorten wires → cut instructions executed (use `+`/`-` so disabled lines cost nothing) → replace busy-waiting with `slp`/`slx`.

### 3.4 Debugging checklist — the mistakes the manual warns about

- **Blocked on XBus.** If a chip freezes, there's a reader with no writer or a writer with no reader. Simple I/O never blocks; XBus always can.
- **Conditionals never firing.** They start **disabled**. No test instruction has run yet, or a later test overwrote the flag.
- **Lost pin output.** You wrote a value to `pN` then later *read* `pN` — reading flips the pin to input mode and clears your output.
- **Silent clamping.** Arithmetic pins at ±999 instead of overflowing. `mul` is the usual culprit.
- **Off-by-one time units.** One `slp`/blocking op per output value, no more, no less.
- **Wrong register on wrong chip.** `dat`, `x2`, `x3` don't exist on an MC4000.
- **`not` isn't bitwise.** It maps 0 → 100 and everything else → 0.
- **No division.** Reach for the MC4010, or restructure.

### 3.5 What "good" looks like

The score screen tracks **cost, power, and lines of code**, and they trade off against each other. Recommended practice: solve it, submit it, look at the histogram, then do **one** optimisation pass. Chasing all three metrics on every puzzle is how people stall out — the game expects you to leave some on the table and keep moving. The genuine skill being built is reading a terse datasheet and a vague brief and turning them into a timing-correct design; the histograms are just feedback.

---

*Other guides in this folder worth using after the manual: the MCxxxx Reference Card 2.0