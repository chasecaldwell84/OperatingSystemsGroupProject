# Milestone 1: Design Proposal

**Course:** CSI 4337, Fall 2026: Four-Model Parallel Processing Workbench
**Team name:** Table or Booth

**Group members:**
- Dakota Hernandez
- Owen Chipman
- Jack Fontenot
- Chase Caldwell
- Jonah Beck

---

## 1. Proposed Class or Module Design

![Module Design](images/updatedmoduledesign.png)



| Model | LOOP | DOT | PI |
|-------|------|-----|----|
| Sequential | `loopseq.cpp` | `dotseq.cpp` | `piseq.cpp` |
| `std::thread` | `loopSTD.cpp` | `dotSTD.cpp` | `piSTD.cpp` |
| OpenMP | `loopopen.cpp` | `dotopen.cpp` | `piopen.cpp` |
| MPI | `loopmpi.cpp` | `dotmpi.cpp` | `pimpi.cpp` |

Shared modules:

| Module | Job |
|--------|-----|
| `Config` | Reads and validates the input, and sets `INVALID_INPUT` or `UNSUPPORTED_MODE`. Under MPI, only rank 0 reads it. |
| `main.cpp` | Calls the right solver function for the mode and workload. COMPARE calls all four. |
| `Workload` | Generates element `i` from its index and the seed, so every model sees the same data. |
| `Timer` | Times only the compute step. Reports average, best, and worst over the repetitions. |
| `TraceLog` | Records TRACE events. It is off in timed runs. |
| `ResultValidator` | One check for all modes, against the sequential result. |
| `ReportFormatter` | Prints all output, from rank 0 only. Printing is not timed. |
| `ExperimentRunner` | Runs the timing experiments and writes CSV for the graphs. |

Every solver has the same call so main can dispatch them all the same

```cpp
struct RunResult {
    double    result;       
    int       checksum;       
    int       work_done;      
    int       active_workers; 
    std::vector<double> times_ms;      
};

```

## 2. Proposed Workload Representation


### Workload 1: LOOP (independent loop)
- Each element 'i' will be calculated such that no two elements are the same
- Elements will be calculated independently

### Workload 2: DOT (dot product)



### Workload 3: PI (Monte Carlo)

- Each trial `i` makes a random point `(x, y)` from the seed and `i`, so the same trial always
  gets the same point and no random generator is shared.
- Each worker counts how many of its points land inside the circle (`x*x + y*y <= 1`).
- The counts are added up, and `pi_est = 4 * hits / n`.

## 3. Proposed Thread and Rank Partitioning Strategies



## 4. Proposed Validation Strategy

Every parallel result is checked against the sequential result 

- **LOOP:** the checksum must match exactly.
- **DOT:** the result must match exactly
- **PI:** the hit count must match exactly, and every model must run exactly n trials.

The report prints `VALID: YES` or `VALID: NO`. Bad input prints `STATUS: INVALID_INPUT`. A
`tests/` folder holds inputs with their expected output, and a script runs them all.

## 5. Proposed Timing Methodology

- Use `std::chrono::steady_clock`, 
- Time only the computation, including thread creation, joining, and MPI communication.
- Don't time input parsing, setup, MPI start-up and shutdown, or printing.

## 6. Initial Explanation of Relevant Operating-System Concepts

- **Processes vs. threads.** 

- **Shared address space.** 

- **Shared vs. private data.** 

- **Thread creation and joining.** 

- **Race conditions.** 

- **Mutexes and critical sections.** 

- **Reduction.** 

- **Atomic operations.**

- **MPI ranks, communicators, and messages.** 

- **OS thread scheduling.**

- **Load balancing.**

- **Memory bandwidth and false sharing.** 

- **Timing variability.**

- **Amdahl's Law.** 
A way of determining how much a program can be improved by introducing
 multithreading. A program can not simply increase its speed by doubling the threads
 as the entire program cannot be threaded there will always be parts of the code 
 that stay sequential and arent improved by multithreading.

$S = \frac{1}{P + \frac{(1 - P)}{N}}$

S = speed up
P = part of the task that can be parallelized
N = number of proccessors



## 7. Planned Division of Responsibilities

Work is split by workload. Each workload owner also owns part of the shared project
| Workload | Owners |
|----------|--------|
| Workload 1: Independent Loop (LOOP) | Jack and Jonah |
| Workload 2: Dot Product (DOT) | Chase and Owen |
| Workload 3: Monte Carlo Pi (PI) | Dakota |

| Member | Workload / models | Shared work |
|--------|-------------------|-----------------------|
| Dakota Hernandez | PI: Sequential, std::thread, OpenMP, MPI | `Config` and parser (main), `Timer`, `ResultValidator`, `tests/` and test runner |
| Jack Fontenot | LOOP: Sequential, OpenMP | `WorkScheduler`, output formatting, build script, Amdahl analysis |
| Jonah Beck | LOOP: std::thread, MPI | `MpiEnv` (init, broadcast, timing, finalize), MPI docs |
| Owen Chipman | DOT: Sequential, std::thread | `SyncUtils`, race-condition demo, `TraceLog` |
| Chase Caldwell | DOT: OpenMP, MPI | `main` and mode dispatch, COMPARE mode, `ExperimentRunner`, graphs |

**Work Everyone is Required to do:** everyone keeps notebook entries for 
their own work one entry per commit or per group of commits
