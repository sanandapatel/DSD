# Week 2: Arithmetic and Logic Unit (ALU) - Detailed Lecture Notes
## Complete Explanation with Code Comments and Design Considerations

---

## **Overview: From Data Path to Complete Processor Architecture**

This lecture focuses on designing the **Arithmetic and Logic Unit (ALU)**, which is a critical component of any processor's data path. We'll build on the register bank concepts from W2-L1 and add computational capability to create a functional processing pipeline.

---

## **Part 1: ALU Fundamentals**

### **1.1 The Discriminant Computation Case Study**

**Context:** We're designing a circuit to compute the discriminant of a quadratic equation
- Formula: **Δ = b² - 4ac**
- This is a practical example of a real computation that requires multiple operations in sequence

**Why This Example?**
- Simple enough to understand the concepts
- Complex enough to demonstrate register allocation and data flow
- Shows real-world processor design patterns

---

### **1.2 Operations in Our ALU**

Our ALU will support **four basic arithmetic operations:**

| Operation | Symbol | Example | 8-bit Result Challenge |
|-----------|--------|---------|----------------------|
| **Addition** | + | 10 + 5 = 15 | Small numbers OK; 200+100 overflows |
| **Subtraction** | - | 10 - 5 = 5 | Can result in negative numbers |
| **Multiplication** | × | 10 × 5 = 50 | 200 × 200 = 40000 (exceeds 8 bits!) |
| **Division** | ÷ | 10 ÷ 5 = 2 | Integer division loses fractional parts |

**Note:** For the discriminant computation itself, technically we only need multiplication and subtraction. We include addition and division for completeness and future use cases.

---

### **1.3 Number Representation: 8-bit Constraint**

#### **The Problem: Limited Range**

```
8-bit unsigned representation:
Range: 0 to 255 (2^8 - 1)

Example Issues:
- If input A = 200 and input B = 100
- A + B = 300, but 8-bit max = 255 → OVERFLOW!
- A × B = 20,000, but 8-bit max = 255 → HUGE OVERFLOW!

8-bit signed representation (two's complement):
Range: -128 to +127 (still 256 total values)
Problems are even more severe for signed numbers!
```

#### **The Solution: Accept the Limitation**

We're making a **simplifying assumption** that:
1. Input values are small enough that results fit in 8 bits
2. OR we'll upgrade to larger bit widths (16-bit, 32-bit) later
3. OR we'll implement special handling (saturation, wrapping, error flags)

**Key Insight:** This is NOT a fundamental problem with our design approach—it's just a parameter we can change later. Everything we design works equally well with 16-bit, 32-bit, or 64-bit representations.

#### **Overflow and Carry Conditions**

```
Addition Overflow Example:
200 + 100 = 300
In 8 bits: 1100_1000 + 0110_0100
Result: 0010_1100 (which is 44 in decimal, WRONG!)
Carry flag set: indicates overflow occurred

Multiplication Overflow Example:
200 × 200 = 40,000
Result completely wrong in 8 bits

Division Truncation:
15 ÷ 4 = 3.75 (true result)
Integer division: 3 (fractional part lost)
So we cannot represent fractional results with integer arithmetic
```

#### **How to Handle These Issues**

1. **Wider bus width:** Use 16-bit or 32-bit instead of 8-bit
2. **Fixed-point representation:** Allocate bits for fractional part (e.g., 4 bits integer, 4 bits fraction)
3. **Floating-point:** Use IEEE 754 standard (more complex hardware)
4. **Status flags:** Capture overflow/underflow and let software handle it

**Our Approach:** Accept the limitation for now; focus on the architecture.

---

## **Part 2: ALU Architecture and Design**

### **2.1 Operation Selection and Encoding**

#### **How Many Selection Bits Do We Need?**

For N operations, we need **⌈log₂(N)⌉ bits** to uniquely encode each one.

**Formula:** Minimum bits = ceiling(log₂(number_of_operations))

```
Examples:

2 operations: ⌈log₂(2)⌉ = 1 bit
   00 → Operation 0
   01 → Operation 1
   
4 operations: ⌈log₂(4)⌉ = 2 bits
   00 → ADD
   01 → SUB
   10 → MUL
   11 → DIV
   
5 operations: ⌈log₂(5)⌉ = 3 bits (need at least 3 bits even though log₂(5)=2.32)
   000 → Operation 0
   001 → Operation 1
   010 → Operation 2
   011 → Operation 3
   100 → Operation 4

8 operations: ⌈log₂(8)⌉ = 3 bits
   000 to 111 → All 8 operations

16 operations: ⌈log₂(16)⌉ = 4 bits
   0000 to 1111 → All 16 operations
```

**For our 4-operation ALU:** We need **2 selection bits**

---

### **2.2 Operation Encoding Schemes**

#### **Method 1: Hardcoded Constants (Recommended)**

```verilog
// Define operation codes as local parameters
// These are explicit bit patterns we choose
localparam OP_ADD = 2'b00;   // Addition
localparam OP_SUB = 2'b01;   // Subtraction  
localparam OP_MUL = 2'b10;   // Multiplication
localparam OP_DIV = 2'b11;   // Division

// Why use localparam instead of regular parameter?
// localparam: Local to module, cannot be overridden by instantiation
// parameter: Can be overridden, allowing flexibility
// We use localparam because these are internal constants
```

**Advantages:**
- Explicit, readable code
- Operation codes are fixed and known
- Easy to debug and understand

**Disadvantages:**
- Need to manually maintain encoding
- Different modules must agree on encoding (potential for mismatch)

#### **Method 2: Enumerated Data Types (Be Careful!)**

```verilog
// enum syntax in SystemVerilog
enum logic [1:0] {
    OP_ADD = 2'b00,
    OP_SUB = 2'b01,
    OP_MUL = 2'b10,
    OP_DIV = 2'b11
} operation_e;

// WARNING: Enum enumerations can be compiler-dependent!
// Different tools might assign values differently
// Only use if you explicitly assign values (as shown above)
```

**Caution:** If the operation code comes from another module (like a controller), both modules must use the SAME enum definition, or explicitly assigned values. Otherwise, the compiler might choose different orderings in each module.

**Example of Enum Problem:**
```
Module A:
enum {ADD, SUB, MUL, DIV} op;  // Compiler assigns 0,1,2,3

Module B:
enum {ADD, SUB, MUL, DIV} op;  // Different compiler might assign 0,1,2,3
                               // But if only a subset is used, assignments might differ!
                               // Result: Enum values don't match between modules!
```

**Best Practice:** If using enums across modules, **explicitly assign values:**
```verilog
typedef enum logic [1:0] {
    OP_ADD = 2'b00,
    OP_SUB = 2'b01,
    OP_MUL = 2'b10,
    OP_DIV = 2'b11
} operation_t;
```

---

### **2.3 ALU Implementation in Verilog**

#### **Complete ALU Module**

```verilog
module alu #(
    parameter WIDTH = 8          // Data width: number of bits per operand
) (
    input  logic [WIDTH-1:0] operand_a,    // First input operand (8 bits for WIDTH=8)
    input  logic [WIDTH-1:0] operand_b,    // Second input operand
    input  logic [1:0] operation,          // Operation selector: 2 bits for 4 operations
    output logic [WIDTH-1:0] result        // Output result (same width as inputs)
);

// Define operation codes as local parameters for readability
// These constants represent which operation to perform
localparam OP_ADD = 2'b00;   // 2'b00 = 0 in binary = Addition
localparam OP_SUB = 2'b01;   // 2'b01 = 1 in binary = Subtraction
localparam OP_MUL = 2'b10;   // 2'b10 = 2 in binary = Multiplication
localparam OP_DIV = 2'b11;   // 2'b11 = 3 in binary = Division

// COMBINATIONAL LOGIC - Purely combinational, no state
// This always_comb block runs whenever inputs change
// No clock needed, result appears after gate propagation delays
always_comb begin
    // Case statement selects which operation to perform based on 'operation' input
    case (operation)
        OP_ADD: result = operand_a + operand_b;    // Add two operands
        OP_SUB: result = operand_a - operand_b;    // Subtract operand_b from operand_a
        OP_MUL: result = operand_a * operand_b;    // Multiply operands
        OP_DIV: result = operand_a / operand_b;    // Divide operand_a by operand_b (integer division)
        default: result = '0;                      // Default: return all zeros
    endcase
end

endmodule
```

**Code Comments Explained:**

- **`always_comb`**: Combinational logic block
  - No clock or event trigger
  - Executes whenever ANY input changes
  - Must fully specify all outputs (no latches)
  - Result available after propagation delay (nanoseconds)

- **`case (operation)`**: Multiplexer logic
  - Selects which operation based on 'operation' input
  - Like a multiplexer selecting between 4 different circuits
  - Only one branch executes per cycle

- **Result width:** Output is same width as inputs
  - Overflow/underflow happens silently (result truncates)
  - Upper bits discarded

---

#### **Synthesis Challenges**

**Problem with Division:**
```verilog
// This code will likely FAIL to synthesize:
OP_DIV: result = operand_a / operand_b;

// Why? Because:
// 1. Division in combinational logic is complex
// 2. Would require elaborate circuit (multiple gates)
// 3. Most synthesis tools cannot implement combinational dividers
// 4. Division is usually iterative (multiple clock cycles)
```

**Solutions:**

1. **Remove division from ALU:**
   - Only implement: ADD, SUB, MUL
   - Use 1-bit operation selector
   - Implement division separately (if needed)

2. **Use iterative/sequential division:**
   - Implement division as finite state machine
   - Takes multiple clock cycles
   - Requires control logic (later topic)

3. **Use pre-built IP blocks:**
   - Many vendors provide divider generators
   - Optimize for speed vs area vs latency

**Our Approach:** For now, accept that division won't synthesize, and focus on the architecture. Later, we'll handle operations that span multiple cycles.

---

#### **Key Insight: ALU as Parallel Computation**

```
Conceptually, here's what happens inside the ALU:

        operand_a ──┐
        operand_b ──┤
                    ├──> Adder Circuit ──┐
        operation ──┤                     │
                    ├──> Subtractor ──┐   │
                    │                 │   │
                    ├──> Multiplier ──┼─> Multiplexer ──> result
                    │                 │
                    ├──> Divider ─────┘
                    │
        operation ──┘ (select lines for mux)

Architecture:
- ALL operations run in PARALLEL
- Adder computes A+B
- Subtractor computes A-B  
- Multiplier computes A*B
- Divider computes A/B
- Multiplexer selects which result based on operation code

Why ALL operations?
- Combinational logic: No state, no cycles
- All paths active simultaneously
- Mux just picks the right answer
- Alternative: Serial approach (slower, uses mux of inputs, one operation)

Delay:
- Ripple-carry adder: ~2-3 ns (simple)
- Subtractor: ~2-3 ns (similar to adder)
- Multiplier: ~5-10 ns (Wallace tree or other methods)
- Divider: Very complex (10-20+ ns or multiple cycles)
- Multiplexer: ~0.5-1 ns (selects which result)

Critical path = Longest delay through any operation + mux delay
            ≈ Multiplier delay + Mux delay
            ≈ 10-11 ns for typical 8-bit ALU
```

---

## **Part 3: Register File for ALU Interface**

### **3.1 Why a Register File?**

The ALU needs **operands** (inputs) and a place to store **results** (outputs). We can't just feed inputs directly from external pins—we need temporary storage between operations.

```
Data Flow:
External Input → Register File (storage) 
              → ALU (computation)
              → Register File (storage)
              → External Output or next ALU operation
```

---

### **3.2 Register File Interface for ALU**

#### **Read-Write Port Configuration**

For the discriminant computation, we need:
- **Two read ports:** Read two operands simultaneously for ALU
- **One write port:** Write ALU result back to register

Why two read ports?
- ALU takes two inputs (A and B)
- Both must come from register file at same time
- Allows: `result = reg[address1] OP reg[address2]`

```
Register File Ports:

Read Port 1 (rs1):
  Input: rs1_addr (which register to read)
  Output: rs1_data (the value from that register)
  
Read Port 2 (rs2):
  Input: rs2_addr (which register to read)
  Output: rs2_data (the value from that register)
  
Write Port (rd):
  Inputs: rd_addr (where to write), wr_data (what to write)
  Action: On clock edge with write enable
```

---

### **3.3 Register File Module with Multiple Ports**

#### **Register File for 8 Registers, 8-bit values**

```verilog
module register_file #(
    parameter WIDTH = 8,       // Width of each register (8 bits)
    parameter DEPTH = 8,       // Number of registers (0-7 = 8 total)
                               // Needs log2(8) = 3 bits to address
    parameter ADDR_WIDTH = 3   // Address width: log2(DEPTH) = 3 bits
) (
    input  logic clk,
    input  logic rst_n,
    
    // Write Port (synchronous - happens on clock edge)
    input  logic wr_enable,                  // Enable writing on this cycle
    input  logic [ADDR_WIDTH-1:0] wr_addr,   // Which register to write (0-7)
    input  logic [WIDTH-1:0] wr_data,        // Value to write
    
    // Read Port 1 (asynchronous - happens immediately)
    input  logic [ADDR_WIDTH-1:0] rs1_addr,  // Which register to read
    output logic [WIDTH-1:0] rs1_data,       // Value from that register
    
    // Read Port 2 (asynchronous - happens immediately)
    input  logic [ADDR_WIDTH-1:0] rs2_addr,  // Which register to read
    output logic [WIDTH-1:0] rs2_data        // Value from that register
);

// Register array: 8 registers × 8 bits each
logic [WIDTH-1:0] registers [DEPTH-1:0];

// WRITE OPERATION (Synchronous - happens on clock edge)
// Block is triggered by positive clock edge or negative reset
always_ff @(posedge clk or negedge rst_n) begin
    if (~rst_n) begin
        // RESET: Initialize all registers to zero
        // Loop through each register and clear it
        for (int i = 0; i < DEPTH; i = i + 1) begin
            registers[i] <= '0;  // '0 means all bits = 0, works for any width
        end
    end else if (wr_enable) begin
        // WRITE: On clock edge, if write enabled, update the register
        // wr_addr selects which register (0-7)
        // wr_data contains the new value to store
        registers[wr_addr] <= wr_data;
        
        // NOTE: The assignment is NON-BLOCKING (<=)
        // Non-blocking is correct for sequential logic
        // Value doesn't appear at output until NEXT clock edge
    end
    // If wr_enable is low, all registers retain their current values
end

// READ OPERATIONS (Asynchronous - happens immediately)
// Combinational assignment: no clock needed
// Address directly selects which register's data appears at output

// Read Port 1: rs1_addr selects which register → rs1_data is that value
assign rs1_data = registers[rs1_addr];

// Read Port 2: rs2_addr selects which register → rs2_data is that value
assign rs2_data = registers[rs2_addr];

// KEY INSIGHT: Addresses can be the SAME
// If rs1_addr == rs2_addr, both outputs show same register (allowed)
// If rs1_addr == wr_addr, we read OLD value (not yet written)

endmodule
```

**Important Timing Notes:**

```
Register File Timing:

At clock edge:
1. Read addresses settle first
2. Combinational logic (mux) selects data from array
3. rs1_data and rs2_data appear immediately (combinational)
4. This data goes to ALU

ALU processes during cycle:
5. ALU does computation (while register file is idle)
6. Result stabilizes before next clock edge

Just before next clock edge:
7. Setup time: ALU result must be stable at wr_data input
8. On clock edge: Result written to registers[wr_addr]
9. On NEXT clock edge: New value available for reading

Latency:
- Read: 0 cycles (combinational)
- Write: 1 cycle delay (result appears next cycle)
```

---

## **Part 4: Addressing Register Allocation**

### **4.1 The Register Allocation Problem**

**Question:** When computing the discriminant Δ = b² - 4ac, where do we store intermediate results?

```
Initial state:
- R0 ← value a (coefficient)
- R1 ← value b (coefficient)
- R2 ← value c (coefficient)
- R7 ← constant 4 (for later use)

Computation sequence:
1. temp1 = b * b        → needs register for result
2. temp2 = a * c        → needs different register
3. temp3 = temp2 * 4    → needs different register
4. result = temp1 - temp3  → final result
```

### **4.2 Register Lifetime Analysis**

**Key Concept:** The **lifetime** of a variable is the range of instructions where it might be used.

```
Pseudocode for discriminant:

Instruction 1: R3 = R1 * R1   // b²
Instruction 2: R4 = R0 * R2   // a*c
Instruction 3: R4 = R4 * R7   // (a*c)*4   ← REUSES R4!
Instruction 4: R5 = R3 - R4   // final result

Lifetime of R0 (value a):
- Set initially
- Used in instruction 2 (a*c)
- Used in instruction 3? No, only R4 appears
- Lifetime: Instructions 0-3 (could potentially be reused after inst 2)

Lifetime of R1 (value b):
- Set initially
- Used in instruction 1 (b²)
- Never used again
- Could be overwritten after instruction 1

Lifetime of R3 (b²):
- Written in instruction 1
- Used in instruction 4
- Lifetime: Instructions 1-4 (MUST be preserved!)

Lifetime of R4 (4*a*c):
- Written in instruction 2
- Used again in instruction 3 (both source and destination)
- Used in instruction 4
- Lifetime: Instructions 2-4
```

### **4.3 Live Variable Analysis**

A variable is **live** at an instruction if:
- It has been defined previously, AND
- It will be used in some future instruction, AND
- The current instruction might affect when it's used

```
Example: Which variables are live at instruction 3?

Instruction 3: R4 = R4 * R7

Before instruction 3:
- R0: Not used after inst 2, and not read in 3 → NOT live
- R1: Not used after inst 1, and not read in 3 → NOT live
- R2: Not used after inst 2, and not read in 3 → NOT live
- R3: Used in instruction 4 → LIVE (needed for final result)
- R4: Read in inst 3 (source), written in inst 3 (dest) → LIVE
- R7: Read in inst 3 → LIVE

After instruction 3:
- R0: Can be overwritten (never used again)
- R1: Can be overwritten (never used again)
- R2: Can be overwritten (never used again)
- R3: Must be preserved
- R4: Must be preserved (used in inst 4)
- R7: Can be overwritten if 4 is no longer needed
```

### **4.4 Why This Matters for Register Allocation**

```
Goal: Minimize number of registers needed

Algorithm:
1. Compute lifetime for each variable
2. For variable V, find last use instruction L(V)
3. After instruction L(V), V's register can be reused for other values

Example - Optimal allocation:

Instruction 1: R3 = R1 * R1   // temp1 (lifetime ends at inst 4)
Instruction 2: R4 = R0 * R2   // temp2 (lifetime ends at inst 3)
Instruction 3: R4 = R4 * R7   // temp2 is last used here!
                              // After this, can reuse R4 for next variable
Instruction 4: R5 = R3 - R4   // temp3 (final result)

Result: Need R0,R1,R2,R3,R4,R5,R7 = 7 registers minimum
        Could get away with fewer if we reuse R0,R1,R2 for other operations

But if we don't do analysis:
Instruction 1: R3 = R1 * R1
Instruction 2: R4 = R0 * R2
Instruction 3: R5 = R4 * R7   // Created new register instead of reusing R4
Instruction 4: R6 = R3 - R5   // Created another new register

Result: Need R0-R7 = 8 registers (wastes one!)
```

### **4.5 Handling Immediate Operands**

**Problem:** The ALU needs two operands from register file. What if we need a constant?

```
Example: temp3 = temp2 * 4

We need to multiply by constant 4, but constants aren't in register file
Solution: Store constant in a register!
- R7 contains the value 4
- ALU reads R4 and R7, multiplies them
- R4 = R4 * R7 works fine!

Drawback:
- Uses up one register (R7) just for the constant
- Not scalable if many different constants needed

Better solution (not yet implemented):
- Immediate operand encoding in instruction
- Instruction specifies: "multiply register by constant"
- Constant value encoded directly in instruction bits
- Would require more complex instruction format
```

---

## **Part 5: Critical Path and Timing Analysis**

### **5.1 What Is Critical Path?**

The **critical path** is the longest delay from input to output through any combinational circuit.

```
Critical Path = The path through the circuit with the longest total gate delay

Circuit propagation time = Critical path delay + setup/hold times

Processor clock frequency inversely related to critical path:
f_max = 1 / (critical_path_delay + setup_time + clock_skew)
```

### **5.2 Paths Through Our ALU**

```
Possible paths from inputs to output:

Path 1: operand_a → Adder → Multiplexer → result
        Delay = T_add + T_mux

Path 2: operand_b → Adder → Multiplexer → result
        Delay = T_add + T_mux

Path 3: operand_a → Subtractor → Multiplexer → result
        Delay = T_sub + T_mux ≈ T_add (similar delay)

Path 4: operand_a → Multiplier → Multiplexer → result
        Delay = T_mul + T_mux  ← LONGEST (multiplier is slow!)

Path 5: operand_b → Multiplier → Multiplexer → result
        Delay = T_mul + T_mux  ← LONGEST (multiplier is slow!)

Path 6: operation[0] → Multiplexer → result
        Delay = T_mux (just selector logic)
        ← SHORTEST

Typical gate delays (8-bit):
- T_mux = 0.5 ns (fast, just transmission gates/logic)
- T_add = 2-3 ns (ripple carry adder)
- T_sub = 2-3 ns (similar to adder)
- T_mul = 5-10 ns (Wallace tree multiplier is complex!)

Critical path ≈ 5-10 ns + 0.5 ns = 5.5-10.5 ns
Maximum frequency ≈ 1 / (10.5 ns) ≈ 95 MHz
```

### **5.3 How Multiplexer Works**

```
Multiplexer with 4 inputs:

   in[0] ─┐
   in[1] ─┤
   in[2] ─┼─> out
   in[3] ─┤
           └─ sel[1:0] (select lines)

Logic:
IF sel == 2'b00: out = in[0]
IF sel == 2'b01: out = in[1]
IF sel == 2'b10: out = in[2]
IF sel == 2'b11: out = in[3]

Implementation (simplified):
out = (in[0] & ~sel[1] & ~sel[0])  // Select in[0] when sel=00
    | (in[1] & ~sel[1] &  sel[0])  // Select in[1] when sel=01
    | (in[2] &  sel[1] & ~sel[0])  // Select in[2] when sel=10
    | (in[3] &  sel[1] &  sel[0])  // Select in[3] when sel=11

Each path:
1. Decode selector bits (few gates)
2. AND each input with its select term
3. OR all results together
Total: 2-3 gate delays

Trick: Even if selector is stable, logic gates still toggle!
- If input changes while selector stable, output changes
- If selector changes while inputs stable, different input is selected
- Takes time for new value to appear (not instant)
```

### **5.4 Critical Path Through Register File + ALU**

```
Complete timing for one instruction cycle:

1. Clock rises (start of cycle)
2. Read address (rs1_addr, rs2_addr) propagates through mux
3. Register file mux delays ≈ 2-3 ns
4. rs1_data and rs2_data stable
5. ALU input stage (additional small delay)
6. Multiplier computes (longest operation): 5-10 ns
7. Multiplexer selects result: 0.5 ns
8. wr_data stabilizes
9. Just before next clock edge: Setup time requires data stable
10. Next clock edge: New value written to register

Total available time = Clock period
Clock period = Critical path + setup time + margin
T_clock = 10.5 ns (for critical path) + 0.5 ns (setup) + 0.5 ns (margin) ≈ 11.5 ns

Maximum frequency = 1 / 11.5 ns ≈ 87 MHz
```

---

## **Part 6: Complete Data Path Assembly**

### **6.1 Connecting ALU and Register File**

```
Module: ALU_RegisterFile_Top

Inputs:
- clk: Clock signal (synchronized)
- rst_n: Active-low reset
- wr_enable: Control signal (enable writing results back)
- alu_op[1:0]: Operation selection (00=ADD, 01=SUB, 10=MUL, 11=DIV)
- rs1_addr[2:0]: Read address for operand 1 (which register)
- rs2_addr[2:0]: Read address for operand 2 (which register)
- rd_addr[2:0]: Write address (where to store result)

Outputs:
- rs1_data[7:0]: Value read from register rs1
- rs2_data[7:0]: Value read from register rs2
- result[7:0]: ALU result (ready to be written)

Internal data flow:

[Register File]
     | (asynchronous read)
     |rs1_data, rs2_data
     v
[ALU]
     | (combinational computation)
     |result
     v
[back to Register File write port on next clock edge]
```

### **6.2 SystemVerilog Top-Level Module**

```verilog
module alu_regfile_top #(
    parameter WIDTH = 8,           // Data width
    parameter REGFILE_DEPTH = 8,   // Number of registers
    parameter REGFILE_ADDR_WIDTH = 3  // log2(REGFILE_DEPTH)
) (
    input  logic clk,
    input  logic rst_n,
    
    // Control signals
    input  logic wr_enable,                              // Write enable for register write
    input  logic [1:0] alu_op,                           // ALU operation select
    
    // Address inputs (which registers to use)
    input  logic [REGFILE_ADDR_WIDTH-1:0] rs1_addr,      // Read source 1 address
    input  logic [REGFILE_ADDR_WIDTH-1:0] rs2_addr,      // Read source 2 address
    input  logic [REGFILE_ADDR_WIDTH-1:0] rd_addr,       // Read destination address
    
    // Data outputs (for debugging/monitoring)
    output logic [WIDTH-1:0] rs1_data,                   // Read data from source 1
    output logic [WIDTH-1:0] rs2_data,                   // Read data from source 2
    output logic [WIDTH-1:0] result                      // ALU result
);

// Instantiate Register File
// This handles storage of all register values
register_file #(
    .WIDTH(WIDTH),
    .DEPTH(REGFILE_DEPTH),
    .ADDR_WIDTH(REGFILE_ADDR_WIDTH)
) rf (
    .clk(clk),
    .rst_n(rst_n),
    .wr_enable(wr_enable),
    .wr_addr(rd_addr),                 // Write address (destination)
    .wr_data(result),                  // What to write (ALU result)
    .rs1_addr(rs1_addr),               // Read address 1
    .rs1_data(rs1_data),               // Read data 1 → goes to ALU
    .rs2_addr(rs2_addr),               // Read address 2
    .rs2_data(rs2_data)                // Read data 2 → goes to ALU
);

// Instantiate ALU
// Takes register values and produces result
alu #(
    .WIDTH(WIDTH)
) alu_inst (
    .operand_a(rs1_data),              // Input: first operand (from register)
    .operand_b(rs2_data),              // Input: second operand (from register)
    .operation(alu_op),                // Input: which operation (00-11)
    .result(result)                    // Output: result back to register
);

endmodule
```

**Module Connections Explained:**

```
Data Flow Path:

rs1_addr → [Register File Mux] → rs1_data ──┐
                                             ├──> [ALU Multiplexer] → result → [Write back]
rs2_addr → [Register File Mux] → rs2_data ──┘

Control Flow Path:

alu_op → [ALU Mux selector]
rd_addr → [Register file write address decoder]
wr_enable → [Register file write control]
clk, rst_n → [Sequential control]
```

---

## **Part 7: Timing Diagram and Signal Behavior**

### **7.1 Complete Instruction Timing**

```
For instruction: R3 = R1 * R1  (b² computation)

Timeline:
│
├─ T0: Clock rising edge
│      │
│      ├─ rs1_addr = 001 (request R1)
│      ├─ rs2_addr = 001 (request R1 - same register twice OK!)
│      ├─ alu_op = 10 (MULTIPLY operation)
│      ├─ rd_addr = 011 (write result to R3)
│      ├─ wr_enable = 1 (enable the write)
│
├─ T0 → T0+t_mux_delay: Register file multiplexer settling
│      │
│      ├─ Address decoder activates R1's output
│      ├─ Multiplexer gates stabilize
│      ├─ rs1_data begins showing R1's value
│      ├─ rs2_data begins showing R1's value
│      │   (both are the same since both read R1)
│
├─ T0+t_mux → T0+t_mux+t_alu: ALU computation phase
│      │
│      ├─ Multiplier receives rs1_data and rs2_data
│      ├─ All 4 operations compute in parallel
│      │  ├─ Adder: computes A+B
│      │  ├─ Subtractor: computes A-B
│      │  ├─ Multiplier: computes A×B = R1×R1 ← We want this!
│      │  └─ Divider: computes A÷B
│      ├─ alu_op=10 selector is stable (MULTIPLY)
│      └─ ALU mux selects multiplication result
│          result = R1 × R1
│
├─ T0+t_mux+t_alu → T1-t_setup: Result settling
│      │
│      ├─ result [7:0] bus stabilizes
│      ├─ All intermediate toggling has stopped
│      └─ Value is stable and ready for capture
│
├─ T1-t_setup: Setup time requirement
│      │
│      ├─ result must be stable NOW
│      ├─ wr_data latch needs stable input
│      └─ wr_enable still high, rd_addr still 011
│
├─ T1: Clock rising edge (next cycle)
│      │
│      ├─ Register file latches: wr_data ← result
│      ├─ R3 now contains R1×R1
│      ├─ Simultaneously in register file:
│      │  ├─ rs1_addr ← new address (next instruction)
│      │  └─ rs1_data ← new register value
│      └─ Next instruction can start
│
├─ T1 → T2: Hold time + next instruction processing
│      │
│      ├─ R3 has the new value R1×R1 internally
│      ├─ But this value isn't visible at output yet!
│      ├─ If next instruction reads R3, it will get OLD value!
│      └─ Next cycle's rs1_data reflects new register contents
│

Key timing constraints:

Setup Time (t_setup):
- Result must be stable before clock edge
- Typical: 0.5 ns for 8-bit operations
- If violated: Metastable state, unpredictable result

Hold Time (t_hold):
- Result must remain stable after clock edge
- Typical: 0.2 ns
- If violated: Previously stored value might be wrong

Propagation Delay (t_prop):
- Time from clock edge to output valid
- Typical: 0.3-0.5 ns
- This is when rs1_data, rs2_data reflect new values
```

---

## **Part 8: Instruction Encoding Introduction**

### **8.1 The Need for Instruction Encoding**

**Problem:** How does the control logic know which signals to set?

```
Currently:
- We manually set: rs1_addr, rs2_addr, rd_addr, alu_op, wr_enable
- This is done externally (test bench or control module)
- Not scalable for real processor

Solution:
- Combine all control signals into a single "instruction" word
- One instruction encodes everything needed for one operation
- Instruction stays the same width regardless of complexity
```

### **8.2 Example Instruction Format**

```
Simple Instruction Format:

[7:6] Operation (alu_op):   2 bits
      00 = ADD
      01 = SUB
      10 = MUL
      11 = DIV

[5:3] Destination Register (rd_addr):  3 bits
      000 = R0
      001 = R1
      ...
      111 = R7

[2:0] Source Register 1 (rs1_addr):    3 bits
      000 = R0
      001 = R1
      ...
      111 = R7

Missing: rs2_addr!
This is a simplified format. Real instructions need both sources.
```

**Better format:**
```
Extended Instruction Format (16 bits total):

[15:14] Operation (alu_op):   2 bits
[13:11] Destination (rd_addr):   3 bits
[10:8]  Source 1 (rs1_addr):     3 bits
[7:5]   Source 2 (rs2_addr):     3 bits
[4:0]   Reserved/Flags:          5 bits (for future use)

Example instruction: R3 = R1 * R1

Binary: 10_011_001_001_00000
       op rd    rs1  rs2  reserved

This single 16-bit value encodes the entire operation!
```

### **8.3 Instruction Decoding**

```verilog
// Instruction decoding logic
logic [15:0] instruction;       // Input: 16-bit instruction
logic [1:0] alu_op;             // Decoded fields
logic [2:0] rd_addr, rs1_addr, rs2_addr;
logic wr_enable;

// Extract fields from instruction
always_comb begin
    // Extract operation: bits [15:14]
    alu_op = instruction[15:14];
    
    // Extract destination: bits [13:11]
    rd_addr = instruction[13:11];
    
    // Extract source 1: bits [10:8]
    rs1_addr = instruction[10:8];
    
    // Extract source 2: bits [7:5]
    rs2_addr = instruction[7:5];
    
    // Write enable: always 1 for this instruction type
    // (other instruction types might not write)
    wr_enable = 1'b1;
end
```

---

## **Summary: ALU in Processor Context**

| Component | Purpose | Timing | Notes |
|-----------|---------|--------|-------|
| **ALU** | Performs arithmetic/logic | Combinational | Fast, no clock needed |
| **Register File** | Storage between operations | Sync write, async read | Enables pipelining |
| **Instruction** | Specifies what to do | Static during execution | Decoded for all control signals |
| **Control Logic** | Decodes instructions | Combinational | Generates control signals |
| **Clock** | Synchronizes operations | Periodic pulse | Determines frequency |

**Complete Execution Cycle:**
1. Instruction specifies: which registers, which operation
2. Register file reads operands combinationally
3. ALU computes result combinationally
4. Clock edge: result written back to register
5. Next instruction begins (new addresses, new operation)

**Critical Issues to Address:**
- How to handle operations taking multiple cycles (division)?
- How to handle branching and conditional execution?
- How to prevent hazards (reading value before it's written)?

These are topics for later lectures.