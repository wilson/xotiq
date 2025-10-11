### **xotiq**
*(pronounced “exotic”)*

An HDL for designing and simulating energy-based, probabilistic hardware, to implement the _Dirac Dataflow Architecture_.

-----

### 1\. The Philosophy: Physical Intelligence

`xotiq` is not an HDL for describing clocks and logic gates. It's a language for describing a **network of influences** that computes by physically settling into a low-energy state.

This architecture is a direct hardware implementation of a **Stochastic Hopfield Network / Boltzmann Machine**. It provides a physical substrate for brain-inspired computing models like the **Free Energy Principle (FEP)**, where computation is a process of Bayesian inference via energy minimization.

### 2\. The Language: A Hybrid Approach

`xotiq` is a hybrid language that separates the computational fabric from its controller.

  * **Probabilistic Fabric (Declarative):** The core of the design is described as a graph of `p_node`s (stochastic neurons). You define the network's "knowledge" or "internal model" using a simple, declarative syntax for weighted connections, which directly describes the J-matrix.

```
// Declare probabilistic nodes
p_node A, B, C;

// Define the J-matrix of influences
connect A -> B with weight -1.0;
connect C -> A with weight +0.5;
```

  * **Deterministic Control (Imperative):** The `p_node` fabric is managed by traditional, deterministic logic built from `d_node`s. This logic, which resembles standard Verilog, acts as the "scientist" running the experiment. It uses special commands to interact with the fabric:
      * **`clamp(p_node, probability)`:** Provides sensory input by forcing a node to a specific state.
      * **`observe(p_node, duration)`:** Reads the resulting inference by measuring a node's time-averaged state after the network has settled.

### 3\. The Compiler Toolchain: From Language to Hardware

The compiler translates a `xotiq` design into a "management regime" for a specific hardware target. The backend architecture is modular to support different stages of development.

  * **Simulation Backend -\> Verilog-AMS:** For high-fidelity, mixed-signal simulation. This backend models the actual analog physics of the components, allowing for accurate verification of the energy-based dynamics.
  * **Emulation Backend -\> SystemVerilog:** For purely digital emulation on standard FPGAs. This backend replaces the analog physics with a PRNG (e.g., an LFSR) and digital comparators, enabling rapid testing of the network's logic on hardware like the iCE40.
  * **Physical Prototype Backend -\> Verilog + I²C Script:** For targeting a real hybrid system. This generates the deterministic control FSM in Verilog for a digital controller (FPGA) and the raw I²C command stream to configure the analog fabric on an FPAA (e.g., a GreenPAK chip).


### 4\. The Python

A Python-based suite serves as the compiler frontend and orchestration layer.

  * Parses `xotiq` code into a hardware-agnostic Intermediate Representation (IR).
  * Drives the selected backend to generate the target output (Verilog-AMS, SystemVerilog, etc.).
  * Manages the physical hardware interface for programming and running the FPGA+FPAA prototype.
  * Provides visualization tools (`matplotlib`, `plotly`) for analyzing energy landscapes and system state distributions.

### 5\. Future Directions

  * Formalize the IR into a true **Hardware Abstraction Layer (HAL)**, allowing new hardware backends to be added easily.
  * Develop a high-performance physical backend that emits **SPI commands or memory-mapped I/O** to manage a future custom photonic or spintronic chip.
  * Implement a **JAX/NumPyro backend**. This would enable powerful, gradient-based optimization of the network's physical weights (the J-matrix) directly from high-level problem descriptions.
