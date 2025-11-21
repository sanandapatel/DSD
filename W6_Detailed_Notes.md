# Week 6: Memory and Registers in Digital Design
## Comprehensive Lecture Notes with Code Explanations

---

## **Lecture 1: Registers and Register Organization**

### **Core Concepts**

#### **1. Flip-Flop Fundamentals**
A **flip-flop** is the basic building block of sequential logic and storage elements. The **D-type edge-triggered flip-flop** is the most commonly used:

- **Inputs:** Clock (CLK), Data input (D), Clock Enable (optional)
- **Output:** Data output (Q)
- **Operation:** Data at D is captured and transferred to Q only at the active clock edge (rising or falling edge)
- **Always present:** Q always has a valid logic value (0 or 1) - never in a tri-state
- **Synchronization:** The flip-flop synchronizes asynchronous data changes with the clock signal

**Why Clock Enable Instead of Clock Gating?**
- **Clock Gating:** Physically disabling the clock using AND gates introduces extra delay in the clock path
- **Problems:** Makes timing analysis complex and unreliable
- **Solution:** Keep clock uniform across all flip-flops, use clock enable signal to control when data updates occur
- **Advantage:** Clock enable can be irregular - controlled by internal logic without affecting clock distribution

#### **2. Grouping Flip-Flops: The Register Concept**

When multiple flip-flops are organized together, they form a **register** with shared control signals:

```verilog
// Two flip-flops with shared clock and enable
// FF1: D0 input → Q0 output (synchronized by CLK)
// FF2: D1 input → Q1 output (synchronized by CLK)
// Both share: ENABLE signal
// When ENABLE = 1: Both FFs update at clock edge
// When ENABLE = 0: Both FFs retain previous values
```

**Key Advantage:** Grouping flip-flops through enable signals allows precise control over when groups of data get updated simultaneously.

#### **3. Register Banks: Scaling to Multiple Registers**

A **register bank** or **register file** is an array of registers with shared control and address decoding:

**Structure:**
- Multiple N-bit registers stored in an array
- Address decoder to select which register to write to
- Multiplexer to select which register to read from
- Common write enable and clock signals

**Example Configuration:**
```
Depth = 4 registers
Width = 8 bits per register
Address width = log₂(4) = 2 bits

Addresses:
00 → Register 0
01 → Register 1
10 → Register 2
11 → Register 3
```

#### **4. SystemVerilog Register Bank Implementation**

```verilog
module register_bank #(
    parameter WIDTH = 8,      // Width of each register (number of bits)
    parameter DEPTH = 4       // Number of registers in the bank
) (
    input  logic clk,
    input  logic rst_n,
    input  logic wr_enable,                    // Overall write enable for the bank
    input  logic [$clog2(DEPTH)-1:0] wr_addr, // Write address (which register to update)
    input  logic [WIDTH-1:0] data_in,          // Data to write into the register
    input  logic [$clog2(DEPTH)-1:0] rd_addr, // Read address (which register to read from)
    output logic [WIDTH-1:0] data_out          // Data read from the register
);

// Declare the register array
// Array of registers, each WIDTH bits wide, DEPTH registers total
logic [WIDTH-1:0] registers [DEPTH-1:0];

// WRITE OPERATION (Synchronous) - occurs on clock edge
always_ff @(posedge clk or negedge rst_n) begin
    if (~rst_n) begin
        // RESET: Initialize all registers to zero on async reset
        // Loop through each register and clear its value
        for (int i = 0; i < DEPTH; i = i + 1) begin
            registers[i] <= '0;  // '0 means all bits set to 0
        end
    end else if (wr_enable) begin
        // WRITE: Only update the register at the addressed location
        // wr_addr selects which register, data_in contains the new value
        registers[wr_addr] <= data_in;
    end
    // If wr_enable is not asserted, registers retain their previous values
end

// READ OPERATION (Combinational) - happens immediately, no clock wait
// The address directly selects which register value appears at output
// This is asynchronous read - no latency
assign data_out = registers[rd_addr];

endmodule
```

**Important Code Comments:**

- `$clog2(DEPTH)`: Built-in function calculating log₂(depth) rounded up. For depth=4: $clog2(4)=2
  - Tells us we need 2-bit address for 4 registers
- `[WIDTH-1:0]`: Bit range notation - from bit WIDTH-1 down to bit 0
- `always_ff @(posedge clk or negedge rst_n)`: Sequential logic triggered by clock or async reset
- `'0`: Literal value - all bits set to 0 (works for any width)
- `registers[wr_addr] <= data_in`: Non-blocking assignment (proper for synchronous logic)
- `assign data_out = registers[rd_addr]`: Combinational assignment (no delay)

**Synchronous vs Asynchronous Operations:**
- **Write:** Synchronous - new data only appears when CLK edge occurs
- **Read:** Combinational - data immediately available based on address, no wait needed
- **Reset:** Asynchronous - immediate effect when reset asserted, doesn't wait for clock

---

## **Lecture 2: Clock Gating and Design Considerations**

### **Power Consumption in Digital Circuits**

#### **Understanding Power Dissipation**

Power consumption in digital circuits comes from **switching activity**:

1. **Charge and Discharge Cycles:**
   - When a gate output switches from 0→1: Charge accumulates on load capacitance from power supply
   - When output switches from 1→0: Charge is discharged to ground
   - Each cycle requires current flow: Energy = ½CV²Δf (proportional to frequency and switching)

2. **The Clock Tree Problem:**
   - Clock signal is **always switching** (0→1→0→1...)
   - Clock connects to **every single flip-flop** in the design
   - Clock tree accounts for **significant portion** of total chip power consumption
   - Dominates power even when design is idle (enable=0)

#### **Clock Gating Technique**

**Goal:** Reduce power by preventing clock from toggling when no updates needed

**Implementation (NOT Recommended for this course):**
```verilog
// Bad approach: Using AND gate to gate the clock
assign gated_clock = clock AND enable;
```

**Problems:**
- Introduces **additional delay** in clock path
- Makes **timing analysis** very complex and unreliable
- Difficult to predict exactly how each flip-flop's timing is affected
- Can cause **setup/hold violations** due to increased clock skew

#### **Preferred Approach: Enable-based Control**

```verilog
// GOOD approach: Clock always runs uniformly, enable controls updates
always_ff @(posedge clk) begin
    if (enable) begin
        // Update only when enabled
        state <= next_state;
    end
    // When enable=0, state retains previous value
end
```

**Advantages:**
- Clock distribution remains **clean and uniform**
- All flip-flops see **same clock timing**
- Enable signal can be **controlled by any logic** without affecting clock tree
- Much easier to **analyze timing**
- Industry-standard approach

#### **Clock Tree Distribution**

The clock signal must reach all flip-flops in the design with minimal **skew** (delay differences):

**Tree Structure:**
```
        Clock Input
            |
        /---+---\
       /         \
    Buffer1      Buffer2
     / \          / \
   FF1 FF2      FF3 FF4
```

**Optimization Techniques:**
- **Clock Tree Balancing:** Make all paths from source to flip-flops roughly equal length
- **Register Placement:** Position flip-flops to minimize clock distribution distance
- **Clock Re-routing:** Tools automatically optimize clock tree during place and route

---

## **Lecture 3: Random Access Memory (RAM) Fundamentals**

### **What is RAM?**

#### **Definition**
**Random Access Memory** = Any memory where any location can be accessed in **constant time**, regardless of which address is requested.

#### **Historical Context**
Before RAM, memory was stored on **tapes:**
- Could only access data sequentially
- To read byte at position 1000, had to scan through all previous bytes
- Access time varied greatly depending on location

**RAM Advantage:** Access any address in same amount of time

#### **Key Distinction: ROM vs RAM**
Common misconception: People think ROM and RAM are opposites
- **Reality:** Both are forms of random access memory!
- **ROM:** Random Access **Read-Only** Memory (can't write)
- **RAM:** Random Access **Read-Write** Memory (can read and write)
- **The key is "Random Access"**, not whether it's readable or writable

#### **Register Bank as RAM**

A register bank is actually a simple form of RAM:

```
Register Bank Architecture:
├─ Memory Array: Stores data in registers
├─ Write Path:
│  ├─ Decoder: Converts address to enable signal for specific register
│  ├─ Data In: Value to be written
│  └─ Write Enable: Controls whether writing happens
└─ Read Path:
   ├─ Multiplexer: Selects which register's data to output
   └─ Address: Controls which register is selected
```

**Why It's Random Access:**
- Doesn't matter which of 4 registers is accessed
- Path through decoder = constant delay
- Path through multiplexer = constant delay
- Total access time is **independent of address**

#### **Contrast: Sequential vs Random Access**

```
Sequential Memory (Tape):
Address 1: 1 unit time to reach
Address 100: ~100 units time to reach (must scan through all previous data)
Address 1000: ~1000 units time to reach
Problem: Access time depends on location!

Random Access Memory (Register Bank):
Address 1: ~1 unit time
Address 100: ~1 unit time (same decoder/mux delay)
Address 1000: ~1 unit time (same decoder/mux delay)
Benefit: Predictable, uniform access time!
```

### **RAM Configuration Parameters**

**Typical RAM Specifications:**

1. **Depth:** Number of addressable locations
   - Example: 1K (1024 locations), 2K (2048 locations)
   
2. **Width:** Number of bits in each word
   - Example: 8-bit word, 16-bit word, 32-bit word

3. **Common Configurations:**
   - 1K × 8: 1024 locations, 8 bits each = 8,192 total bits
   - 2K × 16: 2048 locations, 16 bits each = 32,768 total bits
   - 4K × 32: 4096 locations, 32 bits each = 131,072 total bits

**Address Width Calculation:**
- For 1K (1024) locations: Need log₂(1024) = 10 bits
- For 4K (4096) locations: Need log₂(4096) = 12 bits

---

## **Lecture 4: FPGA Memory Resources**

### **Types of Memory in FPGAs**

Modern FPGAs offer multiple memory solutions with different characteristics:

#### **1. Distributed RAM (LUT-RAM)**

**Implementation:**
- Uses **lookup tables (LUTs)** as small memory blocks
- Each LUT can be configured as both logic and storage

**Capacity:**
- Single LUT with 6 inputs: 2⁶ = 64 bits storage
- Can be combined for larger memories

**Structure:**
```
LUT = Truth Table (Logic Function)
↓
Can also be viewed as Storage Array

Example: 4-input LUT
Inputs: I3, I2, I1, I0
Output: Y

Truth Table interpretation:
I3 I2 I1 I0 | Y
0  0  0  0  | 0  (stored value at address 0)
0  0  0  1  | 1  (stored value at address 1)
0  0  1  0  | 0  (stored value at address 2)
...
1  1  1  1  | 1  (stored value at address 15)

By changing "how we interpret" the LUT:
- Logic mode: Combinational function
- Memory mode: Read-only storage with 4-bit address, 1-bit output
```

**Advantages:**
- **Very fast** access time (combinational read in one cycle)
- **Distributed** throughout FPGA (no centralized location)
- **No latency:** Data output immediately upon address change

**Disadvantages:**
- **Small capacity** per LUT
- Best for small memories only (tens of bits)
- Limited applicability for large storage

**Best Use Cases:**
- Small shift registers
- Small buffers
- Small lookup tables
- Low-latency storage needs

#### **2. Block RAM (BRAM)**

**Description:**
- Dedicated memory blocks built into FPGA fabric
- **Most common** form of RAM in modern FPGAs
- Typically **1K × 18** or **1K × 36** bit configuration

**Characteristics:**

```
BRAM Capabilities:
├─ True Dual-Port: Can read from one address and write to another SIMULTANEOUSLY
├─ Synchronous Operation: All operations synchronized to clock
│  ├─ Writes: Data stored on clock edge only
│  └─ Reads: Data output valid after 1 clock cycle (registered output)
├─ Configurable Aspect Ratio:
│  ├─ Deep and Narrow: 1K × 8 (1024 locations, 8 bits each)
│  ├─ Shallow and Wide: 512 × 32 (512 locations, 32 bits each)
│  └─ Combinations: 2K × 16, 4K × 8, etc.
└─ Initial Values: Can use $readmemh/$readmemb in simulation and synthesis

Typical Sizes Across Vendors:
- Xilinx Artix-7: 36 Kb per BRAM (1024 × 36 or equivalent)
- Altera MAX 10: 10 Kb per embedded memory (1024 × 10 or equivalent)
```

**Key Difference: Synchronous Read**
```verilog
// In Register Bank (LUT-RAM): Combinational Read
assign data_out = registers[address];  // Immediate!

// In Block RAM: Synchronous Read (1-cycle latency)
always_ff @(posedge clk) begin
    data_out <= memory[address];  // Available NEXT cycle
end
```

**Why 1-Cycle Read Latency?**
- Allows timing to be relaxed
- Frees up time between address presentation and data return
- Trade-off: Latency for better timing margin

#### **3. Ultra RAM**

- More dense version of Block RAM
- Specialized for very large memory arrays
- Higher capacity, some restrictions on usage
- Not covered in detail for this course

### **SystemVerilog BRAM Implementation**

```verilog
module bram_sync #(
    parameter WIDTH = 32,      // Data width: number of bits per word
    parameter DEPTH = 1024,    // Number of words in memory
    parameter INIT_FILE = ""   // Path to initialization file (optional)
) (
    input  logic clk,
    input  logic rst_n,
    
    // Write Port A
    input  logic wr_en_a,                          // Write enable for port A
    input  logic [$clog2(DEPTH)-1:0] wr_addr_a,   // Write address for port A
    input  logic [WIDTH-1:0] wr_data_a,            // Data to write to port A
    
    // Read Port B (simultaneous read while writing)
    input  logic [$clog2(DEPTH)-1:0] rd_addr_b,   // Read address for port B
    output logic [WIDTH-1:0] rd_data_b             // Data read from port B
);

// Memory array declaration
logic [WIDTH-1:0] mem [DEPTH-1:0];

// Optional: Initialize from file
initial begin
    if (INIT_FILE != "") begin
        // Load memory contents from initialization file
        // Format: $readmemh("data.mem", mem);
        // Data file format example:
        // Address Value
        // 0000 DEADBEEF
        // 0001 CAFEBABE
        // etc.
    end
end

// SYNCHRONOUS WRITE (Port A)
always_ff @(posedge clk or negedge rst_n) begin
    if (~rst_n) begin
        // Typically BRAMs don't have reset for data
        // (data preservation is often desired)
        // Only reset control signals if needed
    end else if (wr_en_a) begin
        // On clock edge, if write enabled, store data at specified address
        mem[wr_addr_a] <= wr_data_a;
    end
end

// SYNCHRONOUS READ (Port B)
// Data is registered, available after next clock cycle
always_ff @(posedge clk) begin
    // Read address is sampled on clock edge
    // Data appears at output on the NEXT clock edge
    rd_data_b <= mem[rd_addr_b];
end

endmodule
```

**Code Explanation:**

- **True Dual Port:** Port A writes while Port B reads simultaneously
- **Synchronous Write:** Data stored only on rising clock edge
- **Synchronous Read:** Output updated on rising clock edge
  - Write address/data: sampled on clock
  - Data output: available on NEXT clock (1-cycle latency)
- **$readmemh:** Load hex values from file (Verilog system task)

**Memory File Format (.mem):**
```
// Hex values, one per line or space-separated
@0000           // @ specifies address (optional)
DEADBEEF
CAFEBABE
12345678

// Or sparse format:
@0000 DEADBEEF  // Address 0x0000 = 0xDEADBEEF
@0004 CAFEBABE  // Address 0x0004 = 0xCAFEBABE
```

---

## **Lecture 5: FIFO Buffers**

### **FIFO Concept: First-In-First-Out**

#### **Logical Definition**

A **FIFO** is a data queue where:
1. Data enters from one end (input port)
2. Data exits from the other end (output port)
3. **FIFO Property:** First data in = First data out

**Simple Analogy:**
```
Queue of patients at hospital:
Patient A arrives → joins queue
Patient B arrives → joins queue after A
Patient C arrives → joins queue after B

Service (data removal):
Patient A leaves first (was first in)
Then Patient B leaves (was second in)
Then Patient C leaves (was third in)

Result: FIFO behavior!
```

#### **Why FIFOs Are Useful**

1. **Clock Domain Crossing:**
   - Read with one clock, write with another clock
   - Implicitly handles synchronization between different clock domains
   - Prevents metastability issues

2. **Data Buffering:**
   - Temporary storage when production ≠ consumption rate
   - Producer can write in bursts, consumer reads continuously

3. **Rate Matching (Speed Adaptation):**
   - Producer writes at high speed in bursts
   - Consumer reads at steady, slower speed
   - FIFO bridges the speed gap

#### **Real-World Example: UART Buffering**

```
Scenario:
- CPU: 100+ MHz operation
- UART: 9600 bits/second transmission

Problem without FIFO:
- At 100 MHz, CPU could transmit 1 bit in thousands of cycles
- Can't "sit around" waiting for UART to transmit bit-by-bit
- Huge waste of CPU cycles

Solution with FIFO:
- CPU dumps entire message into FIFO buffer at CPU speed (microseconds)
- UART reads from FIFO and transmits at its own speed (milliseconds per character)
- CPU freed up to do other work immediately
```

#### **Video Buffering Example**

```
Scenario:
- Network: Bursty, sends data in chunks
- Video player: Needs smooth, continuous stream

Without FIFO:
- Gets 10 MB of data, can watch for few seconds
- Then network congestion, no data arriving
- Buffer empty → buffering wheel appears

With FIFO:
- When connection open, network floods FIFO with 50+ MB
- Video player reads from FIFO continuously
- Even if network becomes congested, enough data buffered to continue
- Won't rebuffer unless FIFO completely empties
```

### **FIFO Implementation**

#### **Architecture**

```
FIFO Structure:

        Input Side          Memory Array          Output Side
        (Write Port)        (Storage)            (Read Port)
        
        wr_data                                   rd_data
           |                                         ^
           v                                         |
        +------+    Write Pointer    Read Pointer  +------+
        |  wr  |-->  Register (ptr) + Register  -->|  rd  |
        | ctrl |      (auto-increment)             | ctrl |
        +------+         |    ^                     +------+
           |             v    |
           |         +--------+--------+
           +-------->|   Memory Array  |<--------+
           wr_en     | (Dual Port RAM) | rd_en
                     +--------+--------+
                              ^
                              |
                    Status Flags:
                    - empty flag
                    - full flag
                    - valid flag
```

**Key Components:**

1. **Memory Array:** Circular buffer to store data
2. **Write Pointer:** Tracks where next write will occur
3. **Read Pointer:** Tracks where next read will occur
4. **Status Flags:**
   - **Full:** Write pointer wrapped around to read pointer
   - **Empty:** Read pointer reached write pointer
   - **Valid:** Data available for reading

**Circular Buffer Concept:**
```
FIFO with 4 locations (indices 0-3):

Initial state (empty):
wr_ptr = rd_ptr = 0
+---+---+---+---+
| 0 | 1 | 2 | 3 |  (empty)
+---+---+---+---+
 ↑
wr_ptr, rd_ptr

After writing 3 values (A, B, C):
wr_ptr = 3, rd_ptr = 0
+---+---+---+---+
| A | B | C | - |
+---+---+---+---+
 ↑       ↑
 |       wr_ptr
 rd_ptr

After reading 2 values (A, B):
wr_ptr = 3, rd_ptr = 2
+---+---+---+---+
| A | B | C | - |
+---+---+---+---+
     ↑   ↑
    rd_ptr (read C next)
         wr_ptr

Write wraps around (write D at address 0):
wr_ptr = 0, rd_ptr = 2
+---+---+---+---+
| D | B | C | - |  (now FULL)
+---+---+---+---+
 ↑       ↑
 (wrapped) (still reading)
```

#### **Pointer Management**

To detect full vs empty with same pointer values:

```verilog
// Circular counter wraps around:
// Counter range: 0 to MAX_PTR
// MAX_PTR = DEPTH (same as address bits)

// Full condition: pointers equal AND MSB different
always @(*) begin
    full = (wr_ptr_inc == rd_ptr);  // Would wrap to same address
end

// Empty condition: pointers exactly equal
always @(*) begin
    empty = (wr_ptr == rd_ptr);  // At same location, MSB same
end

// In practice: Often use extra bit to distinguish
// wr_ptr[PTR_WIDTH] vs rd_ptr[PTR_WIDTH] (MSB different = full)
```

#### **FIFO Verilog Template**

```verilog
module fifo_sync #(
    parameter WIDTH = 32,      // Data width
    parameter DEPTH = 16       // Number of entries (should be power of 2)
) (
    input  logic clk,
    input  logic rst_n,
    
    // Write Interface
    input  logic [WIDTH-1:0] wr_data,
    input  logic wr_en,                    // Assert to write data
    output logic full,                     // Asserted when FIFO cannot accept more data
    
    // Read Interface
    output logic [WIDTH-1:0] rd_data,
    input  logic rd_en,                    // Assert to read data
    output logic empty,                    // Asserted when FIFO has no data
    output logic valid                     // Asserted when rd_data is valid
);

// Pointer width needs extra bit to distinguish full from empty
localparam PTR_WIDTH = $clog2(DEPTH) + 1;

logic [PTR_WIDTH-1:0] wr_ptr, rd_ptr;     // Write and Read pointers
logic [WIDTH-1:0] mem [DEPTH-1:0];        // Memory array

// Calculate next pointer value (wraps around circularly)
logic [PTR_WIDTH-1:0] wr_ptr_next, rd_ptr_next;
assign wr_ptr_next = wr_ptr + 1;          // Next write location
assign rd_ptr_next = rd_ptr + 1;          // Next read location

// WRITE OPERATION (Synchronous)
always_ff @(posedge clk or negedge rst_n) begin
    if (~rst_n) begin
        wr_ptr <= '0;                       // Reset pointer to 0
    end else if (wr_en && !full) begin
        mem[wr_ptr[$clog2(DEPTH)-1:0]] <= wr_data;  // Store at lower PTR_WIDTH bits
        wr_ptr <= wr_ptr_next;              // Increment pointer
    end
end

// READ OPERATION (Synchronous)
always_ff @(posedge clk or negedge rst_n) begin
    if (~rst_n) begin
        rd_ptr <= '0;                       // Reset pointer to 0
        valid <= '0;                        // No valid data initially
    end else if (rd_en && !empty) begin
        rd_ptr <= rd_ptr_next;              // Increment pointer
        valid <= 1'b1;                      // Mark data as valid
        rd_data <= mem[rd_ptr_next[$clog2(DEPTH)-1:0]];  // Output next value
    end else begin
        valid <= 1'b0;                      // No new data this cycle
    end
end

// STATUS FLAGS (Combinational)
// Full: MSB different AND lower bits equal
//   wr_ptr = 1_0101, rd_ptr = 0_0101 → MSB: 1≠0 (different), Lower: 0101=0101 (same)
//   This means write wrapped around to same location as read → FULL
assign full = (wr_ptr[PTR_WIDTH-1] != rd_ptr[PTR_WIDTH-1]) && 
              (wr_ptr[$clog2(DEPTH)-1:0] == rd_ptr[$clog2(DEPTH)-1:0]);

// Empty: Pointers completely equal (including MSB)
assign empty = (wr_ptr == rd_ptr);

endmodule
```

**Code Comments Explained:**

- `PTR_WIDTH = $clog2(DEPTH) + 1`: Extra bit to distinguish full/empty states
  - Example: DEPTH=16 needs 4 bits for address, but 5 bits for pointer (includes wrap-around bit)
  
- `wr_ptr[PTR_WIDTH-1]` vs `rd_ptr[PTR_WIDTH-1]`: MSB compares wrap-around bits
  - When MSB differs but lower bits same → wrapped around → FULL
  - When all bits same → Empty
  
- `wr_ptr[$clog2(DEPTH)-1:0]`: Extract only address bits (mask off MSB)
  - Used to index into memory array

**Timing Behavior:**

```
Cycle 0: wr_en=1, rd_en=0
        wr_ptr becomes 1
        data_in stored at mem[0]

Cycle 1: wr_en=1, rd_en=1
        wr_ptr becomes 2 (writes mem[1])
        rd_ptr becomes 1 (reads from mem[1])  
        rd_data shows mem[1] with 1-cycle latency

Cycle 2: rd_en=1
        rd_ptr becomes 2
        rd_data shows mem[2]

When empty: rd_en=0 even if rd_en asserted
When full: wr_data ignored even if wr_en asserted
```

---

## **Summary: Memory Hierarchy in Digital Design**

| **Component** | **Size** | **Speed** | **Access** | **Use Case** |
|--------------|----------|-----------|-----------|------------|
| Flip-Flops | 1 bit each | Fastest | Combinational | State variables |
| Register Bank (LUT-RAM) | Tens of bits | Very Fast | 1-cycle | Small buffers, shift registers |
| Block RAM | 1K-36K bits | Fast | 1-cycle | Data storage, FIFOs |
| SRAM (discrete) | Megabytes | Medium | Multiple cycles | Large caches |
| DRAM | Gigabytes | Slow | Many cycles | Main memory (PC/Phone) |
| Flash/SSD | Terabytes | Very Slow | Milliseconds | Non-volatile storage |

---

## **Key Takeaways**

1. **Registers** = Groups of flip-flops with shared control
2. **Register Banks** = Arrays of registers with address decoding (simplest form of RAM)
3. **RAM** = Random Access (constant time regardless of address, NOT about read/write capability)
4. **FPGA Memory Options:**
   - Distributed RAM: Fast, small, distributed
   - Block RAM: Larger, synchronous, slower than LUT-RAM
5. **FIFO Buffers** = Circular queues for rate matching and clock domain crossing
6. **Synchronous Design:** Write on clock edges, read combinationally or registered
7. **Enable-based Control:** Preferred over clock gating for power management