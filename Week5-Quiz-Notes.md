# Week 5 Digital System Design - Quiz Notes

## 1. Hierarchical Design

### Definition
- Breaking down a system into components with clear interfaces
- Enables managing complexity in large systems

### Design Hierarchy Levels
1. **System Level**: SOC (System on Chip)
2. **Subsystem Level**: CPU, GPU, Memory Controller, Peripherals
3. **Module Level**: ALU, Cache, Register File, Branch Predictor
4. **Component Level**: Adders, Registers, Flip-flops

### Top-Down vs Bottom-Up Design

**Top-Down Flow:**
- Start with system specification
- Decompose into smaller subsystems
- Define interfaces between components
- Implement individual components

**Bottom-Up Flow:**
- Design basic building blocks first
- Combine into larger modules
- Use reusable IP blocks
- Integrate to create final system

**Meet in the Middle:**
- Most practical approach
- Teams work on different levels simultaneously
- Converge through well-defined interfaces

### Interface Definition
- Clear specification of inputs, outputs, functionality
- Example: Adder interface
  - Inputs: Two 8-bit inputs
  - Outputs: 8-bit sum, 1-bit carry/overflow
  - May need: clock, reset, start, done signals

### Abstraction Levels
- **System Level**: Overall architecture
- **Algorithm Level**: Different implementation algorithms
- **RTL Level**: Register transfers and sequential steps
- **Gate Level**: Actual hardware implementation

---

## 2. Parallel Multiplier Design

### Concept
- Multiplication using partial products (like manual multiplication)
- All partial products generated simultaneously
- Added together using an adder tree

### Example: 456 × 123 (Decimal)
```
    456
  × 123
  -----
   1368  (456 × 3)    PP0
  9120   (456 × 2, shifted)  PP1
 45600   (456 × 1, shifted)  PP2
-------
 56088
```

### Binary Implementation (4-bit example: 1010 × 0101)
- Each partial product = multiplicand AND multiplier bit
- Shift left by position
- Add all partial products

### Hierarchical Breakdown
1. **Partial Product Generator**
   - Generates n partial products for n-bit multiplier
   - Each PP[i] = multiplicand × multiplier[i]
   - Handles shifting automatically

2. **Adder Tree**
   - Combines partial products in stages
   - Stage 1: n inputs → n/2 outputs
   - Stage 2: n/2 inputs → n/4 outputs
   - Continue until single result

### Verilog Code Structure

**Module Definition (Parameterized):**
```verilog
module parallel_multiplier #(parameter WIDTH = 8) (
    input [WIDTH-1:0] a, b,
    output [2*WIDTH-1:0] product
);
```

**Partial Product Generator:**
```verilog
// Using generate for loop
genvar i;
generate
    for (i = 0; i < WIDTH; i = i + 1) begin
        assign partial_products[i] = a & {WIDTH{b[i]}};
        // Shift handled by indexing
    end
endgenerate

// Handle signed: MSB uses subtraction
assign partial_products[WIDTH-1] = -1 * (a & {WIDTH{b[WIDTH-1]}});
```

**Adder Tree Implementation:**
```verilog
// Initialize stage sums
for (i = 0; i < WIDTH; i = i + 1) begin
    stage_sums[0][i] = partial_products[i];
end

// Tree stages
for (stage = 1; stage <= $clog2(WIDTH); stage++) begin
    for (i = 0; i < WIDTH/(2<<stage); i++) begin
        stage_sums[stage][i] = stage_sums[stage-1][2*i] + 
                                stage_sums[stage-1][2*i+1];
    end
end

assign product = stage_sums[final_stage][0];
```

### Characteristics
- **Purely combinational** (no clock, no registers)
- **Fast**: Single clock cycle (if clocked externally)
- **Resource intensive**: O(n²) area complexity
- **Width**: n-bit × n-bit = 2n-bit output

---

## 3. FSM + Data Path Architecture (Control + Data Path)

### Concept
- **Data Path**: Performs computations (adders, multipliers, ALUs)
  - Often combinational
  - Output depends on current inputs
  - Examples: Adder, Multiplier, ALU

- **Control Logic (FSM)**: Manages when/how operations occur
  - Tells data path when to operate
  - Selects which inputs to use
  - Detects when computation is complete

### Benefits of Separation
1. **Modularity**: Independent design and optimization
2. **Reusability**: Data path components reusable across designs
3. **Independent Optimization**: 
   - Control: State encoding, FSM optimization
   - Data path: Timing, power, area optimization
4. **Independent Verification**: Test modules separately

---

## 4. Sequential Multiplier Design

### Concept
- Iterative approach: computes over multiple clock cycles
- Uses **one adder + registers** (resource sharing)
- Takes n cycles for n-bit multiplication

### Operation
1. Initialize product register to 0
2. For each bit of multiplier (n iterations):
   - If multiplier bit = 1: Add multiplicand to product
   - Shift multiplicand left (or shift multiplier right)
3. After n cycles: product is ready

### State Machine States
- **IDLE**: Waiting for start signal
- **MULTIPLY**: Performing multiplication (n cycles)
  - Conditional addition based on multiplier bit
  - Unconditional shifting
- **DONE**: Assert done signal, return to IDLE

### Control Signals
- **Inputs**:
  - `start`: External signal to begin
  - `clk`, `reset`: Clock and reset
  
- **Internal Control Signals**:
  - `clear_product`: Reset product register
  - `add_multiplicand`: Enable addition
  - `shift_multiplier`: Shift multiplier right
  
- **Outputs**:
  - `done`: Indicates completion

### Timing Diagram
```
Clock:      __|‾|_|‾|_|‾|_|‾|_|‾|_
State:      IDLE | MULTIPLY (n cycles) | DONE | IDLE
Start:      _____|‾‾|_____________________
Add:        _______|?|?|?|?|_____________  (depends on bits)
Shift:      _______|‾‾‾‾‾‾‾|_____________
Done:       ______________________|‾|____
```

### Verilog FSM Structure

**State Declaration:**
```verilog
typedef enum logic [2:0] {
    IDLE = 3'b000,
    INIT = 3'b001,
    MULTIPLY = 3'b010,
    SHIFT = 3'b011,
    DONE = 3'b100
} state_t;

state_t current_state, next_state;
```

**State Transition (Sequential Logic):**
```verilog
always_ff @(posedge clk or posedge reset) begin
    if (reset)
        current_state <= IDLE;
    else
        current_state <= next_state;
end
```

**Next State Logic (Combinational):**
```verilog
always_comb begin
    case (current_state)
        IDLE: 
            next_state = start ? INIT : IDLE;
        INIT:
            next_state = MULTIPLY;
        MULTIPLY:
            next_state = (counter == WIDTH-1) ? DONE : MULTIPLY;
        DONE:
            next_state = IDLE;
        default:
            next_state = IDLE;
    endcase
end
```

**Output Logic (Mealy or Moore):**
```verilog
// Mealy: Outputs depend on state AND inputs
always_comb begin
    add_multiplicand = (current_state == MULTIPLY) && multiplier[0];
    shift_multiplier = (current_state == MULTIPLY);
    clear_product = (current_state == INIT);
    done = (current_state == DONE);
end
```

### Data Path Components
1. **Multiplicand Register**: Stores multiplicand
2. **Multiplier Register**: Stores multiplier, shifts right
3. **Product Register**: Accumulates partial product
4. **Adder**: Adds multiplicand to product when enabled
5. **Counter**: Tracks number of iterations

### Mealy vs Moore
- **Conditional addition**: Mealy-like (depends on multiplier bit)
- **Unconditional shift**: Moore-like (depends only on state)

---

## 5. Design Trade-offs

### Parallel Multiplier
**Advantages:**
- Fast: 1 cycle (combinational)
- Simple control (no FSM needed)

**Disadvantages:**
- Large area: O(n²) for n-bit multiplication
- High power consumption
- Long combinational path (affects max clock frequency)

### Sequential Multiplier
**Advantages:**
- Small area: O(n) - single adder + registers
- Lower power per operation
- Shorter critical path → Higher max clock frequency possible

**Disadvantages:**
- Slow: n cycles for n-bit multiplication
- Requires FSM (control overhead)
- Lower throughput if used continuously

### Resource vs Time Trade-off
- **Parallel**: More resources, less time
- **Sequential**: Fewer resources, more time
- Choice depends on:
  - Available area (FPGA/ASIC size)
  - Speed requirements
  - Power budget
  - Cost constraints

---

## 6. Multi-Objective Optimization

### Optimization Definition
- Cost function to optimize (minimize or maximize)
- Subject to constraints (inequalities)
- Find optimal value of design parameters

### Design Objectives
1. **Area**: Resource utilization (LUTs, flip-flops)
2. **Speed**: Clock frequency, latency (cycles)
3. **Power**: Dynamic and static power consumption
4. **Cost**: Manufacturing/component cost

### Design Space Exploration
- **Architecture Selection**: Parallel vs Sequential, Pipeline stages
- **Resource Sharing**: Reuse hardware across time
- **Pipelining**: Add registers to break long paths
- **Clock Frequency**: Balance speed vs power

### Pareto Optimality
**Scenario: Three solutions with Area, Time, Power**

| Solution | Area | Time | Power |
|----------|------|------|-------|
| 1        | 20   | 30   | 20    |
| 2        | 40   | 50   | 30    |
| 3        | 30   | 10   | 25    |

**Analysis:**
- Solution 2 is **dominated** by Solution 1 (worse in all metrics)
- Solutions 1 and 3 are **Pareto optimal** (trade-offs exist)
  - Solution 1: Better area and power
  - Solution 3: Better time
- Cannot choose between 1 and 3 without priorities

### Trade-off Principles
- Cannot simultaneously optimize all objectives
- Designer must prioritize based on application
- Some solutions strictly dominate others (eliminate those)
- Remaining solutions form Pareto frontier

---

## 7. Synthesis Process

### Definition
- Converts RTL (Verilog/VHDL) to gate-level netlist
- Maps to technology-specific primitives (LUTs, flip-flops)

### Synthesis Steps
1. **Parsing**: Convert text to internal representation
2. **High-Level Optimization**: 
   - Boolean optimization
   - FSM state optimization
   - Dead code elimination
3. **Technology Mapping**: 
   - Map to FPGA LUTs (2-input to 6-input)
   - Or map to ASIC standard cells (NAND, NOR, etc.)
4. **Timing Analysis**: Calculate delays
5. **Timing Optimization**: Meet timing constraints
6. **Output Generation**: Netlist file

### Yosys Tool
- Open-source synthesis tool
- Command: `synth_xilinx` for Xilinx FPGAs
- Generates verbose output with:
  - RTLIL intermediate representation
  - FSM extraction and optimization
  - Resource utilization statistics
  - Timing analysis

### Synthesis Output Statistics

**For Parallel Multiplier (32-bit):**
```
Wires: 2,800
Wire bits: 10,800
LUT2: 24
LUT6: 652
Estimated LCs: ~4,000
Timing: 3.3 ns (combinational delay)
```

**For Sequential Multiplier (32-bit):**
```
Flip-flops (FD): 114
  - 64 for product register
  - 32 for multiplicand
  - 7 for FSM state
  - Others for control signals
LUTs: Fewer than parallel
Estimated LCs: ~113
Timing: Shorter critical path
```

### FSM Optimization
- Synthesis tool detects FSMs automatically
- Performs **state encoding optimization**
- Example: Converts sequential encoding to **one-hot encoding**
  - State 0: `0000001`
  - State 1: `0000010`
  - State 2: `0000100`
  - etc.
- One-hot faster but uses more flip-flops (acceptable in FPGAs)

### Timing Analysis
- **Critical Path**: Longest delay path
- **Setup Time**: Data must be stable before clock edge
- **Clock-to-Q (TCQ)**: Flip-flop output delay
- For combinational circuits:
  - Reports maximum delay from input to output
  - May warn "no timing arcs" (no flip-flops)

---

## 8. Verification Strategy

### Unit Testing
- Test individual modules independently
- Example: Separate testbenches for:
  - Partial product generator
  - Adder tree
  - Complete multiplier

### Integration Testing
- Test combined modules
- Verify interfaces between components
- Hierarchical testbench structure

### Testbench Structure
```verilog
module multiplier_tb;
    parameter WIDTH = 8;
    parameter N_TESTS = 100;
    
    logic [WIDTH-1:0] a, b;
    logic [2*WIDTH-1:0] product, expected;
    integer error_count = 0;
    
    // Instantiate DUT
    parallel_multiplier #(.WIDTH(WIDTH)) dut (
        .a(a), .b(b), .product(product)
    );
    
    initial begin
        for (int i = 0; i < N_TESTS; i++) begin
            // Generate random inputs
            a = $urandom();
            b = $urandom();
            
            #10; // Wait for combinational delay
            
            // Check result
            expected = a * b; // Use Verilog's * operator
            if (product !== expected) begin
                $display("ERROR: %d * %d = %d, expected %d", 
                         a, b, product, expected);
                error_count++;
            end
        end
        
        if (error_count == 0)
            $display("PASSED all %d tests", N_TESTS);
        else
            $display("FAILED with %d errors", error_count);
            
        $finish;
    end
endmodule
```

### Coverage-Driven Verification
- Measure how much code is exercised by tests
- Types:
  - **Line Coverage**: Each line executed
  - **Branch Coverage**: All branches taken
  - **FSM Coverage**: All states and transitions visited

### Simulation Commands
```bash
# Iverilog simulation
iverilog -g2012 multiplier.v testbench.v -o sim.out
./sim.out

# Yosys synthesis
yosys -p "read_verilog multiplier.v; synth_xilinx; write_verilog -noattr netlist.v"
```

---

## 9. Verilog Coding Constructs

### Generate Statements
```verilog
// Used for replicating hardware structures
genvar i;
generate
    for (i = 0; i < WIDTH; i = i + 1) begin : gen_partial_products
        assign partial_products[i] = a & {WIDTH{b[i]}};
    end
endgenerate
```
- **Compile-time unrolling**: Loop executed during synthesis
- Creates multiple instances of hardware
- Not a runtime loop!

### For Loops in Always Blocks
```verilog
// Behavioral loop (unrolled at synthesis)
always_comb begin
    for (int i = 0; i < WIDTH; i++) begin
        stage_sums[0][i] = partial_products[i];
    end
end
```
- Also unrolled at synthesis
- Creates parallel hardware, not sequential

### Parameterization
```verilog
module multiplier #(
    parameter WIDTH = 8
) (
    input [WIDTH-1:0] a, b,
    output [2*WIDTH-1:0] product
);
    // Module uses WIDTH throughout
endmodule

// Instantiation with different width
multiplier #(.WIDTH(16)) mult_16bit (
    .a(input_a), .b(input_b), .product(result)
);
```

### Array Declarations
```verilog
// 2D array for partial products
logic [2*WIDTH-1:0] partial_products [0:WIDTH-1];

// Accessing: partial_products[i] gives 2*WIDTH-bit value

// Alternative syntax (may have compatibility issues)
logic [0:WIDTH-1][2*WIDTH-1:0] partial_products_alt;
```

### Signed Number Handling
- Use `signed` keyword: `logic signed [WIDTH-1:0] a;`
- Two's complement representation
- MSB handled specially in multiplication
- Example: `assign pp[WIDTH-1] = -1 * (a & {WIDTH{b[WIDTH-1]}});`

---

## 10. Key Concepts Summary

### Design Patterns
1. **Hierarchical Design**: Break into manageable components
2. **FSM + Data Path**: Separate control from computation
3. **Parameterization**: Reusable, scalable designs

### Implementation Styles
1. **Combinational**: No state, immediate output
2. **Sequential**: Uses registers, multi-cycle
3. **Pipelined**: Registers break long paths (not covered here)

### Verilog Constructs
- `generate` for hardware replication
- `always_ff` for sequential logic (registers)
- `always_comb` for combinational logic
- `case` statements for FSM next-state logic
- Parameters for generic designs

### Synthesis Understanding
- RTL → Gates transformation
- FSM automatic detection and optimization
- Resource utilization: LUTs, FFs, DSPs
- Timing analysis: Critical path, setup/hold

### Verification Best Practices
- Unit tests for modules
- Integration tests for system
- Random test vectors
- Self-checking testbenches
- Coverage analysis

---

## Quick Reference Tables

### FSM Types
| Type | Output Depends On | Advantage | Disadvantage |
|------|-------------------|-----------|--------------|
| Moore | Current state only | Glitch-free | May need extra states |
| Mealy | State + inputs | Fewer states | Potential glitches |

### Multiplier Comparison
| Metric | Parallel | Sequential |
|--------|----------|------------|
| Cycles | 1 | n |
| Area | O(n²) | O(n) |
| Registers | 0 | Many (product, multiplicand, etc.) |
| Control | None | FSM required |
| Max Frequency | Lower (long comb path) | Higher (short paths) |

### Synthesis Tool Commands (Yosys)
```bash
# Read Verilog
read_verilog design.v

# Synthesize for Xilinx
synth_xilinx -top module_name

# Generate statistics
stat

# Write netlist
write_verilog -noattr output.v
```

---

## Common Quiz Question Types

### 1. Design Analysis
- Given a design, identify if it's hierarchical, parallel, sequential
- Count resources needed (adders, registers, LUTs)
- Calculate latency and throughput

### 2. Code Understanding
- What hardware does this Verilog create?
- Is this synthesizable? Why/why not?
- Will this create combinational or sequential logic?

### 3. FSM Design
- Draw state diagram for given specification
- Write Verilog for FSM
- Identify Moore vs Mealy characteristics

### 4. Trade-off Questions
- Compare two designs (area, speed, power)
- Which design is Pareto optimal?
- Justify design choice for given constraints

### 5. Synthesis Output Interpretation
- Interpret synthesis reports
- Explain resource utilization
- Calculate critical path delay

---

## Important Formulas

### Multiplier Output Width
- n-bit × n-bit = 2n-bit output
- Covers full range: (2ⁿ-1) × (2ⁿ-1) = 2²ⁿ - 2ⁿ⁺¹ + 1 < 2²ⁿ

### Adder Tree Depth
- For n inputs: ⌈log₂(n)⌉ stages
- Example: 8 inputs → 3 stages (8→4→2→1)

### FSM State Encoding
- **Binary**: ⌈log₂(S)⌉ bits for S states
- **One-Hot**: S bits for S states

### Clock Period Constraint
- T_clk ≥ T_cq + T_comb + T_setup
- For combinational: T_comb is critical path delay

---

## Tips for Quiz Success

1. **Understand, don't memorize code**
   - Know what hardware each construct creates
   - Trace through examples mentally

2. **Draw diagrams**
   - Block diagrams for hierarchical design
   - State diagrams for FSMs
   - Timing diagrams for sequential circuits

3. **Practice resource counting**
   - Count adders, registers, multiplexers
   - Estimate area complexity

4. **Check edge cases**
   - Reset behavior
   - First and last iterations
   - Signed number handling

5. **Verify your FSM**
   - All states reachable?
   - All transitions defined?
   - Reset state correct?

6. **Synthesis awareness**
   - What optimizations might occur?
   - How will FSM be encoded?
   - What resources will be used?

Good luck with your quiz!