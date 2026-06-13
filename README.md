# 32-bit Custom MIPS Processor (3-Cycle Architecture) 

A custom 32-bit MIPS processor built from scratch in Verilog, completely synthesized and implemented for Xilinx 7-Series FPGA architecture (Target: Kintex-7 `xc7k480t`). This project was built as a course project for CS220 - Computer Organization and Architecture at IIT Kanpur under the supervision of [Prof. Mainak Chaudhuri](https://www.iitk.ac.in/dr-mainak-chaudhuri)

This project demonstrates full Register Transfer Level (RTL) design, finite state machine (FSM) control, critical-path timing optimization, and sub-word memory access. It successfully executes cross-compiled C code and meets all timing constraints at 100 MHz.

## System Architecture

```mermaid
graph TD
    %% Styling
    classDef topLevel fill:#2d3436,stroke:#00b894,stroke-width:3px,color:#fff;
    classDef coreModule fill:#0984e3,stroke:#74b9ff,stroke-width:2px,color:#fff;
    classDef subModule fill:#00cec9,stroke:#81ecec,stroke-width:2px,color:#fff;
    classDef memory fill:#e84393,stroke:#fd79a8,stroke-width:2px,color:#fff;
    classDef optimization fill:#fdcb6e,stroke:#ffeaa7,stroke-width:2px,color:#2d3436;

    %% Modules
    Computer[Computer.v <br> Top-Level Architecture]:::topLevel

    subgraph Processor_Subsystem [Processor.v]
        FSM[Control FSM <br> 3-Cycle Sequencer]:::coreModule
        RegFile[Register File <br> 32x32-bit | $0 Hardwired]:::coreModule
        ALU[Main ALU <br> Arithmetic/Logic]:::coreModule
        FastAdder((Fast-Adder Bypass <br> Address Optimization)):::optimization
    end

    Memory[Memory.v <br> Big-Endian Controller <br> lb, lbu, sb, sh logic]:::memory

    %% Connections
    Computer --> Processor_Subsystem
    Computer --> Memory

    FSM -.->|Control Signals| RegFile
    FSM -.->|ALU Opcode| ALU
    FSM -.->|MemRead / MemWrite| Memory

    RegFile ==>|Read Data 1 & 2| ALU
    ALU ==>|ALU Result| RegFile
    
    RegFile ==>|Base Address| FastAdder
    FastAdder ==>|Optimized Mem Address| Memory
    
    Memory ==>|Load Data| RegFile
    RegFile ==>|Store Data| Memory
```

## Key Architectural Features
* **Fast-Adder Bypass:** To resolve critical path timing violations during memory access, a dedicated hardware adder (`fast_mem_addr = SRC1 + BRANCH_OFFSET`) bypasses the main ALU, allowing memory addresses to be computed instantly without waiting for the ALU propagation delay.
* **Big-Endian Memory Controller:** Custom byte-enable extraction logic built directly into the datapath to precisely handle sub-word instructions (`lb`, `lbu`, `lh`, `lhu`, `sb`, `sh`).
* **Hardwired `$0` Register:** The register file physically grounds register `$0` to prevent uninitialized data cascades (the "sea of red Xs") during branching and loading operations.
* **Circular I/O Buffer:** Integrated an AXI-style hardware stall mechanism to securely buffer data out to a host environment without losing cycles.

## Performance & Hardware Metrics

The design was fully placed and routed using Vivado. It comfortably achieves timing closure at a **100 MHz clock (10.00 ns period)**.

| Metric | Value | Raw Report |
| :--- | :--- | :--- |
| **Target Frequency** | 100 MHz (10.00 ns) | [Timing Summary](./docs/Timing%20Summary.txt) |
| **Worst Negative Slack (WNS)** | **+1.132 ns** | [Timing Summary](./docs/Timing%20Summary.txt) |
| **Worst Hold Slack (WHS)** | **+0.136 ns** | [Timing Summary](./docs/Timing%20Summary.txt) |
| **Slice LUTs** | 1,498 (0.50% utilization) | [Utilization Report](./docs/Resource%20Utilization%20Report.txt) |
| **Slice Registers** | 357 (0.06% utilization) | [Utilization Report](./docs/Resource%20Utilization%20Report.txt) |
| **Total On-Chip Power** | 0.192 W | [Power Report](./docs/Power%20Report.txt) |

## Visual Documentation

Following are the physical logic mapping and timing simulations.

**[RTL Schematic (PDF)](./docs/MIPS%20Processor%20RTL%20Schematic.pdf)** - Register Transfer Level Layout of the different modules incorporated in the design.

![Implemented Design (PNG)](./docs/Implemented%20Design.png)
*The physical layout and routing of the logic cells on the FPGA fabric after Place and Route.*

![Simulation Waveform (PNG)](./docs/Simulation%20Waveform.png)
*Behavioral simulation proving correct instruction execution and state transitions.*

## Repository Structure

```text
MIPS-Processor/
├── README.md
├── docs/                                <-- Physical design proofs and reports
│   ├── Implemented Design.png
│   ├── MIPS Processor RTL Schematic.pdf
│   ├── Power Report.txt
│   ├── Resource Utilization Report.txt
│   ├── Simulation Waveform.png
│   └── Timing Summary.txt
├── src/
│   ├── rtl/                             <-- Hardware Description files
│   │   ├── ALU.v
│   │   ├── Computer.v
│   │   ├── Memory.v
│   │   ├── Processor.v
│   │   └── RegisterFile.v
│   ├── include/
│   │   └── defs.vh                      <-- Opcodes and global macros
│   └── tb/
│       └── Computer_tb.v                <-- State-based verification testbench
└── vivado/
    └── Processor System.xpr             <-- Clean Vivado Project File
```

## How to Build and Simulate

1. Clone the repository: `git clone https://github.com/Aman02032006/MIPS-Processor-Implementation.git`
2. Open **Vivado** and open the project file located at `vivado/Processor System.xpr`.
3. To view the waveforms, click **Run Simulation** -> **Run Behavioral Simulation**.
4. To view the physical architecture, click **Open Implemented Design**.