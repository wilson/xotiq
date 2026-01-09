# **xotiq**
*(pronounced "exotic")*

**The Hardware Description Language for Hybrid Physics-Based Computing.**

`xotiq` is a language for designing systems where **deterministic control** (digital logic) interacts directly with **probabilistic physics** (analog dynamics).

It provides a unified abstraction for the emerging class of "post-Moore" accelerators—whether photonic, spintronic, or thermodynamically-driven—allowing engineers to describe computation as a process of energy minimization and physical settling, rather than just boolean switching.

---

## The Problem: The "Digital-Analog" Gap

Modern frontier hardware is hybrid. We are building chips that combine standard digital logic with "exotic" physical substrates (photonic meshes, magnetic tunnel junctions, valleytronic gates) to solve optimization and inference problems.

Current tools force a choice:
* **Verilog/VHDL:** Perfect for the digital control logic, but cannot model the continuous physics of the accelerator fabric.
* **SPICE/Multiphysics:** Perfect for the physics, but cannot model the complex state machines required to drive them.

`xotiq` bridges this gap. It treats the physical fabric not as a "black box" peripheral, but as a first-class citizen in the logic graph, managed by a compiler that understands both **clamping** (control) and **settling** (inference).

---

## The Abstraction

`xotiq` designs are "bipartite graphs". The compiler manages the impedance mismatch between two distinct domains of computing:

### **1. The Deterministic Domain (`d_node`)**
* **Role:** The "Controller." Handles sequential logic, state machines, I/O, and data routing.
* **Behavior:** Classical, restorative, non-linear.
* **Physical Realization:**
    * Standard CMOS Logic (Gates, FSMs).
    * **Photonic Resonators:** Microring resonators or saturable absorbers that "clamp" signals to binary states.
    * **Spintronics:** Stable, high-barrier magnetic junctions.

### **2. The Probabilistic Domain (`p_node`)**
* **Role:** The "Fabric." Handles pattern matching, optimization, and massive parallelism via physics.
* **Behavior:** Continuous, interferometric, energy-based.
* **Physical Realization:**
    * **Photonic Meshes:** Mach-Zehnder Interferometer arrays doing passive matrix math at the speed of light.
    * **Stochastic Logic:** PRNG-driven digital fabrics.
    * **Nanomagnetism:** Superparamagnetic islands interacting via dipole coupling.

### **The Bridge**
Computation is defined by the interaction between these domains:
* **`clamp(node, value)`:** The Digital domain forces a Physical node to a specific state (e.g., applying a voltage bias, tuning a laser input).
* **`observe(node)`:** The Digital domain reads the settled state of the Physical fabric (e.g., ADC, Photodetector).

```rust
// Example: A Hybrid Optical Logic Gate

// 1. The Fabric (p_nodes)
// A passive mesh of waveguides and couplers.
// In a photonic backend, this compiles to an interferometer mesh.
p_node A, B, Output;
link A <-> Output { weight: -1.0 }; // Destructive interference
link B <-> Output { weight: -1.0 }; 

// 2. The Control (d_nodes)
// The "Active" components that drive the laser and read the result.
d_node Controller {
    process {
        // "Clamp" inputs (Inject Coherent Light)
        clamp(A, 1.0);
        clamp(B, 0.0);
        
        // Wait for light to propagate through the mesh
        wait(10.ps);
        
        // "Observe" the result (Read the Photodetector)
        // This implicitly handles the analog-to-digital thresholding
        let result = observe(Output);
    }
}
```

---

## Compiler Backends

The `xotiq` toolchain uses a multi-stage lowering strategy to target increasing levels of physical realism.

* **Target: Digital Emulation (CPU/FPGA)**
* **Output:** SystemVerilog / C++. / etc.
* **Method:** `p_nodes` are lowered to pseudo-random number generators and digital accumulators. Allows for rapid algorithm verification on standard hardware.

* **Target: Analog Synthesis (FPAA/ASIC)**
* **Output:** SPICE Netlist.
* **Method:** `p_nodes` map to op-amp summing junctions or resistor networks; `d_nodes` map to standard voltage-mode logic.

* **Target: Integrated Photonics**
* **Output:** GDSII / Circuit Simulation Interconnects.
* **Method:**
* **WDM Support:** Topological links are assigned specific wavelengths ("colors"), enabling massive parallelism on single waveguides.
* **Non-Linearity:** `d_node` logic is synthesized as resonator/SOA structures for signal restoration.
* **Interference:** `p_node` meshes are synthesized as passive optical linear units.

---

## Contributing

`xotiq` is an open effort to build the standard infrastructure for post-Von-Neumann computing.
Core development focuses on the **Intermediate Representation (IR)** and the **Zig-based reference compiler**.

* **Logic Designers:** Help define standard libraries for probabilistic arithmetic.
* **Physicists:** Help refine the backend models for specific photonic and spintronic constraints.
* **Compiler Engineers:** Help optimize the lowering passes for hybrid FPGA/ASIC targets.

---
*© 02026 Wilson Bilkovich*
