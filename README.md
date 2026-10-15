# RTL, Physical Design and VSDBabySoC — Complete Course Documentation

## 1. Introduction

ASIC (Application-Specific Integrated Circuit) design involves developing, verifying, synthesising, and implementing digital hardware for a specific application. The process begins with a Register Transfer Level (RTL) description and progresses through logic optimisation, gate-level verification, physical design, timing analysis, and layout generation.

This documentation covers three major areas: RTL design and optimisation, Physical Design using OpenLane and the Sky130 PDK, and VSDBabySoC integration and implementation. It includes concepts, tools, workflows, and verification evidence associated with these areas.

---

# 2. RTL Online Course Documentation

## 2.1 Introduction to RTL Design

Register Transfer Level (RTL) design describes digital hardware using Hardware Description Languages such as Verilog. It represents combinational operations, sequential logic, registers, and data transfers. RTL defines the required hardware behaviour before synthesis converts the design into a gate-level netlist.

The RTL design process includes writing Verilog modules, developing testbenches, simulating the design, analysing waveforms, synthesising the RTL, and verifying the resulting implementation.

## 2.2 RTL Design and Verification Flow

A design is developed and checked in the following order before progressing to physical implementation:

1. Design specification
2. Verilog RTL coding
3. Testbench development
4. RTL simulation (if functionality is incorrect, return to RTL coding)
5. RTL synthesis
6. Gate-level netlist
7. Gate-level simulation
8. Waveform and result analysis

Simulation helps identify functional errors, while synthesis produces a hardware representation suitable for further analysis.

## 2.3 Combinational and Sequential Logic Optimisation

Logic optimisation improves circuit efficiency while preserving the intended behaviour. Its main objectives are to reduce unnecessary hardware, improve timing, and manage power consumption.

Common combinational optimisation techniques include:

- **Constant propagation:** Replaces expressions involving known constants with simplified results.
- **Boolean simplification:** Reduces logic expressions using Boolean algebra.
- **Redundant logic removal:** Eliminates logic that does not affect required outputs.
- **Technology mapping:** Maps logic functions to suitable cells available in the target standard-cell library.

### Example 1: AND Logic Optimisation

```verilog
module opt_check(input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```


Therefore, it can be simplified to an AND function.

### Example 2: OR Logic Optimisation

```verilog
module opt_check2(input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```



Therefore, the function is equivalent to an OR operation.

Sequential optimisation applies to circuits containing registers and flip-flops. It includes sequential constant propagation, finite-state-machine state optimisation, retiming, and sequential logic cloning. These techniques can reduce unnecessary state or improve timing, provided the transformations preserve the required behaviour.

## 2.4 D Flip-Flop and Counter Optimisation

A D flip-flop stores the value present at its D input on the active clock edge. Its output remains stored until another applicable clock event or reset changes it.

If a register is guaranteed to hold a constant value under the design's reset and operating assumptions, synthesis may propagate that value through connected logic.



for either value of A. Initial-state and reset behaviour must be considered before declaring a register constant.

Counter optimisation demonstrates how synthesis can remove unused state. If only the least significant bit of a counter contributes to any required output, other bits may be eliminated when they do not affect required behaviour. If the output depends on the complete counter value, the relevant bits must be retained.

Yosys can be used to synthesise these examples and inspect the resulting netlists and cell statistics.

## 2.5 Gate-Level Simulation (GLS)

Gate-Level Simulation verifies the synthesised netlist using testbench inputs. Unlike RTL simulation, GLS uses a gate-level representation and may require standard-cell Verilog models when technology-specific cells are instantiated.

The GLS flow is:

1. Verilog RTL is synthesised by Yosys into a technology-mapped netlist.
2. The netlist, the testbench, and the standard-cell Verilog models are compiled and simulated with Icarus Verilog.
3. The simulation produces output logs and a VCD file.
4. The VCD file is analysed in GTKWave.

The main tools are:

- **Yosys:** RTL synthesis and logic optimisation.
- **Icarus Verilog:** Verilog compilation and simulation.
- **Standard-cell Verilog models:** Functional models for mapped technology cells.
- **GTKWave:** Visualisation and analysis of simulation waveforms.

GLS helps verify that synthesis preserves expected functionality. Timing-aware GLS requires suitable delay models and timing checks; a functional simulation alone does not prove timing closure.

## 2.6 Blocking and Non-Blocking Assignments

Verilog provides two commonly used assignment types:

- **Blocking assignment (`=`):** Executes statements in procedural order. It is commonly used for combinational procedural logic.
- **Non-blocking assignment (`<=`):** Schedules updates for a later simulation event. It is generally preferred for clocked sequential logic.

Example:

```verilog
always @(posedge clk or posedge reset) begin
    if (reset) begin
        q0 <= 1'b0;
        q  <= 1'b0;
    end else begin
        q  <= q0;
        q0 <= d;
    end
end
```

Both right-hand-side expressions are evaluated before the scheduled register updates occur. Consequently, `q` receives the previous value of `q0`, while `q0` receives `d`.

Using blocking assignments in clocked logic can cause order-dependent simulation behaviour when registers depend on each other. Consistent assignment styles help prevent race conditions and simulation-synthesis mismatches.

## 2.7 Simulation-Synthesis Mismatch

A simulation-synthesis mismatch occurs when behaviour observed during RTL simulation differs from the intended or synthesised circuit behaviour.

One possible cause is an incomplete sensitivity list:

```verilog
always @(sel) begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

This block executes when `sel` changes but not necessarily when `i0` or `i1` changes. Therefore, the simulated output may become stale.

A safer combinational style is:

```verilog
always @(*) begin
    if (sel)
        y = i1;
    else
        y = i0;
end
```

Other causes include incomplete combinational assignments that infer latches, inappropriate assignment styles, and differences between simulation models and synthesis interpretations. Comparing RTL waveforms, synthesis reports, mapped netlists, and GLS results helps identify these issues.

## 2.8 RTL Documentation Evidence

The repository should include relevant Verilog source files, testbenches, simulation logs, GTKWave screenshots, Yosys synthesis results, cell statistics, gate-level netlists, and GLS results. Each practical example should describe its objective, implementation, expected behaviour, observed result, and any corrections made.

---

# 3. Physical Design (PD) Online Course Documentation

## 3.1 Introduction to Physical Design

Physical design converts a synthesised netlist into a physical layout suitable for integrated-circuit manufacturing. The course introduces OpenLane, the Sky130 Process Design Kit (PDK), standard-cell libraries, floorplanning, placement, clock-tree synthesis, routing, timing analysis, and physical verification.

The PDK supplies technology-specific information, including process data, physical layers, design rules, and standard-cell libraries. OpenLane coordinates the relevant tools in an ASIC implementation flow.

## 3.2 ASIC RTL-to-GDSII Flow

The conceptual implementation flow is:

1. RTL and timing constraints
2. Logic synthesis
3. Technology-mapped netlist
4. Floorplanning
5. Power planning
6. Pin placement and cell placement
7. Clock tree synthesis
8. Global and detailed routing
9. Timing and physical verification (if required checks fail, debug and optimise, then return to placement)
10. GDSII layout generation

This is a conceptual flow. Actual tool flows may include additional stages, intermediate checks, and iterations. GDSII generation alone does not establish that every timing and physical-verification requirement has passed.

## 3.3 Floorplanning and Power Planning

Floorplanning defines the core and die dimensions, aspect ratio, utilisation, and initial arrangement of the design.


A suitable floorplan provides sufficient space for standard-cell placement, routing, and power distribution. High utilisation may increase congestion, while excessive core area can increase the die size.

Pre-placed cells are assigned physical locations before the remaining cells are placed. Decoupling capacitors help reduce local power-supply fluctuations. Power planning establishes the VDD and ground distribution network using appropriate power connections, rails, straps, and rings.

Pin placement determines the locations of input, output, and clock pins around the design boundary. Appropriate pin placement can improve connectivity and reduce routing difficulties.

## 3.4 Standard-Cell Placement and Libraries

Standard-cell placement assigns physical locations to cells in the synthesised netlist. Global placement determines approximate positions, while detailed placement refines them to satisfy legal placement constraints. Placement optimisation may improve timing and routability by adjusting cell locations, buffering, or sizing.

A standard-cell library contains reusable logic cells such as inverters, NAND gates, multiplexers, buffers, and flip-flops.

Two important library formats are:

| Format | Purpose |
|---|---|
| LEF | Describes physical cell dimensions, pins, and routing obstructions. |
| Liberty (`.lib`) | Describes logical functions and timing, power, and electrical characteristics. |

Physical-design tools use LEF information for placement and routing, while synthesis and timing tools use appropriate logical and electrical library models.

## 3.5 CMOS Inverter and ngspice Characterisation

A CMOS inverter consists of PMOS and NMOS transistors that implement the NOT function. When the input is LOW, the PMOS pulls the output towards the supply voltage. When the input is HIGH, the NMOS pulls the output towards ground.

ngspice can be used to analyse transistor-level circuit behaviour through SPICE simulations:

- **DC analysis:** Examines the voltage-transfer characteristic.
- **Transient analysis:** Observes circuit behaviour over time.
- **Propagation delay:** Measures the time between an input transition and the corresponding output transition.
- **Rise and fall times:** Measure output transition durations using specified voltage thresholds.
- **Power analysis:** Evaluates consumption under defined operating conditions.

Characterisation results explain how input slew, output load, cell sizing, and process conditions affect circuit performance.

## 3.6 CMOS Layout and Sky130

Physical layout represents the geometry used to fabricate an integrated circuit. Tools such as Magic can be used to create and inspect layouts with the relevant technology files.

The layout process involves structures such as active regions, wells, transistor gates, source and drain regions, contacts, vias, and metal interconnect layers. These geometries form the transistors and their connections.

The Sky130 PDK provides technology-specific information used to design and implement circuits for the Sky130 process. Correct technology configuration and compatible libraries are essential for meaningful physical-design results.

## 3.7 Static Timing Analysis (STA)

Static Timing Analysis evaluates whether signal paths satisfy timing requirements without simulating every possible input sequence. OpenSTA analyses timing using the gate-level netlist, Liberty timing libraries, clock definitions, and timing constraints.

Important concepts include:

- **Arrival time:** The time a signal reaches a timing point.
- **Required time:** The timing limit against which arrival is evaluated.
- **Setup time:** The minimum time data must be stable before the active clock edge.
- **Hold time:** The minimum time data must remain stable after the active clock edge.
- **Slack:** The timing margin calculated for a timing check.
- **Critical path:** A path that limits timing performance or has little timing margin.
- **Clock latency and skew:** Characteristics of clock arrival times at sequential elements.

Positive slack generally indicates that the corresponding timing requirement is satisfied, while negative slack indicates a timing violation.

Ideal-clock analysis uses an idealised clock network. Post-CTS and post-route analysis can include clock-network and interconnect effects when the required implementation data is available.

A typical STA process includes loading the timing library, reading the netlist, defining clocks and constraints, linking the design, checking setup and hold requirements, and generating timing reports. Meaningful results require compatible libraries, netlists, and constraints.

## 3.8 Clock Tree Synthesis and Routing

Clock Tree Synthesis (CTS) constructs a distribution network to deliver clock signals to sequential elements. Its objectives include controlling skew, managing latency, and providing sufficient drive strength. Clock buffers may be inserted to distribute the signal across the design.

Routing connects standard-cell pins and other design components through metal interconnects and vias.

- **Global routing:** Determines approximate routing paths and considers congestion.
- **Detailed routing:** Establishes actual wire segments and vias while respecting design rules.
- **Access points:** Identify legal locations for connecting pins and routing segments.
- **Vias:** Provide electrical connections between metal layers.

TritonRoute is the detailed-routing engine discussed in the course material. Routing results must be checked for connectivity, design-rule violations, and timing impact.

## 3.9 Physical Verification and OpenLane

- **Design Rule Checking (DRC):** Verifies that layout geometry follows manufacturing rules such as metal width, spacing, and via requirements.
- **Layout Versus Schematic (LVS):** Checks whether physical connectivity matches the intended circuit.
- **STA:** Checks timing requirements.
- **Power-integrity analysis:** Evaluates the quality of the power-distribution network.

OpenLane provides a flow for coordinating ASIC implementation stages. Example commands from the course material include:

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
run_placement
```

The commands depend on the installed OpenLane version, design configuration, and environment. They should be adapted to the actual project rather than treated as universal commands.

PD documentation should retain configuration files, tool logs, synthesis netlists, cell statistics, floorplan and placement outputs, power-network results, CTS reports, routing results, timing reports, DRC/LVS summaries, and the final GDSII output where available.

---

# 4. VSDBabySoC Documentation

## 4.1 Introduction to System-on-Chip Design

A System-on-Chip (SoC) integrates multiple functional components into one integrated circuit. Depending on its architecture, it may contain a processor, memory, clock-generation circuitry, interfaces, and digital or mixed-signal IP blocks.

VSDBabySoC documentation focuses on the actual design hierarchy, IP integration, functional verification, synthesis, physical implementation, and final results. The architecture must be documented using the real project source files and configuration.

## 4.2 SoC Architecture and Integration

Conceptually, the VSDBabySoC top-level module integrates a processor IP, a clock-generation IP, and other digital or mixed-signal IP. These blocks communicate through interconnect and control signals, which in turn connect to the design's inputs, outputs, and interfaces. This is a conceptual description, not a verified representation of the exact VSDBabySoC hierarchy.

The final repository should replace or supplement this description with the actual top-level module and IP hierarchy. Exact IP names, ports, clock connections, reset signals, signal widths, and interfaces should be taken from the project RTL.

RTL integration combines independently designed modules into the top-level design. The process involves checking module declarations, port connections, signal widths, parameter values, dependencies, clock and reset connections, and source-file inclusion. Successful elaboration confirms that the hierarchy can be resolved, but functional verification is still required.

## 4.3 VSDBabySoC Verification and Synthesis Flow

The intended progression is:

1. VSDBabySoC RTL and configuration
2. RTL compilation and elaboration
3. Functional simulation (if expected behaviour is not verified, debug the RTL or testbench and return to compilation)
4. Synthesis and technology mapping
5. Gate-level netlist
6. Physical design
7. STA and physical verification
8. GDSII generation and results

Actual completion of each stage must be established using the project's execution logs and generated outputs.

## 4.4 Functional Verification

Functional verification checks whether the integrated design behaves according to its specification. The documentation should identify the testbench, input stimuli, expected outputs, observed results, and any errors or warnings.

Depending on the available project configuration, verification may include:

- Compilation and elaboration of the complete design.
- Clock and reset behaviour.
- Processor or top-level output behaviour.
- Interactions between integrated IP blocks.
- Simulation logs and waveform inspection.
- Debugging and retesting after corrections.

Standalone multiplexer, counter, or D flip-flop waveforms should not be used as evidence of complete SoC-level verification.

## 4.5 Synthesis and Physical Implementation

Synthesis converts the supported RTL hierarchy into a gate-level netlist through logic optimisation and technology mapping. Documentation should identify the top-level module, RTL source files, target technology, standard-cell libraries, synthesis warnings, resulting netlist, and cell statistics.

Any black-box IP blocks or simulation-only models must be identified and explained. A black box does not demonstrate that its internal hardware has been implemented.

When the project configuration and available models support physical implementation, the design can proceed through floorplanning, power planning, placement, CTS, routing, timing analysis, and physical verification.

| Stage | Evidence to document |
|---|---|
| Floorplanning | Core and die dimensions, utilisation, and layout |
| Power planning | Power-distribution configuration and connectivity checks |
| Placement | Placement reports and layout |
| CTS | Clock-tree information and reports |
| Routing | Global- and detailed-routing results |
| STA | Setup/hold slack, critical paths, and analysed corners |
| Physical verification | DRC and LVS reports, where applicable |
| Final layout | GDSII file and generation status |

## 4.6 Timing Verification and Final Results

Timing verification evaluates whether the implemented design meets its timing constraints. Reports should identify the clock definitions, timing libraries, analysed process-voltage-temperature (PVT) corners, worst setup and hold slack, critical paths, and unresolved violations.

The documentation should distinguish pre-layout or ideal-clock timing, post-synthesis timing, post-CTS timing, and post-route timing. Multi-corner, multi-mode analysis should be documented when required by the design flow.

The VSDBabySoC results section should include the actual configured top-level module, target technology, synthesis status, cell area, core and die dimensions, placement status, CTS status, routing status, timing results, DRC/LVS status, and GDSII generation status.

A generated GDSII file alone does not prove timing closure or complete sign-off. Each required verification result must be supported by its corresponding report.

---

# 5. Overall Conclusion

The RTL Online Course develops an understanding of Verilog coding, combinational and sequential optimisation, simulation, synthesis, gate-level verification, and simulation-synthesis mismatch. The Physical Design Online Course explains the ASIC implementation process, including standard-cell libraries, floorplanning, power planning, placement, CTS, routing, STA, physical verification, and GDSII generation. The VSDBabySoC section applies these concepts to SoC architecture, IP integration, functional verification, synthesis, and physical implementation.

Together, these modules describe the progression from RTL development to physical chip layout. A complete repository should combine theoretical explanations with practical evidence, including source code, testbenches, configuration files, commands, waveforms, synthesis statistics, timing reports, physical-verification results, and generated layout files.

