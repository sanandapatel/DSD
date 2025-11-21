# Week 8: Immediate Operands in Processor Design - Detailed Lecture Notes
## Complete Explanation with Code Comments and Architectural Considerations

---

## **Overview: Extending Basic Processor Functionality**

This lecture addresses a critical gap left from earlier discussions about ALU operations. We expand processor capability by introducing **immediate operands** — constants embedded directly in instructions. This is essential for modern processor design and dramatically improves code efficiency.

**Key Question:** How do we operate on constants like 0, 1, or bit masks (0xFF, 0xF0) without requiring separate registers to store them?

---

## **Part 1: The Problem - Why Immediate Operands?**

### **1.1 Limitation of Register-Only Operations**

#### **Current Instruction Model (Register-Only)**

```
Instruction Format: ADD Rd, Rs1, Rs2
Meaning: Rd = Rs1 + Rs2

Example: ADD R3, R1, R2
Semantics: R3 ← R1 + R2

Constraint: ALL operands must be in registers
Problem: What if we want to add 1 to a value?
         - We need to have 1 stored in a register (wastes a register)
         - We need a second instruction to load 1 into that register
         - Total: 2 instructions minimum for a simple increment
```

#### **Concrete Problem Scenario: Quadratic Discriminant**

```
Recall: Δ = b² - 4ac

Computation breakdown:
1. temp1 = b * b         (multiply b by itself)
2. temp2 = a * c         (multiply a by c)
3. temp3 = temp2 * 4     ← PROBLEM HERE
4. result = temp1 - temp3

Issue with step 3:
- We need to multiply temp2 by the constant 4
- But with register-only operations: ADD R4, R0, R2
- What is R2? We'd need to ensure the value 4 is stored there
- But how did 4 get into R2 in the first place?

Current workaround (inefficient):
Instruction 1: LOAD R7, 4      # Load 4 into register R7 (hypothetical)
Instruction 2: MUL R4, R6, R7  # Multiply temp2 (in R6) by 4 (from R7)

Problem: Register R7 is "occupied" just for storing the constant 4!
```

### **1.2 Common Use Cases for Constants**

#### **Initialization to Zero**

```c
// C Code
int x = 0;      // Initialize to zero

// Without immediate operands:
LOAD R1, 0      # Load 0 into R1
MOV  R0, R1     # Copy to target (or use R0 if hardwired)

// With immediate operands:
ADD  R0, R0, 0  # R0 = R0 + 0 = 0 (R0 = immediate 0)
```

**Statistical Fact:** Zero is the most frequently used constant in programming!
- Some researchers claim it appears in >30% of instructions
- It makes sense: array indices, loop counters, flag initialization, null pointers

#### **Increment/Decrement Operations**

```c
// C Code
count++;        // Increment by 1

// Without immediate operands:
LOAD R7, 1      # Load 1 into temporary register R7
ADD  R1, R1, R7 # R1 = R1 + 1

// With immediate operands:
ADD  R1, R1, 1  # R1 = R1 + immediate 1
```

**Usage Frequency:** Extremely common in loops and counters

#### **Bit Masking for Embedded Programming**

```c
// C Code - Extract low byte from 32-bit value
uint32_t x = read_register();
uint8_t low_byte = x & 0xFF;       // Mask with 0xFF
uint8_t mid_bits = (x >> 4) & 0x0F; // Mask with 0x0F

// Without immediate operands:
LOAD R3, 0xFF   # Load mask into R3
AND  R0, R1, R3 # R0 = R1 & 0xFF

LOAD R4, 0x0F   # Load different mask
SRL  R2, R1, 4  # R2 = R1 >> 4
AND  R2, R2, R4 # R2 = R2 & 0x0F

// Problem: Wasted registers (R3, R4) just for masks
// Also: Multiple instructions to achieve simple masking

// With immediate operands:
AND  R0, R1, 0xFF   # R0 = R1 & immediate 0xFF
SRL  R2, R1, 4      # R2 = R1 >> immediate 4
AND  R2, R2, 0x0F   # R2 = R2 & immediate 0x0F

// Much cleaner! No extra registers needed for masks
```

### **1.3 Proposed Solution: Immediate Operands**

**Definition:** An **immediate operand** is a constant value that is embedded directly in the instruction, rather than stored in a register.

```
Instruction Anatomy (without immediate):
┌─────────────────────────────────────────────┐
│ Opcode │ Rd │ Rs1 │ Rs2 │ ... (other bits)  │
└─────────────────────────────────────────────┘
 (e.g., 2 bits)(3 bits)(3 bits)

Instruction Anatomy (with immediate):
┌──────────────────────────────────────────────────┐
│ Opcode │ Reg Type │ Rd │ Rs │ Immediate Value │
│        │ (1 bit:  │    │    │  (5-8 bits)     │
│        │ reg vs   │    │    │                 │
│        │ imm)     │    │    │                 │
└──────────────────────────────────────────────────┘

Key Difference:
- Register-only: THREE register addresses
- Immediate: TWO register addresses + ONE constant

Trade-off Analysis:
┌─────────────────────────────────────────────┐
│ BENEFIT               │ COST                │
├─────────────────────────────────────────────┤
│ Reduce register      │ Immediate size      │
│ pressure             │ limited to bits     │
│                      │ available in instr  │
├─────────────────────────────────────────────┤
│ Eliminate extra      │ Can't use arbitrary │
│ load instructions    │ large constants     │
│                      │ (need workarounds)  │
├─────────────────────────────────────────────┤
│ More efficient code  │ Hardware slightly   │
│                      │ more complex        │
└─────────────────────────────────────────────┘
```

---

## **Part 2: Instruction Format Design**

### **2.1 Instruction Types**

#### **Type R: Register-Register Operations**

```verilog
// Format: op Rd, Rs1, Rs2
// All three operands from registers

Instruction Layout (hypothetical 16-bit):
┌────┬───┬───┬───┬─────────┐
│ op │ Rd│Rs1│Rs2│ unused  │
│ 2b │ 3b│ 3b│ 3b│  5b     │
└────┴───┴───┴───┴─────────┘
Total: 16 bits

Example: ADD R3, R1, R2
Encoding:
- op = 00 (ADD operation)
- Rd = 011 (R3)
- Rs1 = 001 (R1)  
- Rs2 = 010 (R2)
- unused = 00000

Binary: 0000110001010000 (16 bits)
```

#### **Type I: Register-Immediate Operations**

```verilog
// Format: op Rd, Rs, Immediate
// One immediate operand, one register operand

Instruction Layout (hypothetical 16-bit):
┌────┬───┬───┬──────────┐
│ op │ Rd│ Rs│Immediate │
│ 2b │ 3b│ 3b│   5b     │
└────┴───┴───┴──────────┘
Total: 16 bits

Example: ADD R3, R1, 5 (meaning: R3 = R1 + 5)
Encoding:
- op = 00 (ADD immediate operation)
- Rd = 011 (R3, destination)
- Rs = 001 (R1, source register)
- Immediate = 00101 (5 in binary)

Binary: 0000110001_00101 (16 bits)

Key Insight:
- Same total bit width as Type R
- Trade: Lost Rs2 register field for immediate field
- Same-size field: 3 bits for register → 5 bits for immediate
```

### **2.2 Instruction Set: R-type vs I-type Operations**

#### **Complete Instruction Breakdown**

```
R-type (Register-Register):
┌──────────────────────────────────────────────┐
│ Operation │ Format        │ Semantics        │
├──────────────────────────────────────────────┤
│ ADD       │ ADD Rd, Rs1, Rs2  │ Rd←Rs1+Rs2 │
│ SUB       │ SUB Rd, Rs1, Rs2  │ Rd←Rs1-Rs2 │
│ MUL       │ MUL Rd, Rs1, Rs2  │ Rd←Rs1*Rs2 │
│ DIV       │ DIV Rd, Rs1, Rs2  │ Rd←Rs1/Rs2 │
│ AND       │ AND Rd, Rs1, Rs2  │ Rd←Rs1&Rs2 │
│ OR        │ OR  Rd, Rs1, Rs2  │ Rd←Rs1|Rs2 │
│ SHL       │ SHL Rd, Rs1, Rs2  │ Rd←Rs1<<Rs2│
│ SHR       │ SHR Rd, Rs1, Rs2  │ Rd←Rs1>>Rs2│
└──────────────────────────────────────────────┘

I-type (Register-Immediate):
┌────────────────────────────────────────┐
│ Operation │ Format         │ Semantics │
├────────────────────────────────────────┤
│ ADDI      │ ADDI Rd, Rs, Imm  │ Rd←Rs+Imm │
│ SUBI      │ SUBI Rd, Rs, Imm  │ Rd←Rs-Imm │
│ MULI      │ MULI Rd, Rs, Imm  │ Rd←Rs*Imm │
│ ANDI      │ ANDI Rd, Rs, Imm  │ Rd←Rs&Imm │
│ ORI       │ ORI  Rd, Rs, Imm  │ Rd←Rs|Imm │
│ SHLI      │ SHLI Rd, Rs, Imm  │ Rd←Rs<<Imm│
│ SHRI      │ SHRI Rd, Rs, Imm  │ Rd←Rs>>Imm│
└────────────────────────────────────────┘

Note: No DIV I, because division by non-power-of-2
constants is uncommon (shift is more useful)
```

### **2.3 Opcode Encoding Strategy**

#### **How Many Bits for Opcode?**

```
Requirement: Distinguish between instruction types
- R-type instructions: ADD, SUB, MUL, DIV, AND, OR, SHL, SHR = 8 types
- I-type instructions: ADDI, SUBI, MULI, ANDI, ORI, SHLI, SHRI = 7 types
- Total: ~15 distinct operations

Bits needed: ⌈log₂(15)⌉ = 4 bits

But we also need a bit to distinguish R-type from I-type!
Some design options:

Option 1: Use high bit to distinguish
┌────┬──────────────┐
│Typ1│   op_code    │
│ 1b │    5b        │
└────┴──────────────┘
- Bit [6]: 0 = R-type, 1 = I-type
- Bits [5:0]: Operation code within that type
- Total: 6 bits for type + opcode

Option 2: Dedicate bits to opcode type
┌──────────────────────────────────────┐
│   opcode_type  │ opcode_within_type  │
│       2b       │         4b          │
└──────────────────────────────────────┘
- Bits [7:6]: 00 = Arithmetic, 01 = Logical, etc.
- Bits [5:2]: Specific operation within category

Option 3: Simpler - Use entirely different opcodes
- ADD (R-type) = 00
- ADDI (I-type) = 01
- SUB (R-type) = 10
- SUBI (I-type) = 11
- etc.

Our Choice: Option 3 (Simplest for our example)
- Bits [1:0]: Distinguishes ADD/ADDI, SUB/SUBI, etc.
- This is what most real ISAs do (RISC-V, MIPS, ARM)
```

#### **Practical Encoding Example**

```
Encoding scheme for 8 operations with I-type variants:

Operation Code:
┌──────────────────────────────────────────────┐
│ Instruction │ Bit [7:6] │ Bit [1:0] │ Notes │
├──────────────────────────────────────────────┤
│ ADD         │ xx        │ 00        │ R-type│
│ ADDI        │ xx        │ 01        │ I-type│
│ SUB         │ xx        │ 10        │ R-type│
│ SUBI        │ xx        │ 11        │ I-type│
└──────────────────────────────────────────────┘

Alternative (more intuitive):
┌──────────────────────────────────────────────┐
│ Instruction │ Opcode (3b) │ Type (1b) │      │
├──────────────────────────────────────────────┤
│ ADD/ADDI    │ 000         │ 0/1       │      │
│ SUB/SUBI    │ 001         │ 0/1       │      │
│ MUL/MULI    │ 010         │ 0/1       │      │
│ AND/ANDI    │ 011         │ 0/1       │      │
│ OR/ORI      │ 100         │ 0/1       │      │
│ SHL/SHLI    │ 101         │ 0/1       │      │
│ SHR/SHRI    │ 110         │ 0/1       │      │
└──────────────────────────────────────────────┘

Instruction bit layout:
┌───┬───────┬───────┐
│ T │ Opcode│ Imm   │
│1b │  3b   │  3b   │
└───┴───────┴───────┘
Bit [7]: 0 = R-type (third register), 1 = I-type (immediate)
Bits [6:4]: Operation (000=ADD, 001=SUB, etc.)
Bits [3:1]: Register field (or immediate for I-type)
```

---

## **Part 3: Immediate Operand Range and Encoding**

### **3.1 Sign Extension for Small Immediates**

#### **The Problem: Size Mismatch**

```
Scenario:
- Register data width: 32 bits
- Instruction immediate field: 5 bits (due to space constraints)
- Problem: How to use 5-bit immediate with 32-bit ALU?

5-bit immediate range (signed):
-16 to +15 (values: -10000 to +01111 in binary)

32-bit ALU expects:
Full 32-bit input (0x00000000 to 0xFFFFFFFF for unsigned)

Solution: SIGN EXTENSION
```

#### **Sign Extension Process**

```verilog
// Verilog: Sign extend 5-bit immediate to 32 bits

// Input: 5-bit immediate
logic [4:0] imm_5bit;

// Process: Check MSB (bit [4])
//   If MSB=1: number is negative → fill upper bits with 1s
//   If MSB=0: number is positive → fill upper bits with 0s

logic [31:0] imm_32bit;

// Method 1: Explicit replication
assign imm_32bit = {{27{imm_5bit[4]}}, imm_5bit};
//                  └─ 27 copies ─┘
//                  of sign bit

// Method 2: Using Verilog $signed
assign imm_32bit = {{27{imm_5bit[4]}}, imm_5bit[4:0]};

// Examples:
// imm_5bit = 5'b01111 (15 in decimal, positive)
// imm_32bit = 32'b00000000_00000000_00000000_00001111 = 0x0000000F
//
// imm_5bit = 5'b10000 (-16 in decimal, negative, two's complement)
// imm_32bit = 32'b11111111_11111111_11111111_11110000 = 0xFFFFFFF0
```

**Why Sign Extension?**

```
When we add: R1 + imm_5bit
We need both operands to be same width!

Example 1: Add R1 (32-bit value 100) + imm_5bit (5-bit value 5)
Without sign extension:
Trying to add: 32-bit 100 + 5-bit 5 (bits don't align!)

With sign extension:
100 in 32-bit = 0x00000064
5 in 32-bit = 0x00000005
Result: 0x00000069 = 105 (correct!)

Example 2: Add R1 (32-bit value 100) + imm_5bit (5-bit value -1)
-1 in 5-bit (two's complement): 5'b11111

Without sign extension (treating as unsigned 31):
32-bit 31 instead of -1!
Result would be 100 + 31 = 131 (WRONG!)

With sign extension:
-1 in 5-bit = 5'b11111
Sign extend: 32'b11111111_11111111_11111111_11111111 = 0xFFFFFFFF = -1
Result: 100 + (-1) = 99 (correct!)
```

### **3.2 Handling Large Constants**

#### **Problem: Limited Immediate Size**

```c
// C Code: Load a large constant
uint32_t large_value = 0x12345678;  // 32-bit value
register = large_value;

// With 5-bit immediate (range -16 to +15):
// We can load: -16 to +15 (only 32 possible values!)
// We CANNOT load 0x12345678 directly

// Real ISAs handle this with strategies:
```

#### **Strategy 1: Multiple Load Instructions**

```
// Example: Load 0x12345678 into R1 using 32-bit ISA

// RISC-V example (actual ISA uses slightly different names):
LUI  R1, 0x12345   # Load Upper Immediate: R1 = 0x12345 << 12
ADDI R1, R1, 0x678 # Add 0x678 to R1

Result in R1: (0x12345 << 12) | 0x678 = 0x12345678

// This works because:
// - LUI puts immediate in upper 20 bits
// - ADDI adds lower 12 bits
// - Together they form full 32-bit constant
```

#### **Strategy 2: Using R0 with Immediate Operations**

```
// If R0 is hardwired to 0, loading becomes simpler:

// Load 15 into R1:
ADD  R1, R0, 15      # R1 = 0 + 15 = 15

// Why this works:
// - R0 always contains 0 (hardwired in hardware)
// - ADD R1, R0, 15 uses immediate-type ADD
// - Result: R1 = 0 + 15

// Without hardwired R0:
// - Would need: LOAD R1, 15 (separate load instruction)
// - Uses more instruction space
```

#### **Strategy 3: Using Only Register Operations**

```
// Some ISAs (like original ARM) don't have immediate-operand instructions
// for all operations. Instead, they provide:

// 1. General-purpose register operations
ADD R1, R2, R3  # Must pre-load constants into registers

// 2. OR use specialized pseudo-instructions
// But these are macro-expanded by assembler into real instructions
```

---

## **Part 4: ALU Hardware Implementation with Immediates**

### **4.1 Modified ALU Architecture**

#### **Original ALU (Register-Only)**

```verilog
module alu_register_only #(parameter WIDTH = 32) (
    input  logic [WIDTH-1:0] operand_a,    // From register
    input  logic [WIDTH-1:0] operand_b,    // From register
    input  logic [3:0] operation,          // ADD, SUB, MUL, etc.
    output logic [WIDTH-1:0] result        // To register
);

always_comb begin
    case (operation)
        4'b0000: result = operand_a + operand_b;  // ADD
        4'b0001: result = operand_a - operand_b;  // SUB
        4'b0010: result = operand_a * operand_b;  // MUL
        // ... etc
        default: result = '0;
    endcase
end
endmodule
```

#### **Modified ALU (Register or Immediate)**

```verilog
module alu_with_immediate #(parameter WIDTH = 32) (
    input  logic [WIDTH-1:0] operand_rs,       // Source register from register file
    input  logic [WIDTH-1:0] operand_rt,       // Second source register
    input  logic [WIDTH-1:0] imm_extended,     // Immediate (already sign-extended)
    input  logic use_immediate,                // Control: 1 = use immediate, 0 = use register
    input  logic [3:0] operation,              // Operation selector
    output logic [WIDTH-1:0] result            // Result
);

// Intermediate signal: which value to use as second operand?
logic [WIDTH-1:0] operand_b;

// SELECT SECOND OPERAND
// If use_immediate=1, use sign-extended immediate
// Otherwise, use second source register
always_comb begin
    if (use_immediate) begin
        operand_b = imm_extended;  // Sign-extended immediate
    end else begin
        operand_b = operand_rt;    // Second register
    end
end

// ALU COMPUTATION (same as before)
always_comb begin
    case (operation)
        4'b0000: result = operand_rs + operand_b;
        4'b0001: result = operand_rs - operand_b;
        4'b0010: result = operand_rs * operand_b;
        4'b0011: result = operand_rs / operand_b;
        4'b0100: result = operand_rs & operand_b;
        4'b0101: result = operand_rs | operand_b;
        4'b0110: result = operand_rs << operand_b;
        4'b0111: result = operand_rs >> operand_b;
        default: result = '0;
    endcase
end

endmodule
```

**Key Changes:**

```
Old ALU:
  operand_a, operand_b → ALU computation → result

New ALU:
  operand_rs ──┐
               ├─→ Multiplexer ──→ operand_b ┐
  imm_extended─┤  (controlled by              ├─→ ALU ─→ result
  use_immediate   use_immediate)              │
               │                          operand_rs
  operand_rt ──┘

Architecture adds:
1. Input: use_immediate signal (control bit)
2. Input: imm_extended (sign-extended immediate)
3. Intermediate: Multiplexer to select operand_b
4. Rest stays the same: ALU computation logic identical

Complexity Addition: Minimal! Just a multiplexer and a few control signals
```

### **4.2 Sign Extension Module**

```verilog
// Standalone sign-extension module

module sign_extend #(
    parameter IMM_WIDTH = 5,       // Input width (small immediate)
    parameter OUTPUT_WIDTH = 32    // Output width (register width)
) (
    input  logic [IMM_WIDTH-1:0] imm_in,
    output logic [OUTPUT_WIDTH-1:0] imm_out
);

// Replicate sign bit (MSB) to fill upper bits
assign imm_out = {{(OUTPUT_WIDTH - IMM_WIDTH){imm_in[IMM_WIDTH-1]}}, imm_in};
//                └─ Repeat sign bit ─┘
//                OUTPUT_WIDTH - IMM_WIDTH times

// Example: IMM_WIDTH=5, OUTPUT_WIDTH=32
// Repeat (32-5)=27 copies of imm_in[4]
// Then append the 5-bit immediate

// imm_in = 5'b11111 (-1 in two's complement)
// imm_in[4] = 1 (MSB)
// imm_out = 27 ones + 5 ones = 32'hFFFFFFFF = -1 (correct!)
//
// imm_in = 5'b01111 (15 in two's complement)
// imm_in[4] = 0 (MSB)
// imm_out = 27 zeros + 5 bits (01111) = 32'h0000000F = 15 (correct!)

endmodule
```

---

## **Part 5: Practical Use Cases and R0 Optimization**

### **5.1 The Hardwired Zero Register**

#### **Common Architecture Decision**

```
Many modern RISC architectures (RISC-V, MIPS, ARM) use:

Register R0 (x0 in RISC-V) = Always 0 (hardwired)

Implementation in Hardware:
┌─────────────────────────────────────────────┐
│ Register File with R0 Hardwired to 0         │
├─────────────────────────────────────────────┤
│ During WRITE:                               │
│ if (wr_addr == 0) ignore write_data;        │
│ else registers[wr_addr] <= wr_data;         │
│                                             │
│ During READ:                                │
│ if (rd_addr == 0) rd_data = 32'b0;          │
│ else rd_data = registers[rd_addr];          │
└─────────────────────────────────────────────┘

Cost: One "unused" register (acceptable trade-off)
Benefit: Huge! Enables powerful addressing modes
```

#### **Advantages of Hardwired Zero**

```
1. INITIALIZATION (Most common use of zero)
   ADDI R1, R0, 0   # R1 = 0 + 0 = 0 (initialize R1 to zero)
   
   Without R0=0:
   Would need separate LOAD instruction or immediate move

2. MOVE INSTRUCTION (pseudo-operation)
   ADD  R1, R0, value  # R1 = 0 + value = value
   
   Without R0:
   Would need explicit MOVE instruction (wastes opcode space)

3. COMPARISON AGAINST ZERO
   ADD  R2, R1, 0     # R2 = R1 + 0 = R1
   BEQ  R2, R0, label # Branch if R2 == 0
   
   Natural way to check if value is zero!

4. CONSTANT LOADING (with multi-step)
   ADDI R1, R0, 15      # R1 = 0 + 15 = 15
   
   Without R0:
   Need a different source register (wastes register!)
```

### **5.2 Practical Programming Examples**

#### **Example 1: Zero Initialization**

```c
// C Code
int x = 0;

// Compiled to:
ADDI R1, R0, 0   # R1 ← R0 + 0 = 0 + 0 = 0

// Why not simpler instructions? Because:
// - Instruction encoding must be uniform
// - Every instruction follows same format
// - Even "load zero" must use ADDI format
// - Machine still benefits from immediate encoding
```

#### **Example 2: Increment Operation**

```c
// C Code
int count = ...;
count++;

// Compiled to:
ADDI R2, R1, 1   # R2 ← R1 + 1 (where R1 holds count)

// Efficiency: Single instruction instead of:
// LOAD R3, 1          # Load 1 into R3
// ADD  R2, R1, R3     # R2 ← R1 + R3
```

#### **Example 3: Bit Masking**

```c
// C Code - Extract bottom 4 bits
uint8_t bottom_bits = value & 0x0F;

// Compiled to:
ANDI R2, R1, 0x0F   # R2 ← R1 & 0x0F

// Without immediate:
LOAD R3, 0x0F       # Load mask into R3
AND  R2, R1, R3     # R2 ← R1 & R3
```

#### **Example 4: Extract Bits with Shift**

```c
// C Code - Get bits [7:4] from 32-bit value
uint8_t mid_bits = (value >> 4) & 0x0F;

// Compiled to:
SHRI R2, R1, 4      # R2 ← R1 >> 4
ANDI R2, R2, 0x0F   # R2 ← R2 & 0x0F

// Why separate instructions?
// - Shift by immediate
// - And with immediate
// - Each instruction has separate immediate encoding

// Without immediate:
LOAD R3, 4          # Load 4 into R3
SHR  R2, R1, R3     # R2 ← R1 >> R3
LOAD R4, 0x0F       # Load 0x0F into R4
AND  R2, R2, R4     # R2 ← R2 & R4
// (3+ instructions vs 2 with immediates!)
```

#### **Example 5: Large Constant Loading**

```c
// C Code - Load 32-bit constant
uint32_t large = 0x3E8;  // 1000 in decimal

// With 5-bit immediate (range -16 to +15):
// Cannot load 0x3E8 directly!

// Solution: Multi-step process
ADDI R1, R0, 15      # Load 15
SHLI R1, R1, 6       # R1 = 15 << 6 = 960
ADDI R1, R1, 8       # R1 = 960 + 8 = 968
// Still can't get exactly 1000! (would need more steps)

// Better ISAs have larger immediates:
ADDI R1, R0, 1000    # Load 1000 (if 11-bit immediate available)

// Or use multi-instruction loader:
LUI  R1, 0x0         # Load Upper Immediate
ADDI R1, R1, 0x3E8   # Add lower bits
```

### **5.3 Instruction Encoding Trade-off**

```
Current Encoding (hypothetical):
┌────┬──────┬──────┬──────┐
│ Op │  Rd  │  Rs  │ Imm  │
│ 2b │  3b  │  3b  │  5b  │
└────┴──────┴──────┴──────┘
Total: 16 bits

Options to extend immediate range:

Option A: 8-bit immediate (lose 3 register bits)
┌────┬──────┬──────┬────────┐
│ Op │ Rd   │ Rs   │  Imm   │
│ 2b │ 3b   │ 3b   │  8b    │
└────┴──────┴──────┴────────┘
- Immediate: -128 to +127 (much better!)
- But lose register addressing (can't reach all 8 registers)
- Result: Fewer usable registers per instruction
- Trade-off: Not worth it for this design

Option B: Larger instruction format (32-bit)
┌────┬──────┬──────┬───────────┐
│ Op │ Rd   │ Rs   │    Imm    │
│ 4b │ 5b   │ 5b   │   14b     │
└────┴──────┴──────┴───────────┘
- Support 32 registers with 5 bits each
- Support larger immediates: -8192 to +8191
- Instruction size: 32 bits (vs 16 bits)
- Real ISAs (RISC-V, MIPS, ARM): Use this!

Current Trade-off Analysis:
┌─────────────────────────────────────────┐
│ 5-bit immediate:                        │
│ ✓ Small instructions (16-bit)           │
│ ✓ Fit in cache efficiently              │
│ ✗ Limited immediate range               │
│ ✗ Limited register set (8 total)        │
│                                         │
│ 14-bit immediate (32-bit ISA):         │
│ ✓ Large immediate range                 │
│ ✓ More registers (32 total)             │
│ ✗ Larger instructions (uses cache)      │
│ ✗ More complex encoding                 │
└─────────────────────────────────────────┘

Real ISAs typically choose 32-bit instructions with:
- 3-5 bits for opcode
- 5 bits per register (32 registers)
- 12-16 bits for immediate
```

---

## **Part 6: Hardware Implementation Summary**

### **6.1 Complete Data Path with Immediates**

```
Data Flow:

┌──────────────────────────────────────────────────────┐
│                  Register File                        │
│  Reads: Rs from instruction                          │
│  Output: rs_value (32 bits)                          │
└───────────────────┬──────────────────────────────────┘
                    │
                    ▼
    rs_value (32 bits) → ALU input_a
                    
┌──────────────────────────────────────────────────────┐
│           Sign Extension Module                       │
│  Input: imm (5 bits from instruction)                │
│  Output: imm_extended (32 bits)                      │
└───────────────────┬──────────────────────────────────┘
                    │
                    ▼
    imm_extended (32 bits) ──┐
                             │
                        ┌────▼─────┐
    rt_value ──────────┤ Mux 2:1   │
    use_immediate ─────┤ Control   ├──▶ ALU input_b
                        └──────────┘
                             │
                             ▼
    ALU operation: input_a (op) input_b → result

    result → Register File write port
                   ↓
             Update register[rd]
```

### **6.2 Enhanced ALU Module (Complete)**

```verilog
module alu_enhanced #(
    parameter WIDTH = 32,
    parameter IMM_WIDTH = 5
) (
    // Register inputs
    input  logic [WIDTH-1:0] rs_data,      // Source 1
    input  logic [WIDTH-1:0] rt_data,      // Source 2 (if register-based)
    
    // Immediate input
    input  logic [IMM_WIDTH-1:0] imm_small,// Raw 5-bit immediate
    
    // Control signals
    input  logic use_immediate,            // 1=use imm, 0=use rt_data
    input  logic [3:0] operation,          // Operation selector
    
    // Output
    output logic [WIDTH-1:0] result
);

// STAGE 1: Sign extension of immediate
logic [WIDTH-1:0] imm_extended;
assign imm_extended = {{(WIDTH - IMM_WIDTH){imm_small[IMM_WIDTH-1]}}, imm_small};

// STAGE 2: Operand selection (multiplexer)
logic [WIDTH-1:0] operand_b;
assign operand_b = use_immediate ? imm_extended : rt_data;

// STAGE 3: ALU operations (same as before)
always_comb begin
    case (operation)
        4'b0000: result = rs_data + operand_b;      // ADD
        4'b0001: result = rs_data - operand_b;      // SUB
        4'b0010: result = rs_data * operand_b;      // MUL
        4'b0011: result = rs_data / operand_b;      // DIV
        4'b0100: result = rs_data & operand_b;      // AND
        4'b0101: result = rs_data | operand_b;      // OR
        4'b0110: result = rs_data << operand_b;     // SHL
        4'b0111: result = rs_data >> operand_b;     // SHR
        default: result = '0;
    endcase
end

endmodule
```

---

## **Part 7: Summary and Design Philosophy**

### **7.1 Key Benefits of Immediate Operands**

```
╔═══════════════════════════════════════════════════════╗
║          BENEFITS OF IMMEDIATE OPERANDS              ║
╠═══════════════════════════════════════════════════════╣
║ 1. Reduced Register Pressure                         ║
║    - Don't waste registers storing constants         ║
║    - More registers available for real data         ║
║                                                     ║
║ 2. Fewer Instructions                               ║
║    - ADDI R1, R2, 5 (1 instruction)                 ║
║    - vs LOAD R3,5 + ADD R1, R2, R3 (2 instructions)║
║                                                     ║
║ 3. Code Density                                     ║
║    - Fewer instructions → smaller code size        ║
║    - Better cache utilization                      ║
║    - Faster program loading                        ║
║                                                     ║
║ 4. Performance                                      ║
║    - Fewer instructions → faster execution         ║
║    - Less memory bandwidth needed                  ║
║    - Pipeline more efficient                       ║
║                                                     ║
║ 5. Common Operations                                ║
║    - Zero initialization                           ║
║    - Incrementing/decrementing                     ║
║    - Bit masking and shifting                      ║
║    - All optimized with immediates                 ║
╚═══════════════════════════════════════════════════════╝
```

### **7.2 Architectural Design Trade-offs**

```
Choice 1: Register-Only ISA (no immediates)
─────────────────────────────────────────
Pro:
- Simpler instruction encoding
- Uniform instruction format
- Simpler ALU hardware

Con:
- Must pre-load all constants into registers
- Wastes register file with constants
- More instructions for simple operations
- Worse code density

Example: LOAD-STORE architectures from 1970s-1980s
```

```
Choice 2: Register + Immediate ISA (our approach)
─────────────────────────────────────────────────
Pro:
- Efficient constant operations
- Better code density
- Reduced register pressure
- Hardware complexity is minimal (just a mux)

Con:
- Immediate size limited by instruction width
- Slight instruction encoding complexity

Example: Modern RISC ISAs (RISC-V, MIPS, ARM, x86)
```

### **7.3 Design Philosophy**

```
Immediate operands reflect fundamental processor design principle:

"Optimize for the Common Case"

Statistical Fact:
- Zero is the most common operand (>30% of instructions)
- Small integers (1, 2, 4) are very common
- Bit masks (0xFF, 0xF0) are essential for embedded systems

Solution:
- Make immediate operations efficient
- Support sign extension for negative numbers
- Use architectural tricks (hardwired R0=0) to maximize utility
- Trade small amount of instruction encoding space for big wins
  in code density and register efficiency

Result:
- Better code generation by compilers
- Smaller binaries
- Faster program execution
- More efficient hardware

This is why almost ALL modern ISAs (RISC-V, ARM, x86, MIPS, PowerPC)
include immediate-operand instructions.
```

---

## **Part 8: Extension Ideas (For Future Lectures)**

```
Possible Extensions:

1. Large Constant Loading
   - Load Upper Immediate (LUI) instruction
   - Combines with ADDI for 32-bit constants

2. PC-Relative Addressing
   - Use immediate as offset from Program Counter
   - Essential for branches and jumps

3. Addressing Modes
   - Base + Immediate (for memory access)
   - Example: Load from address [R1 + 4]

4. Conditional Execution
   - Test and branch based on ALU flags
   - Use immediate for branch offset

5. Variable-Width Instructions
   - Some instructions 16-bit, others 32-bit
   - Trade-off between code density and expressiveness
```

---

## **Comprehensive Summary Table**

| Aspect | Register-Only | Register + Immediate |
|--------|--------------|----------------------|
| **Operations** | R1 OP R2 → R3 | R1 OP Constant → R3 |
| **Const Handling** | Pre-load into register | Direct in instruction |
| **Register Usage** | All 8 registers for data | Fewer registers for constants |
| **Code Density** | Poor (extra LOAD instructions) | Good (immediate encoded) |
| **Hardware** | Simple ALU | ALU + Mux + Sign Extension |
| **Common Use** | General computation | Initialize, increment, mask |
| **Zero Handling** | Wastes register | Can hardwire R0=0 |
| **Examples** | RISC-I (1981) | RISC-V, ARM, x86 (modern) |

