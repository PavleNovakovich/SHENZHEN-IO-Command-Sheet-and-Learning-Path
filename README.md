# SHENZHEN I/O — Command Sheet & Learning Path

A beginner-friendly reference for understanding **SHENZHEN I/O** — its programming language, registers, I/O systems, components, timing, and the general process of solving puzzles.

This project is intended as a **quick reference and simplified learning guide** for players who find the original in-box manual difficult to navigate or who want a practical reference while working through the game.

> **This guide is a player-created simplification of the SHENZHEN I/O manual. It is intended as a learning aid and quick reference, not as a replacement for the original manual.**

---

## 📖 Main Guide

The main document contains the complete **Command Sheet & Learning Path**:

It is divided into three main parts:

### Part 1 — The Command Sheet

A quick reference for the SHENZHEN I/O programming language, including:

- Operand notation
- Registers
- Basic instructions
- Arithmetic instructions
- Test instructions and conditional execution
- Program structure and syntax
- Simple I/O vs XBus
- Time units and `slp`

This section is designed to be useful while you are actively solving a puzzle, rather than something you need to memorize from beginning to end.

### Part 2 — Parts Quick Reference

A compact reference for the components you encounter in the game, including:

- MC4000 and MC6000 microcontrollers
- MC4010 math co-processor
- DX300 Digital I/O Expander
- 100P-14 and 200P-14 memory
- LC70Gxx logic gates
- Input/output peripherals
- Other specialized components

Use this section **on demand** when a puzzle introduces a component you are unfamiliar with.

### Part 3 — How to Learn the Game

A practical learning path based on the structure of the original manual.

It covers:

1. How to use the manual
2. A recommended reading order
3. A per-puzzle workflow
4. A debugging checklist
5. How to approach optimization

---

## 🚀 Where Should I Start?

If you're completely new to **SHENZHEN I/O**, don't try to read everything at once.

A recommended starting path is:

1. **Learn Simple I/O vs XBus**
2. **Understand basic program structure**
3. **Learn the basic commands**
4. **Learn arithmetic**
5. **Understand time units and `slp`**
6. **Learn conditional execution**
7. **Study the worked example**
8. **Use component datasheets only when a puzzle requires them**

The main guide contains a more detailed version of this learning path.

---

## 💡 Quick Reference

Some of the most important concepts to understand early are:

### `acc`

The primary general-purpose accumulator. Arithmetic instructions implicitly read from and write to `acc`.

### Simple I/O vs XBus

These are two different systems and behave differently.

**Simple I/O**
- Uses pins such as `p0` and `p1`
- Uses continuous signal levels
- Can be read or written at any time

**XBus**
- Uses pins such as `x0`–`x3`
- Transfers discrete data packets
- Transfers are synchronized
- A chip can block while waiting for the other side

Understanding this distinction is fundamental to solving SHENZHEN I/O puzzles.

### Conditional execution

Test instructions such as `teq`, `tgt`, `tlt`, and `tcp` control whether subsequent instructions prefixed with `+` or `-` execute.

### Timing

`slp` advances the program to the start of a later time unit. Timing is often just as important as the logic of the program.

---

## 🧩 A Practical Puzzle-Solving Workflow

When approaching a new puzzle:

1. **Read the brief carefully.**
   - Identify the required output.
   - Pay attention to timing requirements.

2. **Check the relevant supplemental data.**
   - Tables, diagrams, colour information, product lists, and other puzzle-specific data may be provided in the manual.

3. **Read the datasheet for unfamiliar components.**
   - Pay particular attention to the type of each pin.

4. **Sketch the dataflow.**
   - Decide which chip handles each part of the problem and what data travels between chips.

5. **Build the simplest working solution first.**

6. **Simulate and step through the program.**
   - Timing problems are often the source of unexpected behaviour.

7. **Optimize only after the solution works.**
   - Consider chips, wiring, instruction count, power, and cost.

---

## 🐛 Common Problems

If something isn't working, check:

- Is an XBus connection waiting for a reader or writer?
- Has a test instruction actually run before a conditional instruction?
- Did reading a simple I/O pin switch it into input mode?
- Did an arithmetic operation clamp a value to the `-999` to `999` range?
- Is the program correctly synchronized with the required time units?
- Are you using a register that exists on the particular microcontroller?
- Are you treating `not` as logical NOT rather than bitwise NOT?
- Do you need the MC4010 for division?

The full **Debugging Checklist** is included in Part 3 of the main guide.

---

## 📚 About the Source Material

The Command Sheet & Learning Path was compiled using the **SHENZHEN I/O manual** supplied with the game. The main guide includes page references to the source manual so that readers can consult the original documentation when they need more detail.

The original manual is structured primarily as a **reference binder**, rather than a tutorial intended to be read from beginning to end. This project reorganizes some of that information into a more approachable quick-reference and learning path.

---

## 🎯 Purpose of This Repository

The goal is simple:

> **Make SHENZHEN I/O easier to learn without removing the challenge of solving the puzzles yourself.**

This guide is meant to sit beside the game while you play — something you can quickly search when you forget a command, don't remember how a component works, or aren't sure what to study next.

---

## 📁 Repository Structure

```text
shenzhen-io-command-sheet/
│
├── README.md
│
└── SHENZHEN-IO-Command-Sheet-and-Learning-Path
```

Additional examples and references can be added as the project grows.

---

## ⚠️ Note

SHENZHEN I/O and its associated materials are the property of their respective rights holders. This repository contains a player-created reference and simplification intended to help with learning and understanding the game.
