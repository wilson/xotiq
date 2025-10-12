### **xotiq**
*(pronounced “exotic”)*

A hardware description language for designing and simulating energy-based, probabilistic hardware.  
Designed to implement the *Dirac Dataflow Architecture*.  

-----

### 1\. The Philosophy: Automated "Right Action"

`xotiq` is not an HDL for describing clocks and logic gates.  
It's a language for describing a **network of influences** that computes by physically settling into a low-energy state within nanoseconds.  
The architecture is explicitly designed as a **non-cryogenic, short-coherence-time accelerator**.  

This architecture is a direct hardware implementation of a **Stochastic Hopfield Network / Boltzmann Machine**.  
It provides a physical substrate for brain-inspired computing models like the **Free Energy Principle (FEP)**, where computation is a process of Bayesian inference via energy minimization.  
The goal is not to compete with general-purpose quantum computers, but to solve a specific class of optimization and inference problems with unprecedented speed and efficiency at room temperature.  

### 2\. The Language: A Hybrid Approach

`xotiq` is a hybrid language that separates the computational fabric from its controller.

  * **Probabilistic Fabric (Declarative):** The core of the design is a graph of `p_node`s (stochastic neurons) whose states are represented by a **quantized probability axis** (e.g., 10-bit precision). You define the network's "knowledge": the energy landscape—using a declarative syntax for weighted connections (the J-matrix).  

    ```
    // Declare probabilistic nodes
    p_node A, B, C;

    // Define the J-matrix of influences
    connect A -> B with weight -1.0;
    connect C -> A with weight +0.5;
    ```

  * **Deterministic Control (Imperative):** The `p_node` fabric is managed by traditional, deterministic logic built from `d_node`s. This logic resembles standard Verilog and acts as the "investigator running the experiment", using special commands to interact with the fabric:

      * **`clamp(p_node, probability)`:** Provides sensory input by using local **electrical gates** to force a node's state.
      * **`observe(p_node, duration)`:** Reads the resulting inference by measuring a node's time-averaged state after the network has settled.

### 3\. The Compiler Toolchain: A Multi-Target Approach

The compiler's core design is modular, enabled by a hardware-agnostic Intermediate Representation (IR). This allows `xotiq` to target a range of backends, from simple emulators to high-fidelity physics simulators.

#### Prototyping & Emulation Backends

  * **Digital Emulation -\> SystemVerilog:** Generates a purely digital model using PRNGs to enable rapid testing of network logic on any standard FPGA (e.g., iCE40).
  * **Physical Prototype -\> Verilog + I²C Script:** Generates the hybrid code for the initial hardware prototype: a Verilog FSM for the FPGA controller and the I²C command stream to configure the FPAA analog fabric (e.g., GreenPAK).

#### Advanced Simulation & Modeling Backends

  * **Circuit-Level Simulation -\> Xyce / Verilog-AMS:** Translates the `xotiq` graph into a netlist for powerful, open-source SPICE-compatible simulators. This enables high-fidelity simulation of the analog circuit dynamics, including the behavior of op-amps and other components.
  * **Device-Level Physics Modeling -\> COMSOL / Lumerical:** The ultimate simulation target. This backend will translate the `xotiq` design into a full multiphysics model, allowing simulation of the underlying valleytronic and photonic behavior of the p-nodes themselves. This directly aligns with the goal of creating a "Comsol-for-Molecules" for novel computing architectures.

### 4\. The Python Host

A Python-based suite serves as the compiler frontend and orchestration layer.

  * Parses `xotiq` code into a hardware-agnostic Intermediate Representation (IR).
  * Drives the selected backend to generate the target output (Verilog, Xyce netlist, etc.).
  * Manages the physical hardware interface for programming and running the FPGA+FPAA prototype.
  * Provides visualization tools (`matplotlib`, `plotly`) for analyzing energy landscapes and system state distributions.

### 5\. Future Directions

  * **Formalize the IR into a true Hardware Abstraction Layer (HAL):** This makes the compiler truly retargetable, creating a stable interface between the language frontend and the growing list of simulation and hardware backends.
  * **Create a Device Synthesis Backend:** This takes a verified design and generates the full "management regime" (e.g., SPI commands, memory-mapped I/O configs) required to run on a final, custom-designed spintronic or valleytronic chip.
  * **Implement a JAX/NumPyro Backend:** This would enable powerful, gradient-based optimization and training of the network's physical weights (the J-matrix) directly from high-level problem descriptions, bridging the gap between training and physical design.
