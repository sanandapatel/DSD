# Digital System Design (DSD) - Complete Question Bank with Detailed Explanations

## Overview
This document contains all DSD exam questions extracted from both exam papers with comprehensive, child-friendly explanations and analogies.

---

## SECTION 1: TRUE/FALSE CONCEPT QUESTIONS

### Question 115: Modularity and Complexity
**Question:** Modularity increases complexity by breaking the design into smaller components, making maintenance and verification more difficult.

**Correct Answer:** FALSE

**Detailed Explanation:**

Imagine building a large LEGO castle. You have two ways to approach it:

**Approach 1 (No Modularity):** Try to build the entire castle as one huge block without breaking it down. If something goes wrong, you have to throw away the entire thing and start over. Fixing a problem means understanding how every single piece relates to every other piece.

**Approach 2 (With Modularity):** Break the castle into sections—the walls, the towers, the gates, the roof. Build each section separately. If the tower has a problem, you only fix that tower without touching the walls. Each section is simpler to understand and test independently.

**In Digital System Design:**
- **Without modularity:** One massive 10,000-line hardware code is hard to debug. A bug could be anywhere.
- **With modularity:** Split into smaller blocks (ALU, Controller, Memory). Each block has 500 lines. Much easier to find bugs.

**Real Example:**
Think of a computer:
- CPU module (handles calculations)
- Memory module (stores data)
- I/O module (handles input/output)

Each can be tested separately before combining them. This makes everything **simpler**, not complex.

---

### Question 116: Bitwise Right Shift for Division
**Question:** To efficiently divide an unsigned number N by 8 using bitwise operations, shift N right by log₂(8) positions.

**Correct Answer:** TRUE

**Detailed Explanation:**

**The Mathematical Relationship:**

When you shift a binary number right by n positions, you're dividing by 2ⁿ.

- log₂(8) = log₂(2³) = 3
- So shift right by 3 positions = divide by 2³ = divide by 8

**Concrete Example:**
```
Let's divide 64 by 8

Binary: 64 = 0b01000000 (8 bits)
        
Shifting right by 3 positions:
0b01000000 >> 3 = 0b00001000 = 8 (which is 64 ÷ 8 ✓)
```

**Why Does This Work?**

Think of binary positions like place values:
```
Position:  7  6  5  4  3  2  1  0
Binary:    0  1  0  0  0  0  0  0
Value:    128 64 32 16  8  4  2  1
```

When you shift right by 3, every digit moves 3 positions right:
```
Position:  7  6  5  4  3  2  1  0
After:     0  0  0  0  1  0  0  0  = 8
```

The 1 that was at position 6 (value 64) is now at position 3 (value 8). That's exactly 64 ÷ 8!

**More Examples:**
- 32 >> 3 = 4 (32 ÷ 8 = 4) ✓
- 16 >> 3 = 2 (16 ÷ 8 = 2) ✓
- 256 >> 3 = 32 (256 ÷ 8 = 32) ✓

**Why Is This Efficient?**
A shift operation is one CPU instruction and runs in 1 clock cycle, while a division instruction might take many cycles. So this is fast!

---

### Question 117: Parallel Multiplier with Ripple-Carry
**Question:** A parallel multiplier with a ripple-carry structure reduces latency.

**Correct Answer:** FALSE

**Detailed Explanation:**

**Understanding Latency:**
Latency is the time delay from when you start something until you get the result.

**Parallel vs. Sequential Multiplier:**

**Sequential Multiplier:**
- Works step-by-step
- Multiply 5 × 3 in 8 steps
- Each step processes one bit of the multiplier
- Takes 8 clock cycles (if 8-bit multiplier)
- But each step is simple, so each clock cycle is fast
- Total time = 8 × (fast clock cycle) = moderate latency

**Parallel Multiplier (with Ripple-Carry):**
- Tries to do all 8 bits at once
- BUT: Ripple-carry means results flow sequentially through the circuit
- Like dominoes falling: the rightmost result must finish before the next position can start
- Takes 1 clock cycle, BUT the clock cycle is very slow (waiting for ripple)
- Total time = 1 × (very slow clock cycle) = high latency

**Analogy:**
Imagine a relay race where 8 people need to hand off a baton:

**Sequential approach:** Each person runs with the baton one at a time, 8 seconds per person = 8 seconds total

**Ripple-carry parallel approach:** All 8 people run at once, BUT each can only start after the previous person finishes their portion. So person 2 waits for person 1, person 3 waits for person 2, etc. They end up running sequentially anyway, taking 8 seconds total, but now they're all tired!

**Solution:**
Use a **tree-based multiplier** with carry-lookahead logic. This truly parallelizes everything and reduces latency.

**Conclusion:** Ripple-carry is actually bad for latency. The statement is FALSE.

---

### Question 118: SystemVerilog Immediate Assertion
**Question:** If the immediate assertion in SystemVerilog fails, the simulation will continue while displaying an error message.

**Correct Answer:** TRUE

**Detailed Explanation:**

**Understanding Assertions:**
Assertions are like safety checks in code. They monitor whether things are working correctly.

**Two Types of Assertions:**

**1. Immediate Assertion:**
- Checks immediately when the line is executed
- Like a smoke detector that alerts you to smoke
- If it fails: Shows an error message but **continues** running
- Execution: **Does NOT stop**
- Use case: Non-critical checks

**Example:**
```verilog
always @(posedge clk) begin
    assert (value != 0) else $display("ERROR: Value is zero");
    // Code continues here even if assertion failed
end
```

**2. Concurrent Assertion:**
- Monitors conditions over time
- Like a security alarm that stops everything
- If it fails: **Stops simulation immediately**
- Execution: **HALTS**
- Use case: Critical design violations

**Example:**
```verilog
assert property (if (reset) @(posedge clk) value == 0)
// If this fails, simulation stops
```

**Analogy:**
- **Immediate assertion:** Like a warning light in a car that says "Check engine" but the car still runs
- **Concurrent assertion:** Like a seatbelt that locks and stops you from moving forward

**Why Different Behavior?**

Immediate assertions are for debugging and monitoring—you want to know there's an issue but keep testing.

Concurrent assertions are for design correctness—if something is fundamentally wrong, the entire simulation is invalid.

**Answer:** TRUE - Immediate assertions continue execution while displaying error messages.

---

### Question 119: 6T SRAM Memory Calculation
**Question:** A 64-byte memory implemented using 6T SRAM requires 384 transistors.

**Correct Answer:** FALSE (Correct answer should be 3,072 transistors)

**Detailed Explanation:**

**Understanding 6T SRAM:**
6T SRAM means each memory cell uses 6 transistors.

**Calculation Steps:**

**Step 1: Convert bytes to bits**
```
64 bytes = 64 × 8 bits = 512 bits
```

Each byte has 8 bits, so 64 bytes = 512 individual bits that need to be stored.

**Step 2: Calculate transistors needed**
```
512 bits × 6 transistors/bit = 3,072 transistors
```

Since each bit needs 6 transistors, we multiply.

**Step 3: Verify the given answer**
The question claims 384 transistors. Let's check:
```
384 transistors ÷ 6 transistors/bit = 64 bits = 8 bytes
```

384 transistors would only store 8 bytes, not 64 bytes!

**Why 6 Transistors Per Cell?**

A 6T SRAM cell consists of:
- 2 transistors forming a **latch** (memory storage)
- 2 transistors for the **bit lines** (data access)
- 2 transistors for the **word line** (row selection)

Together: 6 transistors per bit

**Extended Example:**

For different memory sizes:
- 32 bytes = 256 bits → 256 × 6 = 1,536 transistors
- 64 bytes = 512 bits → 512 × 6 = 3,072 transistors ✓
- 128 bits = 1 byte → 128 × 6 = 768 transistors

**Answer:** FALSE - It actually requires 3,072 transistors, not 384.

---

### Question 120: FIFO Full Flag Logic
**Question:** In a FIFO design, when the read and write pointers are equal, the "Full" status flag is asserted.

**Correct Answer:** FALSE

**Detailed Explanation:**

**Understanding FIFO Pointers:**

A FIFO (First-In-First-Out) is like a queue at a ticket counter:
- **Write pointer:** Points to where new data will be written (next empty spot)
- **Read pointer:** Points to where old data will be read from (next full spot)

**Circular Buffer Concept:**
Imagine a circular array with positions 0, 1, 2, 3, then it wraps back to 0.

**Three Pointer Scenarios:**

**Scenario 1: Both Equal = EMPTY**
```
Positions:  0  1  2  3
Write/Read: ↓  
            (both at same position)

Status: EMPTY (no data between them)
```

**Scenario 2: Write Ahead of Read = Data Inside**
```
Positions:  0  1  2  3
Read:       ↓
Write:           ↓

Data is between read and write pointers
Status: PARTIAL (has data)
```

**Scenario 3: Write Catches Up = FULL**
```
Positions:  0  1  2  3
Write:      ↓
Read:       ↓ (one position behind write)

When write is one position BEHIND read in a circular buffer
Status: FULL (every position filled)
```

**The Logic:**
- **Empty condition:** Read pointer == Write pointer
- **Full condition:** Write pointer is **exactly one position behind** the Read pointer (in circular wrap-around)

**Why Not "Equal = Full"?**

If both are equal in a circular buffer, there's actually NO data because there's no "between" two identical positions. 

Think of it like this: If you're standing at the same spot as someone else, you're not between anything!

**Answer:** FALSE - When pointers are equal, the FIFO is EMPTY, not FULL. FULL occurs when the write pointer is one position behind the read pointer.

---

### Question 121: Register Fan-Out and Delay
**Question:** In a register bank, increasing fan-out decreases the output delay of a register.

**Correct Answer:** FALSE

**Detailed Explanation:**

**Understanding Fan-Out:**

Fan-out is the number of different components a register output must drive/supply.

**Analogy - Water Pressure:**

Imagine a water pump connected to pipes:
- **Fan-out = 1:** Pump supplies water to 1 pipe → high water pressure
- **Fan-out = 5:** Pump supplies water to 5 pipes → pressure divides among pipes
- **Fan-out = 10:** Pump supplies water to 10 pipes → pressure is very weak

**The Physics:**

In electronics, a register output has a finite amount of electrical current (like water flow):
- More fan-out = spreading that current among more destinations
- Each destination gets less current
- Weaker current = slower voltage rise time = **longer delay**

**Detailed Breakdown:**

```
Maximum Current = I_max (fixed property of register)

With fan-out = 1:
  Current per destination = I_max / 1 = I_max
  → Voltage rises quickly → LOW DELAY

With fan-out = 5:
  Current per destination = I_max / 5 = 0.2 × I_max
  → Voltage rises slowly → HIGHER DELAY

With fan-out = 10:
  Current per destination = I_max / 10 = 0.1 × I_max
  → Voltage rises very slowly → EVEN HIGHER DELAY
```

**Formula Impact:**
Delay = Capacitance × Resistance

Higher fan-out increases equivalent capacitance (more loads), which increases delay.

**Power Consumption Impact:**
Higher fan-out also means:
- More current flowing
- More heat generated
- **Higher power consumption**

**Answer:** FALSE - Increasing fan-out **increases** output delay and power consumption. The statement says it "decreases" delay, which is incorrect.

---

### Question 122: CPU Program Counter Reset
**Question:** When the CPU is reset, the program counter (PC) goes to a defined start point.

**Correct Answer:** TRUE

**Detailed Explanation:**

**Understanding the Program Counter (PC):**

The PC is like a bookmark in a book that tells you which line to read next:
- It holds the memory address of the next instruction
- After executing each instruction, it increments
- It's the CPU's way of knowing where it is in the program

**Reset Scenario:**

When a CPU powers on or is reset:
- The PC could be pointing anywhere (random data in the register)
- The CPU doesn't know what instructions to fetch
- **Solution:** Force PC to a fixed, known address called the **Reset Vector**

**Typical Reset Vector Addresses:**
- x86 processors: 0xFFFF0000
- ARM processors: 0x00000000
- MIPS processors: 0x80000000
- FPGA designs: Often 0x00000000

**Why This Is Important:**

Think of it like a book:
- When you open a new book, you always start at page 1
- You don't open to a random page
- Same with CPU: Always start at a defined address (e.g., address 0)

**Boot Process:**
```
1. CPU powers on
2. Reset signal is asserted
3. PC is forced to reset vector (e.g., 0x00000000)
4. Next clock cycle: CPU fetches instruction from address 0x00000000
5. Bootloader or BIOS code executes
6. System initialization proceeds
```

**Example Code:**
```verilog
always @(posedge reset) begin
    PC <= 32'h00000000;  // Reset to address 0
end
```

**Answer:** TRUE - When the CPU is reset, the PC always goes to a defined start point (the reset vector). This ensures consistent and predictable behavior.

---

### Question 123: Branch Condition Logic
**Question:** If the branch condition is false, the PC advances normally to execute the next instruction in sequence.

**Correct Answer:** TRUE

**Detailed Explanation:**

**Understanding Branch Conditions:**

A branch instruction is like a fork in a road:
- **Left fork:** Execute some code (branch taken - condition is TRUE)
- **Right fork:** Continue straight, execute next instruction (branch not taken - condition is FALSE)

**How Branch Works:**

**When Condition is TRUE:**
```
BEQ r1, r0, target_address
    ↓
r1 == r0? YES!
    ↓
PC ← target_address (JUMP to target)
    ↓
Execute instruction at target_address next
```

**When Condition is FALSE:**
```
BEQ r1, r0, target_address
    ↓
r1 == r0? NO!
    ↓
PC ← PC + 4 (continue sequentially)
    ↓
Execute next instruction in sequence
```

**Detailed Example:**

```
Address    Instruction        PC Value After Execution
100        ADD r1, r1, 1      104 (added 4 bytes)
104        BEQ r1, r0, done   ? (depends on condition)
108        ADD r2, r2, 1      ?
112        done: SUB r3, r3, 1 ?

Scenario A: r1 == 0 (condition TRUE)
  → PC becomes address of "done" = 112
  → Next instruction: SUB r3, r3, 1

Scenario B: r1 != 0 (condition FALSE)
  → PC becomes 108 (104 + 4, sequential)
  → Next instruction: ADD r2, r2, 1
```

**Multiple Branch Types:**

```
BEQ  r1, r0, target  // Branch if Equal
BNE  r1, r0, target  // Branch if Not Equal
BLT  r1, r0, target  // Branch if Less Than
BGT  r1, r0, target  // Branch if Greater Than

In all cases:
- Condition TRUE → Jump to target
- Condition FALSE → Continue sequentially (PC += 4)
```

**Answer:** TRUE - When a branch condition is false, the PC advances normally to the next instruction in sequence (typically PC += 4 bytes). The branch is "not taken."

---

### Question 124: Two's Complement Representation
**Question:** In a CPU with 5-bit immediate values using two's complement representation, the immediate value 5'b11001 corresponds to -7 in decimal.

**Correct Answer:** TRUE

**Detailed Explanation:**

**Understanding Two's Complement:**

Two's complement is how computers represent negative numbers in binary.

**5-Bit Binary Range:**

With 5 bits, we can represent:
- **Positive numbers:** 0 to 15 (unsigned)
- **Signed numbers:** -16 to +15 (two's complement)

The leftmost bit is the **sign bit**:
- 0 = positive number
- 1 = negative number

**Converting 5'b11001 to Decimal:**

**Step 1: Recognize it's negative**
```
5'b11001 starts with 1 → It's negative
```

**Step 2: Invert all bits**
```
Original:  1 1 0 0 1
Inverted:  0 0 1 1 0
```

**Step 3: Add 1 to the inverted value**
```
Inverted:  0 0 1 1 0 = 6 in decimal
Add 1:     0 0 1 1 0 + 1 = 0 0 1 1 1 = 7 in decimal
```

**Step 4: Add negative sign**
```
Result = -7
```

**Verification:**
```
5'b11001 = -7? Let's add 7 to it:
  11001  (-7)
+  00111 (+7)
---------
  100000 = overflow, but bottom 5 bits = 00000 = 0 ✓

This confirms 11001 + 00111 = 0, so 11001 = -7
```

**Complete 5-Bit Two's Complement Table:**

```
Binary    Decimal
-----     -------
01111     +15
01110     +14
...
00001     +1
00000     0
11111     -1
11110     -2
...
11001     -7  ← Our question
...
10000     -16
```

**Why Use Two's Complement?**

Alternative methods:
- **Sign-Magnitude:** 10001 could mean -1 or +1 (ambiguous)
- **One's Complement:** Two ways to represent zero (wasteful)
- **Two's Complement:** One representation of zero, clean arithmetic

**Key Property:**
In two's complement, to negate a number:
1. Invert all bits
2. Add 1

This makes subtraction hardware simple!

**Answer:** TRUE - The binary value 5'b11001 in two's complement representation equals -7 in decimal.

---

## SECTION 2: COMPUTATIONAL QUESTIONS (2 MARKS)

### Question 125: Sequential Multiplier Throughput
**Question:** In a synchronous system operating at 200MHz, if a sequential multiplier requires 8 clock cycles to complete one multiplication, what is its throughput in operations per second?

**Correct Answer:** 25 million operations per second

**Detailed Explanation:**

**Understanding Throughput:**

Throughput = How many operations can be completed per second

**Given Information:**
- Clock frequency = 200 MHz = 200,000,000 cycles per second
- Cycles per multiplication = 8 cycles

**Calculation Step 1: Time per multiplication**

```
Time per operation = Number of cycles / Clock frequency
                   = 8 cycles / 200 MHz
                   = 8 / (200 × 10^6) seconds
                   = 4 × 10^-8 seconds
                   = 40 nanoseconds per operation
```

**Calculation Step 2: Operations per second**

```
Throughput = Clock frequency / Cycles per operation
           = 200 MHz / 8
           = 200 × 10^6 / 8
           = 25 × 10^6
           = 25 million operations per second
```

**Intuitive Understanding:**

Think of a factory:
- Production rate = 200 items per second (clock frequency)
- Each item takes 8 production steps (cycles per operation)
- You can complete 200 ÷ 8 = 25 items per second

**Answer Verification:**

Let's check other options:
- 20,000 ops/sec: Way too low (200M ÷ 20k = 10,000 cycles—wrong!)
- 40,000 ops/sec: Still too low (200M ÷ 40k = 5,000 cycles—wrong!)
- 8 million ops/sec: Too low (200M ÷ 8M = 25 cycles—wrong!)
- **25 million ops/sec: 200M ÷ 25M = 8 cycles ✓ CORRECT**

**Answer:** 25 million operations per second

---

### Question 126: Memory Module Configuration
**Question:** If a memory module is configured as 4 × 16, how many bits does each memory location store?

**Correct Answer:** 16 bits

**Detailed Explanation:**

**Understanding Memory Configuration Notation:**

Memory notation "M × N" means:
- **M** = Number of memory locations (addresses)
- **N** = Number of bits stored at each location (word width)

**Notation Breakdown: 4 × 16**

```
4 × 16 memory:
  4    = 4 different addresses (can store data at 4 locations)
  16   = Each location stores 16 bits
```

**Visual Representation:**

```
Address    Data (16 bits)
-------    ---------------
   0       [0000000000000000]
   1       [1111111111111111]
   2       [1010101010101010]
   3       [1100110011001100]

Total capacity = 4 addresses × 16 bits/address = 64 bits total
```

**Analogy:**

Think of a parking lot:
- **4 × 16:** 4 parking spaces, each can hold 16 cars
- 4 = parking spaces (addresses)
- 16 = capacity per space (bits per word)

The question asks: **How many bits per space?** Answer: 16 bits

**Other Examples:**

- 8 × 32 memory: 8 addresses, 32 bits each
- 256 × 8 memory: 256 addresses, 8 bits each
- 1024 × 64 memory: 1024 addresses, 64 bits each

**Answer:** 16 bits per memory location

---

### Question 127: ALU Operation Encoding Bits
**Question:** How many bits are required to select one of the given basic operations (ADD, SUB, MUL, DIV) in a simple ALU?

**Correct Answer:** 2 bits

**Detailed Explanation:**

**Understanding Bit Encoding:**

To select between N different options, you need at least log₂(N) bits.

**Calculation:**

Number of operations = 4 (ADD, SUB, MUL, DIV)

```
bits required = log₂(4) = log₂(2²) = 2 bits
```

**Why 2 Bits?**

With 2 bits, you can represent 2² = 4 different values:

```
Bit Pattern    Decimal    Operation
-----------    -------    ---------
    00           0        ADD
    01           1        SUB
    10           2        MUL
    11           3        DIV
```

**Why Not 1 Bit?**

With 1 bit:
```
Bit Pattern    Decimal
-----------    -------
    0            0
    1            1
```

Only 2¹ = 2 values possible, but we have 4 operations. **Not enough!**

**Why Not 3 Bits?**

With 3 bits:
```
Bit Pattern    Decimal
-----------    -------
    000          0
    001          1
    010          2
    011          3
    100          4
    101          5
    110          6
    111          7
```

3 bits can represent 2³ = 8 values, but we only need 4. **Wasteful but works**

However, we ask for the **minimum** bits required, which is 2.

**General Formula:**

```
bits required = ⌈log₂(n operations)⌉

Examples:
- 2 operations (ADD, SUB):     log₂(2) = 1 bit
- 4 operations (ADD, SUB, MUL, DIV): log₂(4) = 2 bits
- 8 operations:                log₂(8) = 3 bits
- 16 operations:               log₂(16) = 4 bits
```

**Answer:** 2 bits

---

### Question 128: Instruction Binary Encoding
**Question:** Consider the instruction format of the system is {destination_address, source_address1, OP, source_address2}. 

Given:
- OP codes: 000→ADD, 001→SUB, 010→MUL, 011→DIV, 100→AND, 101→OR, 110→NOT
- Register codes: 000→R0, 001→R1, 010→R2, 011→R3, 100→R4, 101→R5, 110→R6, 111→R7

What is the binary encoding of the instruction R3 = R1 * R2?

**Correct Answer:** 011_001_010_010

**Detailed Explanation:**

**Understanding Instruction Encoding:**

Instructions are encoded as binary patterns where each group of bits represents a different part of the instruction.

**Instruction Format Breakdown:**

```
{destination_address, source_address1, OP, source_address2}
```

Each field uses a certain number of bits:
- destination_address: 3 bits (for 8 registers)
- source_address1: 3 bits
- OP: 3 bits (for 7 operations)
- source_address2: 3 bits

**Decoding the Instruction: R3 = R1 * R2**

This means: Multiply R1 by R2 and store result in R3

**Step 1: Identify destination**
```
Destination register = R3
R3 code = 011
```

**Step 2: Identify source1**
```
Source1 register = R1
R1 code = 001
```

**Step 3: Identify operation**
```
Operation = MUL (multiply)
MUL code = 010
```

**Step 4: Identify source2**
```
Source2 register = R2
R2 code = 010
```

**Step 5: Combine in order**
```
{destination, source1, OP, source2}
= {011, 001, 010, 010}
= 011_001_010_010
```

**Verification with Alternative Representation:**

```
011_001_010_010
 |   |   |   |
 R3  R1 MUL  R2

Execute: R3 ← R1 * R2 ✓ Correct!
```

**Checking Other Options:**

```
Option A: 011_010_010_010
  Destination: R3 ✓
  Source1: R2 (wrong, should be R1)
  OP: MUL ✓
  Source2: R2 ✓
  Result: R3 = R2 * R2 (INCORRECT)

Option B: 010_011_010_001
  Destination: R2 (wrong, should be R3)
  Source1: R3 (wrong)
  OP: MUL ✓
  Source2: R1 (wrong)
  (INCORRECT)

Option C: 010_001_010_010
  Destination: R2 (wrong, should be R3)
  Source1: R1 ✓
  OP: MUL ✓
  Source2: R2 ✓
  Result: R2 = R1 * R2 (INCORRECT)

Option D (CORRECT): 011_001_010_010
  Destination: R3 ✓
  Source1: R1 ✓
  OP: MUL ✓
  Source2: R2 ✓
  Result: R3 = R1 * R2 ✓
```

**Answer:** 011_001_010_010

---

### Question 129: Register R0 Hardwired to Zero
**Question:** Why is register r0 often hardwired to 0 in many architectures?

**Correct Answer:** To simplify certain operations like initialization

**Detailed Explanation:**

**Understanding Register Hardwiring:**

"Hardwired" means the register is physically designed to always contain a specific value (in this case, 0) and cannot be changed.

**Why Hardwire R0 to Zero?**

**Reason 1: Simplifying Initialization**

Instead of:
```assembly
ADDI r0, r0, 0    // Initialize r0 to 0
ADD r1, r0, r0    // r1 = 0 + 0 = 0
```

You can simply:
```assembly
ADD r1, r0, r0    // r1 = 0 + 0 = 0
// r0 is always 0, no initialization needed
```

**Reason 2: Clearing Registers**

```assembly
ADD r2, r0, r0    // r2 = 0 (clear r2)
SUB r3, r3, r0    // r3 = r3 - 0 = r3 (unchanged)
```

**Reason 3: Constants in Code**

Often you need 0 in calculations:
```assembly
ADDI r1, r0, 100  // r1 = 100 + 0 (r0 provides the 0)
```

**Reason 4: Efficient Hardware Design**

Benefits:
- Saves one register (no wasted register holding 0)
- Simplifies control logic
- Common initialization value (0) is always available
- Reduces encoding needs

**Analogy:**

Think of a toolbox:
- Instead of storing a hammer in a slot (wasting space), you have a permanently attached hammer
- Whenever you need a hammer, it's always there
- Similarly, whenever you need 0, it's always in R0

**Why Not the Other Options?**

1. **Increase available registers:** R0 hardwired actually uses the same space as a regular register, so no net gain
2. **Allow self-modifying code:** This is possible whether R0 is hardwired or not
3. **Optimize cache memory:** Cache optimization is independent of R0's value

**Historical Context:**

MIPS, RISC-V, ARM, and many other architectures hardwire R0 (or equivalent) to zero because:
- RISC philosophy: Keep hardware simple and let software handle complexity
- Zero is the most frequently needed constant
- Hardware simplification reduces chip area and power

**Answer:** To simplify certain operations like initialization

---

## SECTION 3: COMPREHENSION - SEQUENTIAL MULTIPLIER FSM

### Question 130: Role of IDLE State
**Question:** What is the role of the IDLE state in the FSM?

**Correct Answer:** It keeps the system on standby until the start signal is activated.

**Detailed Explanation:**

**Understanding FSM States:**

A Finite State Machine (FSM) is like a traffic light with specific states and transitions.

**The Three States:**
```
IDLE → MULTIPLY → DONE → IDLE (cycle repeats)
```

**IDLE State Role:**

**Function 1: Standby/Waiting**
- The multiplier waits for a "start" signal
- Like a computer waiting for you to click a button
- No computation happens
- Power can be reduced if needed

**Function 2: Initialization**
In the code:
```verilog
IDLE: begin
    if (start) begin
        multiplicand_reg <= multiplicand;    // Load inputs
        multiplier_reg <= multiplier;
        product_reg <= 16'b0;                // Clear product
        state <= MULTIPLY;                   // Ready to multiply
    end
end
```

When "start" signal arrives:
- The input multiplicand and multiplier are loaded into registers
- Product register is cleared to 0
- Transition to MULTIPLY state

**Function 3: Ready State**
- The system is ready to perform a new multiplication
- When done_flag is cleared, it waits for new inputs

**Analogy - Restaurant Order:**

```
IDLE state = Restaurant closed/waiting for customer
├─ Staff on standby
├─ No cooking happening
└─ When customer arrives (start=1):
    ├─ Record the order (load inputs)
    ├─ Clear the counter (reset product)
    └─ Start cooking (go to MULTIPLY state)
```

**Why This State Exists:**

1. **Synchronization:** Ensures inputs are stable before processing
2. **Reusability:** After one multiplication, returns to IDLE for next operation
3. **Energy efficiency:** Avoids wasteful computations when not needed
4. **Clear state management:** Defines exact behavior at startup

**Answer:** It keeps the system on standby until the start signal is activated. When start signal arrives, it loads the inputs and transitions to the MULTIPLY state.

---

### Question 131: Transition from MULTIPLY to DONE
**Question:** What condition causes the FSM to transition from MULTIPLY to DONE?

**Correct Answer:** When count reaches zero.

**Detailed Explanation:**

**Understanding the Loop Counter:**

The count variable tracks how many bits have been processed.

**Code Analysis:**

```verilog
IDLE: begin
    if (start) begin
        count <= 8;  // Initialize count to 8
        // ...
    end
end

MULTIPLY: begin
    if (multiplier_reg[0])
        product_reg <= product_reg + {8'60, multiplicand_reg};
    multiplicand_reg <= multiplicand_reg << 1;
    multiplier_reg <= multiplier_reg >> 1;
    count <= count - 1;  // Decrement count
    
    if (count == 0)      // Check if count is zero
        state <= DONE;
end
```

**Step-by-Step Execution:**

For 8-bit multiplier (count starts at 8):

```
Iteration    count    Action              Transition
1            8        Process bit 0       count becomes 7, stay in MULTIPLY
2            7        Process bit 1       count becomes 6, stay in MULTIPLY
3            6        Process bit 2       count becomes 5, stay in MULTIPLY
4            5        Process bit 3       count becomes 4, stay in MULTIPLY
5            4        Process bit 4       count becomes 3, stay in MULTIPLY
6            3        Process bit 5       count becomes 2, stay in MULTIPLY
7            2        Process bit 6       count becomes 1, stay in MULTIPLY
8            1        Process bit 7       count becomes 0, GO TO DONE
```

**Why Use a Counter?**

The multiplier has 8 bits. Each iteration processes one bit:
- Iteration 1: Check multiplier[0]
- Iteration 2: Check multiplier[1] (after shift)
- ... and so on

After 8 iterations, all bits are processed → count reaches 0 → go to DONE.

**Timing Relationship:**

```
Multiplication Time = Number of bits × Time per iteration
                    = 8 bits × 1 clock cycle per bit
                    = 8 clock cycles total
```

**Why Not Other Options?**

1. **When multiplicand is fully shifted left:** Multiplicand keeps shifting but the check is count-based
2. **When multiplier_reg becomes zero:** This happens eventually, but not the trigger checked in code
3. **When start is deasserted:** Not related to multiplication completion

**Answer:** When count reaches zero, the FSM transitions from MULTIPLY to DONE.

---

### Question 132: Condition for Adding Multiplicand
**Question:** What happens when multiplier_reg[0] is 1 in the MULTIPLY state?

**Correct Answer:** The multiplicand is added to the product register.

**Detailed Explanation:**

**Understanding the Shift-and-Add Algorithm:**

Multiplication can be done bit-by-bit:
- Check each bit of the multiplier
- If bit is 1, add the multiplicand
- If bit is 0, skip the addition
- Shift both operands for the next iteration

**Code Analysis:**

```verilog
MULTIPLY: begin
    if (multiplier_reg[0])  // Check least significant bit
        product_reg <= product_reg + {8'60, multiplicand_reg};
    multiplicand_reg <= multiplicand_reg << 1;
    multiplier_reg <= multiplier_reg >> 1;
end
```

**What Does multiplier_reg[0] Represent?**

```
multiplier_reg = 8'b10110110

Bit positions:  7 6 5 4 3 2 1 0
Values:         1 0 1 1 0 1 1 0

multiplier_reg[0] = rightmost bit = 0 in this example
```

**Detailed Example: 5 × 3 = 15**

```
multiplicand = 5 = 0b0101
multiplier = 3 = 0b0011

Iteration 1:
  multiplier_reg[0] = 3[0] = 1
  → Add multiplicand to product: 0 + 5 = 5
  → Shift multiplicand left: 5 << 1 = 10
  → Shift multiplier right: 3 >> 1 = 1

Iteration 2:
  multiplier_reg[0] = 1[0] = 1
  → Add multiplicand to product: 5 + 10 = 15
  → Shift multiplicand left: 10 << 1 = 20
  → Shift multiplier right: 1 >> 1 = 0

Result: 15 ✓
```

**Why This Works:**

Multiplier 3 = 0b0011 means:
- 3 = 1×(2⁰) + 1×(2¹) = 1 + 2

So: 5 × 3 = 5 × (1 + 2) = 5 × 1 + 5 × 2 = 5 + 10 = 15

**Why Not Other Options?**

1. **Multiplicand is shifted left:** This happens regardless of multiplier_reg[0] value
2. **FSM transitions to DONE:** Only happens when count reaches zero
3. **Multiplier is added:** Wrong—we add multiplicand, not multiplier

**Answer:** The multiplicand is added to the product register.

---

### Question 133: Wait for Start to Go LOW
**Question:** Why does the FSM wait for start to go LOW before transitioning from DONE to IDLE?

**Correct Answer:** To prevent immediate retriggering of multiplication.

**Detailed Explanation:**

**Understanding the Start Signal:**

The "start" signal is like a button:
- HIGH (1) = Button is pressed
- LOW (0) = Button is released

**The Problem Without This Check:**

```verilog
// BAD DESIGN (without the check):
DONE: begin
    done <= 1;
    product <= product_reg;
    state <= IDLE;  // Goes back immediately
end
```

**What Happens:**

1. Multiplication completes
2. FSM goes to DONE state
3. Next clock cycle: Goes back to IDLE
4. If "start" signal is STILL high (user still pressing button):
```verilog
IDLE: begin
    if (start) begin  // start is still 1!
        multiplicand_reg <= multiplicand;
        multiplier_reg <= multiplier;
        state <= MULTIPLY;  // Start multiplying again!
    end
end
```

5. **Unintended multiplications** happen continuously while button is held down!

**The Solution With Check:**

```verilog
DONE: begin
    done <= 1;
    product <= product_reg;
    if (!start)  // Wait for button to be released
        state <= IDLE;
end
```

Now:

```
Button pressed (start=1):
├─ Multiplication occurs
├─ DONE state: done=1, but waits (state stays DONE)
└─ **Waits here** until user releases button

Button released (start=0):
├─ DONE state detects: if (!start) is TRUE
└─ Transitions to IDLE
```

**Analogy - Microwave Oven:**

```
Without the check:
1. Press "Start" button (held down for 3 seconds)
2. Microwave runs for 3 minutes
3. Beeps (DONE)
4. Immediately starts another 3-minute cycle (BAD!)
5. User still holding button
6. Microwave runs another 3 minutes (BAD!)

With the check:
1. Press "Start" button (held down for 3 seconds)
2. Microwave runs for 3 minutes
3. Beeps (DONE)
4. Waits for button release
5. User releases button (start=0)
6. Microwave stops waiting, ready for next command
```

**Why This Matters:**

1. **User intent:** Single press should trigger single multiplication, not continuous ones
2. **Data consistency:** Prevents overwriting results with new computations
3. **Safe design:** Clear stopping point before new cycle

**Hardware Behavior:**

```
Clock cycles:
Cycle 1: start=1 (pressed)  → IDLE state, sees start=1, loads data, goes to MULTIPLY
Cycle 2-9: (multiplying)    → stay in MULTIPLY
Cycle 10: (done)            → DONE state, checks "if (!start)" = "if (!1)" = FALSE, stays in DONE
Cycle 11: start=0 (released) → DONE state, checks "if (!start)" = "if (1)" = TRUE, goes to IDLE
```

**Answer:** To prevent immediate retriggering of multiplication. Without this check, if the start signal remains HIGH, the multiplier would immediately start a new multiplication cycle as soon as the previous one completes, causing continuous unintended computations.

---

### Question 134: Incorrect Multiplicand Values
**Question:** Which of the following multiplicand values produces an incorrect output based on the behavior of the given sequential multiplier when the multiplier is 2?

**Correct Answer:** 128 and 255

**Detailed Explanation:**

**Understanding Overflow in Sequential Multiplier:**

The multiplier is an 8×8 multiplier producing a 16-bit result.

**Maximum Safe Values:**

With 8-bit inputs and 16-bit output:
```
Maximum result = 255 × 255 = 65,025
Maximum representable = 2^16 - 1 = 65,535
```

Seems safe for single multiplication, but there's a catch!

**The Shift-and-Add Problem:**

During the multiplication process, the multiplicand is shifted LEFT each iteration:

```
multiplicand_reg <= multiplicand_reg << 1;
```

If multiplicand is 8 bits:
- Original: 8 bits
- After 1 shift: would need 9 bits
- After 2 shifts: would need 10 bits
- After 8 shifts: would need 16 bits

**The Issue with 128:**

```
128 = 0b10000000 (8-bit maximum signed, or can cause issues)

When multiplied by 2:
128 × 2 = 256 (fits in result)

But during internal shifts:
Initial: 128 = 0b10000000 (8 bits)
Shift 1: 128 << 1 = 256 = 0b100000000 (9 bits - OVERFLOW!)
```

If the shifted value overflows the 8-bit register:
```
8-bit register can hold: 0-255
After shift: 256 (9 bits) → truncated/wrapped!
```

**The Issue with 255:**

```
255 = 0b11111111 (maximum 8-bit value)

When multiplied by 2:
255 × 2 = 510 (fits in result)

But during internal shifts:
Initial: 255 = 0b11111111
Shift 1: 255 << 1 = 510 = 0b111111110 (9 bits - OVERFLOW!)
```

Again, 510 cannot fit in 8 bits!

**Why 64 and 127 Work:**

```
64 = 0b01000000
64 × 2 = 128 (OK)
64 << 1 = 128 (fits in 8 bits) ✓

127 = 0b01111111
127 × 2 = 254 (OK)
127 << 1 = 254 (fits in 8 bits) ✓
```

**Why Only 2-Bit Multiplier Is Given:**

With multiplier = 2:
- Only a few shift operations occur
- This makes the overflow visible

With larger multipliers, more shifts happen, making overflow even more likely.

**Code Issue:**

```verilog
reg [7:0] multiplicand_reg;  // Only 8 bits!
multiplicand_reg <= multiplicand_reg << 1;  // Can overflow!
```

The register is 8 bits, but shifting can produce 9 bits.

**Solution in Better Design:**

```verilog
reg [15:0] multiplicand_reg;  // Use 16 bits
multiplicand_reg <= {8'b0, multiplicand};  // Sign extend
multiplicand_reg <= multiplicand_reg << 1;  // Now has room
```

**Answer:** 128 and 255 produce incorrect outputs because when shifted left during multiplication, they exceed the 8-bit register capacity, causing overflow and incorrect results.

---

## SECTION 4: COMPREHENSION - ASSEMBLY CODE ANALYSIS

### Question 135: Purpose of Assembly Program
**Question:** What is the purpose of this assembly program? (Program computes sum of array elements)

**Correct Answer:** To compute the sum of 10 elements in an array

**Detailed Explanation:**

**Program Code:**
```assembly
ADDI r1, r0, 1000   // r1 = 1000 (base address)
ADDI r2, r0, 10     // r2 = 10 (counter)
ADDI r5, r0, 0      // r5 = 0 (sum accumulator)

loop:
    BEQ r2, r0, done         // if counter == 0, jump to done
    LOAD r3, r1, 0           // r3 = memory[1000]
    ADD r5, r5, r3           // r5 = r5 + r3 (add to sum)
    ADDI r1, r1, 4           // r1 += 4 (next address)
    SUBI r2, r2, 1           // r2 -= 1 (decrement counter)
    JUMP loop                // go back to loop
done:
```

**Program Breakdown:**

**Initialization Phase:**

```
ADDI r1, r0, 1000
└─ r1 = 1000 (memory base address of array)

ADDI r2, r0, 10
└─ r2 = 10 (counter for 10 elements)

ADDI r5, r0, 0
└─ r5 = 0 (accumulator to store sum)
```

**Loop Phase:**

```
loop:
    BEQ r2, r0, done
    └─ if r2 (counter) == 0, exit loop

    LOAD r3, r1, 0
    └─ r3 = memory[r1] = memory[1000] (load current array element)

    ADD r5, r5, r3
    └─ r5 = r5 + r3 (add current element to sum)

    ADDI r1, r1, 4
    └─ r1 += 4 (move to next element, assuming 4-byte integers)

    SUBI r2, r2, 1
    └─ r2 -= 1 (decrement counter)

    JUMP loop
    └─ repeat loop
```

**Execution Trace:**

```
Initial: r1=1000, r2=10, r5=0

Iteration 1:
  r2 != 0, so continue
  r3 = mem[1000]
  r5 = 0 + mem[1000]
  r1 = 1004, r2 = 9
  JUMP back

Iteration 2:
  r2 != 0, so continue
  r3 = mem[1004]
  r5 = mem[1000] + mem[1004]
  r1 = 1008, r2 = 8
  JUMP back

... (iterations 3-9 similar)

Iteration 10:
  r2 != 0, so continue
  r3 = mem[1036]
  r5 = sum of all 10 elements
  r1 = 1040, r2 = 1
  JUMP back

Iteration 11:
  r2 = 0, so jump to done
  Exit loop

Final: r5 = sum of all 10 array elements
```

**Verification:**

```
Elements in memory:
mem[1000] = A₀
mem[1004] = A₁
mem[1008] = A₂
...
mem[1036] = A₉

Final r5 = A₀ + A₁ + A₂ + ... + A₉
```

**Why Not Other Options?**

1. **Find maximum:** Would need comparisons (BLT, BGT)
2. **Run infinite loop:** Would need unconditional jump, not BEQ
3. **Count elements:** Wouldn't need LOAD and ADD

**Answer:** To compute the sum of 10 elements in an array

---

### Question 136: BEQ Instruction Function
**Question:** What does the instruction `BEQ r1, r0, done` do?

**Correct Answer:** Checks if r1 is zero and exits the loop if true

**Detailed Explanation:**

**Understanding Branch-if-Equal (BEQ):**

BEQ = Branch If Equal

**Syntax:**
```
BEQ register1, register2, target_address
```

**Action:**
- Compare register1 with register2
- If they are **equal**, jump to target_address
- If they are **not equal**, continue to next instruction

**Decoding `BEQ r1, r0, done`:**

```
register1 = r1
register2 = r0 (always 0)
target_address = done

Meaning: If r1 equals r0 (which is 0), jump to done
         = If r1 == 0, jump to done
```

**Execution Logic:**

```
BEQ r1, r0, done
    ↓
Is r1 == r0 ?
    ├─ YES (r1 == 0) → PC ← address of "done" label
    │                → Jump to done label
    │                → Exit loop
    │
    └─ NO (r1 != 0) → PC ← PC + 4
                     → Continue to next instruction
                     → Stay in loop
```

**Context in Loop:**

```assembly
ADDI r2, r0, 10     // r2 = 10

loop:
    BEQ r2, r0, done     // If r2 == 0, exit
    // ... do something ...
    SUBI r2, r2, 1      // Decrement r2
    JUMP loop            // Loop back

done:
```

**Execution Trace:**

```
r2 = 10  → BEQ: 10 != 0 ? NO  → continue
r2 = 9   → BEQ: 9 != 0 ? NO   → continue
r2 = 8   → BEQ: 8 != 0 ? NO   → continue
...
r2 = 1   → BEQ: 1 != 0 ? NO   → continue
r2 = 0   → BEQ: 0 == 0 ? YES  → JUMP to done
```

**Why Not Other Options?**

1. **Compares and jumps if different:** That's BNE (Branch if Not Equal)
2. **Resets r1 to zero:** That would be ADDI, not BEQ
3. **Checks if r1 is zero and enters loop:** No—it exits the loop

**Answer:** Checks if r1 is zero and exits the loop if true.

---

### Question 137: Effect of ADDI r1, r1, 1
**Question:** What is the effect of `ADDI r1, r1, 1` inside the loop?

**Correct Answer:** Increments r1 by 1 in each iteration

**Detailed Explanation:**

**Decoding ADDI:**

ADDI = Add Immediate

**Syntax:**
```
ADDI destination, source, immediate_value
```

**Action:**
```
destination ← source + immediate_value
```

**Decoding `ADDI r1, r1, 1`:**

```
destination = r1
source = r1
immediate_value = 1

Meaning: r1 ← r1 + 1
         = Increment r1 by 1
```

**Execution:**

Each time the instruction executes:

```
Before: r1 = 100
ADDI r1, r1, 1
After: r1 = 101

Next iteration:
Before: r1 = 101
ADDI r1, r1, 1
After: r1 = 102

Next iteration:
Before: r1 = 102
ADDI r1, r1, 1
After: r1 = 103
```

**In Loop Context:**

```assembly
loop:
    BEQ r1, r0, done
    // ... do something ...
    ADDI r1, r1, 1       // r1 increments each iteration
    JUMP loop
```

**Iteration Trace:**

```
Iteration 1: r1 = 50 → do something → ADDI r1,r1,1 → r1 = 51
Iteration 2: r1 = 51 → do something → ADDI r1,r1,1 → r1 = 52
Iteration 3: r1 = 52 → do something → ADDI r1,r1,1 → r1 = 53
```

**Why Not Other Options?**

1. **Decreases r1 by 1:** That would be SUBI (subtract immediate)
2. **Resets r1 to 1 every iteration:** Would use `ADDI r1, r0, 1`
3. **Moves next memory address:** That's conceptual; actual address is in register

**Answer:** Increments r1 by 1 in each iteration.

---

### Question 138: Final Value in Register r2
**Question:** What is stored in register r2 at the end of execution?

**Correct Answer:** The sum of 10 elements of the array (as shown in program context)

**Detailed Explanation:**

**Program Analysis:**

Looking at a typical array sum program:

```assembly
ADDI r1, r0, 1000   // r1 = base address
ADDI r2, r0, 10     // r2 = number of elements
ADDI r5, r0, 0      // r5 = sum (accumulator)

loop:
    BEQ r2, r0, done
    LOAD r3, r1, 0
    ADD r5, r5, r3          // r5 accumulates sum
    ADDI r1, r1, 4
    SUBI r2, r2, 1          // r2 decrements each iteration
    JUMP loop

done:
    // r2 is now 0 (counter reached zero)
    // r5 contains the sum
```

**Why r2 Becomes 0:**

```
r2 starts at 10

Iteration 1: SUBI r2,r2,1 → r2 = 9
Iteration 2: SUBI r2,r2,1 → r2 = 8
Iteration 3: SUBI r2,r2,1 → r2 = 7
...
Iteration 9: SUBI r2,r2,1 → r2 = 1
Iteration 10: SUBI r2,r2,1 → r2 = 0
BEQ r2,r0,done → condition TRUE, exit loop
```

**Final State:**

```
r1 = 1040 (moved past all elements)
r2 = 0 (counter exhausted)
r5 = sum of all 10 elements
```

**Note:** The exact value in r2 depends on program structure, but in typical counter-based loops, r2 is **0** at the end (loop exit condition).

**Why Not Other Options?**

1. **Last element of array:** That's in r3, not r2
2. **Unknown value:** We can trace execution, so it's not unknown
3. **Memory address:** That's in r1

**Answer:** Register r2 contains **0** (the loop counter that reached zero causing loop exit), or in the context of other programs, contains the counter value (which is typically 0).

---

### Question 139: Effect of Removing JUMP loop
**Question:** What would happen if `JUMP loop` were removed?

**Correct Answer:** The program would execute only once

**Detailed Explanation:**

**Understanding JUMP:**

JUMP = Unconditional branch to a target address

```
JUMP target_address
    ↓
PC ← address of target_address
    ↓
Next instruction fetches from target_address
```

**Current Flow With JUMP:**

```assembly
loop:
    BEQ r2, r0, done      // Check exit condition
    LOAD r3, r1, 0        // Do work
    ADD r5, r5, r3
    ADDI r1, r1, 4
    SUBI r2, r2, 1
    JUMP loop             // ALWAYS go back to loop label
done:
```

**Execution Flow:**

```
PC: 100  loop: BEQ r2, r0, done      → r2 = 10, so continue
PC: 104        LOAD r3, r1, 0        → load first element
PC: 108        ADD r5, r5, r3        → add to sum
PC: 112        ADDI r1, r1, 4        → next address
PC: 116        SUBI r2, r2, 1        → r2 = 9
PC: 120        JUMP loop             → go back to PC:100
PC: 100  loop: BEQ r2, r0, done      → r2 = 9, so continue
PC: 104        LOAD r3, r1, 0        → load second element
...
```

**Flow Without JUMP:**

```assembly
loop:
    BEQ r2, r0, done      // Check exit condition
    LOAD r3, r1, 0        // Do work
    ADD r5, r5, r3
    ADDI r1, r1, 4
    SUBI r2, r2, 1
                          // NO JUMP - just fall through!
done:
```

**Execution Flow:**

```
PC: 100  loop: BEQ r2, r0, done      → r2 = 10, so continue
PC: 104        LOAD r3, r1, 0        → load first element
PC: 108        ADD r5, r5, r3        → add to sum
PC: 112        ADDI r1, r1, 4        → next address
PC: 116        SUBI r2, r2, 1        → r2 = 9
PC: 120        (next instruction after SUBI)
PC: 124  done:                        → reached done label directly

Loop never repeats!
```

**Result:**

```
Only first element is processed
r2 = 9 (not 0)
r5 = first element only (not sum of all 10)
```

**Why This Matters:**

The JUMP instruction creates the **loop**:
- Without it: Linear execution, one-pass only
- With it: Circular execution, repeating behavior

**Analogy:**

**With JUMP:** 
```
Read line 1
Read line 2
Read line 3
↓
Go back to line 1
↓
Read line 1 (repeated)
```

**Without JUMP:**
```
Read line 1
Read line 2
Read line 3
↓
(Exit, never read line 1 again)
```

**Answer:** The program would execute only once (process only the first array element) and then exit, because without the JUMP instruction to loop back, the program would flow sequentially to the done label after executing the loop body once.

---

## SECTION 5: KEY CONCEPTS SUMMARY TABLE

| Concept | Key Point | Example |
|---------|-----------|---------|
| **Modularity** | Breaks complexity into manageable blocks | ALU, Controller, Memory separate |
| **Bitwise Operations** | Shift right by n = divide by 2ⁿ | 64 >> 3 = 8 |
| **Parallel Multiplier** | Ripple-carry reduces latency (FALSE) | Tree-based is better |
| **Assertions** | Immediate continues, Concurrent stops | Debug vs. Design check |
| **6T SRAM** | Each bit needs 6 transistors | 512 bits = 3,072 transistors |
| **FIFO Empty/Full** | Equal = Empty, One behind = Full | Queue pointer logic |
| **Fan-Out** | More loads = higher delay | Delay ∝ Fan-out |
| **Program Counter** | Reset to fixed address (vector) | PC ← 0x00000000 |
| **Branch Logic** | FALSE = continue sequentially | PC ← PC + 4 |
| **Two's Complement** | Invert + Add 1 = negate | 11001 = -7 |
| **Throughput** | Clock freq ÷ Cycles per op | 200MHz ÷ 8 = 25 Mops/s |
| **Memory Config** | M × N: M addresses, N bits each | 4 × 16 = 4 locations, 16 bits |
| **Instruction Encoding** | Binary pattern for {dest, src1, op, src2} | 011_001_010_010 |
| **Register R0** | Hardwired to 0 for simplification | Always available zero constant |
| **FSM States** | IDLE → MULTIPLY → DONE → IDLE | Sequential multiplier cycle |
| **Shift-and-Add** | Multiply: Check bit, add if 1, shift | 5 × 3: Add once, add twice |

---

## CONCLUSION

Digital System Design encompasses:
1. **Logical design** (gates, multiplexers, decoders)
2. **Sequential design** (FSMs, state machines)
3. **Memory design** (SRAM, FIFO, caches)
4. **Arithmetic** (adders, multipliers, shifters)
5. **Assembly programming** (instruction execution, loops)

Each concept builds upon fundamentals of binary logic and sequential processing, forming the basis for modern computer architectures.