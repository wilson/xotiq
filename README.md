# **xotiq**

> (pronounced "exotic")

**A Synthesis Toolchain for Clockless and Reversible Computing.**

`xotiq` is an open-source Hardware Description Language (HDL) and compiler infrastructure for the post-von-Neumann omputing era.

Traditional HDLs (Verilog, VHDL) are built on the foundational assumptions of **synchronous clocks** and **irreversible logic** (gates that destroy information and dissipate heat, hitting the thermal wall of Landauer's limit). `xotiq` abandons this entirely. It is designed specifically for emerging physical substrates—such as **All-Optical Photonics** and **Superconducting Fluxonics**—where computation is treated as the continuous, ballistic flight of discrete energy quanta.

In `xotiq`, there are no global clocks, no `wait` states, and no data destruction. Information has primacy, logic is topological, and computation is thermodynamically reversible.

-----

## The Shift: Space *is* Time

Recent hardware breakthroughs have revealed that moving data (parasitic capacitance, electro-optical conversions, clock trees) costs exponentially-more energy than performing computations on it.  

The solution is **Ballistic Reversible Computing**.
  * **Ballistic:** Data representations (Photons, Magnetic Fluxons) propagate through logic elements without losing energy or requiring restoration.
  * **Clockless (Time-of-Flight):** Time is replaced by Space. "Cycles" are dictated purely by time-of-flight (e.g., the length of a Silicon Nitride waveguide or a Long Josephson Junction). Synchronization is achieved via physical path length.
  * **Reversible:** To push energy efficiency past Landauer's Limit ($E \ge k_B T \ln 2$), no information can be erased. `xotiq` enforces logical reversibility—every gate's outputs must map bijectively back to its inputs. Old state is physically ejected and recycled, not overwritten.

`xotiq` provides the Intermediate Representation (IR) to describe, route, and synthesize these zero-dissipation, continuous-flow topologies.

-----

## The Abstraction: Spatiotemporal Graphs

With `xotiq`, we do not write sequential state machines. Instead we construct **Information-Conserving Flow Graphs**. The compiler's primary job is to automatically route paths so that ballistic quanta arrive at interaction points simultaneously, and to enforce reversibility.

### 1\. Flight Paths (`f_node`)

  * **Role:** The Interconnects. They dictate both routing and timing.
  * **Compiler Action:** **Delay Matching.** In a clockless system, if Quanta A and Quanta B must interact, they must arrive at the same picosecond. `xotiq` automatically calculates propagation speeds ($c$ in waveguides, fluxon speed in Long Josephson Junctions) and generates exact spatial routing lengths to mathematically guarantee simultaneous time-of-flight arrivals.

### 2\. Reversible Barriers (`r_node`)

  * **Role:** The Logic. Active, non-linear interactions that alter the path of quanta without absorbing or destroying their kinetic energy.
  * **Compiler Action:** **Reversibility Enforcement.** To prevent heat dissipation, the compiler automatically maps standard logic into reversible equivalents, utilizing **Polarity Filters**, **Optical Crossbars**, and **Delay Loops** (to safely eject and recycle previous states rather than overwriting them).

### 3\. Quanta Injection (`inject`)

  * Computation is triggered by releasing energy into the fabric. The system proceeds deterministically, based entirely o the graph's physical geometry.

-----

## Code Example (Zig SDK)

Here we define a reversible, asynchronous `AND` Gate. This example synthesizes the architecture from **Sandia Patent US12620993B1**. There are no clocks, variables, or destructive overrides.

```zig
const std = @import("std");
const xq = @import("xotiq");

// A Ballistic, Asynchronous Reversible Switch Gate
// Synthesizes to either Superconducting LJJs or Photonic Waveguides
pub const AsyncSwitchGate = struct {
    // Ballistic Terminals: Discrete Quanta (Fluxons or Photons)
    control_in: xq.Port(.in),
    data_in: xq.Port(.in),

    // Reversible Outputs: To evade Landauer's limit, no data is destroyed.
    // Unused or reflected quanta must be cleanly routed to conserve energy.
    control_out:      xq.Port(.out), // Pass-through control (preserves injected energy)
    data_out:         xq.Port(.out), // Evaluated data
    compensation_out: xq.Port(.out), // Reflected "Garbage" output (preserves Boolean state)

    pub const topology = .{
        // Internal Physical Primitives
        .polarity_in   = xq.nodes.PolaritySeparator(),
        .polarity_out  = xq.nodes.PolaritySeparator(),
        .circulator    = xq.nodes.Circulator(),
        .barrier = xq.nodes.ControlledBarrier(),

        // The compiler strictly enforces topological conservation.
        // Paths must be explicitly defined, and flight times are delay-matched automatically.

        // Control Path: Replaces memory state, ejecting the old state ballistically
        xq.route(.{ .src = .control_in,           .dst = .polarity_in.in }),
        xq.route(.{ .src = .polarity_in.pos,      .dst = .barrier.mem_in }),
        xq.route(.{ .src = .barrier.eject,  .dst = .polarity_out.pos }),

        // Delay Loop (Preserves token energy during asynchronous wait)
        xq.route(.{ .src = .polarity_in.neg,      .dst = .polarity_out.neg, .delay = 10 * xq.ps }),
        xq.route(.{ .src = .polarity_out.out,     .dst = .control_out }),

        // Data Path: Ballistic evaluation via time-of-flight
        xq.route(.{ .src = .data_in,           .dst = .circ.in }),
        xq.route(.{ .src = .circulator.fwd,       .dst = .barrier.filter_in }),

        // Output routing based on Barrier Pass/Reflect
        xq.route(.{ .src = .barrier.pass,   .dst = .data_out }),
        xq.route(.{ .src = .barrier.reflect,.dst = .circ.rev }),
        xq.route(.{ .src = .circulator.drop,      .dst = .compensation_out }),
    };
};

pub fn main() !void {
    // Synthesize the spatiotemporal fabric
    // The compiler automatically calculates path lengths for time-of-flight synchronization
    var fabric = try xq.synthesize(AsyncSwitchGate, .SuperconductingBARC);

    // Time-of-Flight Execution
    // Inject quanta (.positive represents magnetic flux orientation or optical phase)
    fabric.inject(.control_in, .positive);
    fabric.inject(.data_in, .positive);

    // Detect the wave-fronts at the sinks
    // There is no clock. Causality is driven entirely by physical propagation.
    const results = try fabric.await_settle();
}
```

-----

## Compiler Backends

The `xotiq` compiler lowers topological flow graphs into physical substrate layouts, verifying thermodynamic reversibility and strict time-of-flight path-matching along the way.

  * **Target: Integrated Photonics (Akhetonics RP / SiN platforms)**

      * **Output:** GDSII / Lumerical Interconnects.
      * **Method:** `f_nodes` are synthesized as precise-length SiN waveguides to ensure wave-fronts collide at exact picosecond intervals. `r_nodes` are lowered to sparse optical crossbars and Semiconductor Optical Amplifiers (SOAs).
      * **Memory:** Synthesizes functional, immutability-first Read-Write-Once-Read-Many (RWORM) Phase-Change Material (PCM) blocks, side-stepping the need for volatile latch states.

  * **Target: Ballistic Superconducting (Sandia BARC / RSFQ / LJJ)**

      * **Output:** Superconducting SPICE / Inductance Networks / GDSII.
      * **Method:** Lowers `r_nodes` into Long Josephson Junction (LJJ) networks. Synthesizes *Circulators* and *Polarity Selectors*. State is handled via Reversible Memory Cells that safely "eject" stored flux quanta into secondary delay loops to avoid Landauer heat generation.

  * **Target: Reversible Emulation (CPU/FPGA)**

      * **Output:** SystemVerilog / C++.
      * **Method:** Validates Spatiotemporal Graphs using high-fidelity time-domain simulation. The compiler verifies that signal fronts arrive at barriers simultaneously and that input entropy strictly matches output entropy (ensuring Landauer compliance) prior to tape-out.

-----

## Inspiration & Citations

`xotiq` synthesizes leading breakthroughs from thermodynamics, quantum mechanics, and continuous-flow physics.

### Target Architectures & Substrates

  * **[Ballistic Superconducting Circuit for Asynchronous Reversible Logic Element](https://ppubs.uspto.gov/api/pdf/downloadPdf/12620993)** (Sandia National Labs, US Patent 12,620,993 B1, 2026)
    > The definitive architectural blueprint for using magnetic flux quanta (fluxons) to achieve zero-dissipation logic via controlled barriers, circulators, and reversible memory cells that safely eject state.
  * **[An All-Optical General-Purpose CPU and Optical Computer Architecture](https://arxiv.org/abs/2403.00045)** (Akhetonics, 2024)
    > Defines the "time-of-flight" computing framework, demonstrating that single-cycle, continuous, clockless data propagation can simulate Turing-complete RISC architectures all-optically, eliminating electro-optical conversion bottlenecks.

### Foundations of Reversible Computing

  * **[Logical Reversibility of Computation](https://dl.acm.org/doi/10.1147/rd.176.0525)** (C.H. Bennett, 1973)
    > The mathematical proof that computation can be performed without energy dissipation if it is logically reversible, providing the escape velocity from Landauer's limit.
  * **[Fundamental Physics of Reversible Computing](https://cra.org/ccc/wp-content/uploads/sites/2/2020/10/CCC-Frank-Day1-v2.pdf)** (M.P. Frank, 2020)
    > Outlines the physical requirements for abandoning von Neumann irreversible logic to scale computing beyond the limits of current CMOS thermodynamics.
  * **[Can Programming Be Liberated from the von Neumann Style?](https://dl.acm.org/doi/10.1145/359576.359579)** (John Backus, 1978)
    > The Turing Award lecture establishing the necessity of functional, side-effect-free programming models—the exact software paradigm required for interfacing with non-volatile, reversible physical fabrics.

-----

*© 02026 Wilson Bilkovich*
