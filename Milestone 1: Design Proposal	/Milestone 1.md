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

## 2. Proposed Workload Representation

### Workload 1: LOOP (independent loop)

### Workload 2: DOT (dot product)

### Workload 3: PI (Monte Carlo)

## 3. Proposed Thread and Rank Partitioning Strategies

## 4. Proposed Validation Strategy

## 5. Proposed Timing Methodology

## 6. Initial Explanation of Relevant Operating-System Concepts

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
