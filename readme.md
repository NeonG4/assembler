# **Instruction Set Architecture (ISA) Reference**  
### *For the Custom 4‑Register CPU*  

---

## **1. Overview**

This document defines the complete instruction set for the custom 4‑register, byte‑addressed CPU. It includes register behavior, instruction semantics, assembler directives, and example usage. The ISA is designed to be minimal, expressive, and compatible with the enhanced Python assembler supporting variables, constants, macros, and directives.

---

## **2. Registers**

| Register | Purpose |
|---------|---------|
| **r1** | Immediate loads (`LOADI`), left operand for `CMP` |
| **r2** | RAM store source (`STORE`), right operand for `CMP`, jump target |
| **r3** | Loop counter, X coordinate, compare result for `JZ` |
| **r4** | Compare result (0 = equal, 1 = less, 2 = greater) |

---

## **3. Unified Instruction Table**

| Instruction | Parameters | Description |
|------------|------------|-------------|
| **LOADI** | literal | Load immediate value into **r1** |
| **STORE** | address | Store **r2** into RAM\[address\] |
| **MOV** | srcReg, destReg | Copy register → register |
| **PUSH2** | — | Push r2 into ALU queue |
| **ACC2R1** | — | Move accumulator → r1 |
| **ACC2Q** | — | Push accumulator into queue |
| **CLRQ** | — | Clear ALU queue |
| **SUM** | — | Sum queue → accumulator |
| **MUL** | — | Multiply queue → accumulator |
| **SAVEIP** | — | Save current instruction pointer into r1 |
| **JMP** | label | Jump to label (assembler expands into 6 bytes) |
| **JZ** | label | Jump to label if **r3 == 0** (assembler expands into 6 bytes) |
| **HALT** | — | Stop execution (IP resets to 0) |
| **INC3** | — | Increment **r3** by 1 |
| **CMP** | — | Compare r1 and r2 → r4 (0 = equal, 1 = r1<r2, 2 = r1>r2) |
| **DRAW** | xReg, yReg, colorReg | Draw pixel at (reg[x], reg[y]) with color reg[color] |
| **ERROR** | — | Load error code |
| **CLRERR** | — | Clear error code |

---

## **4. Control Flow Semantics**

### **JMP label**  
Assembler expands into:  
```
LOADI <addr>
MOV 1 2
JMP
```

### **JZ label**  
Assembler expands into:  
```
LOADI <addr>
MOV 1 2
JZ
```

### **Compare + Conditional Jump Pattern**
```
CMP
MOV 4 3
JZ label
```
`JZ` checks **r3**, not r4.

---

## **5. Comparison Semantics**

`CMP` compares **r1** and **r2** and writes the result to **r4**:

- `0` → equal  
- `1` → r1 < r2  
- `2` → r1 > r2  

`r4` must be moved into `r3` before `JZ`.

---

## **6. Graphics Instructions**

### **DRAW xReg yReg colorReg**

Draws a pixel at:

- X = reg[xReg]  
- Y = reg[yReg]  
- Color = reg[colorReg]  

This is a 4‑byte instruction.

---

## **7. Memory Model**

- RAM is byte‑addressed.  
- `STORE addr` writes **r2 → RAM[addr]**.  
- To store a literal into RAM:

```
LOADI value
MOV 1 2
STORE addr
```

---

## **8. Assembler Directives**

| Directive | Meaning |
|-----------|---------|
| **var name = value** | Allocates RAM, emits initialization code |
| **const NAME = value** | Compile‑time constant, no RAM used |
| **macro NAME args…** | Defines a macro |
| **end** | Ends a macro |
| **.org addr** | Sets code origin (pads with zeros) |
| **.byte v1, v2, …** | Inserts raw bytes into output |

---

## **9. Example Macro**

```
macro LOAD_RAM addr reg
    LOADI addr
    MOV 1 2
    MOV 2 reg
end
```

---

## **10. Example Program: Screen Clear**

```
var x = 0
var y = 0

const WIDTH = 60
const HEIGHT = 45
const COLOR = 19

macro LOAD_RAM addr reg
    LOADI addr
    MOV 1 2
    MOV 2 reg
end

OUTER_LOOP:
LOADI 0
MOV 1 2
STORE x

INNER_LOOP:
LOAD_RAM x 3
LOAD_RAM y 2
LOADI COLOR
DRAW 3 2 1

LOAD_RAM x 3
INC3
MOV 3 2
STORE x

LOAD_RAM x 1
LOADI WIDTH
MOV 1 2
CMP
MOV 4 3
JZ AFTER_INNER

JMP INNER_LOOP

AFTER_INNER:
LOAD_RAM y 3
INC3
MOV 3 2
STORE y

LOAD_RAM y 1
LOADI HEIGHT
MOV 1 2
CMP
MOV 4 3
JZ HALT_LABEL

JMP OUTER_LOOP

HALT_LABEL:
HALT
```

---

## **11. Known Quirks & Hazards**

- `LOADI` always overwrites **r1**  
- `STORE` always writes **r2 → RAM[address]**  
- `CMP` always uses **r1** and **r2**  
- `JZ` checks **r3**, not r4  
- `INC3` only increments **r3**  
- `DRAW` requires three registers  
- `JMP` and `JZ` are pseudo‑instructions (6 bytes each)  

---

**End of ISA Document**
