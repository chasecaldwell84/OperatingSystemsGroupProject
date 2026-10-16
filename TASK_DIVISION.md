# Task Division: Four-Model Parallel Processing Workbench

Team: Dakota, Owen, Jack, Jonah, Chase

Work is split by workload, with shared infrastructure divided among the same people:

| Workload | Owners |
|----------|--------|
| Workload 1: Independent Loop (LOOP) | Jack and Jonah |
| Workload 2: Dot Product (DOT) | Chase and Owen |
| Workload 3: Monte Carlo Pi (PI) | Dakota |

Each person owns their own files, writes their own section of the report and their own development-notebook entries, and must be able to defend their code in the interview.

## Summary

| Member | Workload | Models | Shared infrastructure |
|--------|----------|--------|-----------------------|
| Dakota | PI | Sequential, `std::thread`, OpenMP, MPI | Config/parser, Timer, ResultValidator, `tests/` |
| Jack | LOOP | Sequential, OpenMP | WorkScheduler, output formatting, build script, Amdahl analysis |
| Jonah | LOOP | `std::thread`, MPI | MPI setup (init, config broadcast, timing), MPI docs |
| Owen | DOT | Sequential, `std::thread` | SyncUtils, race-condition demo, TraceLog |
| Chase | DOT | OpenMP, MPI | `main`, COMPARE mode, ExperimentRunner, graphs |

Jack and Jonah split LOOP so that every one of its four models has one owner, and the same goes for Owen and Chase with DOT.

---

## Dakota: Monte Carlo Pi (all four models) and Foundation

**Code**
- **PI workload:** `MonteCarloPiWorkload` with a counter-based RNG (base seed + global trial index, no shared mutable state). Every version must process exactly `SIZE` trials and produce the same samples for a given seed regardless of thread or rank assignment.
- **PI in all four models:**
  - Sequential baseline.
  - `std::thread`: per-thread hit counts with a final reduction; static, cyclic, and dynamic (synchronized chunk counter).
  - OpenMP: `reduction(+:...)` with static, cyclic, and dynamic schedules.
  - MPI: process-local hit counts combined with `MPI_Reduce`; static, cyclic, and dynamic manager-worker (rank 0 does not compute).
- PI chunk and range arithmetic is kept self-contained in the PI files so this work does not wait on `WorkScheduler`.
- `Config` and input parser: read all fields (`WORKLOAD`, `MODE`, `MODEL`, `SIZE`, `THREADS`, `PROCESSES`, `SCHEDULE`, `CHUNK`, `SEED`, `REPETITIONS`), validate them, and report `STATUS: INVALID_INPUT` / `STATUS: UNSUPPORTED_MODE`. Provide a parse-from-string function so rank 0 can broadcast the config.
- `Timer`: monotonic clock (`std::chrono::steady_clock`), with average, best, and worst over repetitions.
- `ResultValidator`: one shared validation path with no per-mode duplication (exact checksum for LOOP, tolerance for DOT, same-samples and trial-count checks for PI).

**Tests**
- `tests/` folder: 3 small + 3 large LOOP tests, 3 small + 3 large DOT tests, 3 PI tests, 3 invalid-input tests, 3 thread/process edge cases, a test for each schedule in each model, trace tests, a race-demo case, and a team-config case. Each test has its input, expected status, expected checksum or result, and what it verifies.
- A test-runner script that runs every test and reports pass/fail.

**Docs**
- README: team name, members, input format, workload explanations, known limitations, file list.
- Design document sections: project description, Monte Carlo Pi (all four models), validation, timing methodology.
- Final report sections: validation, numeric/reproducibility, Monte Carlo RNG handling per thread and per rank (including expected sampling error).
- Development notebook: kept up to date for these items.

---

## Jack: LOOP (Sequential + OpenMP), Scheduling, and Amdahl's Law

**Code**
- `LoopWorkload`: generation from the global index with no overflow (loop formula, unsigned or modular checksum, documented overflow behavior).
- LOOP sequential implementation (correctness and timing baseline).
- LOOP OpenMP implementation: `parallel for` with `schedule(static)`, `schedule(static,c)` for cyclic, and `schedule(dynamic,c)`. Document exactly which construct and schedule each strategy maps to, and report the actual team size (`omp_get_num_threads`). No `std::thread` or MPI inside this executor.
- `WorkScheduler`: shared chunk and range arithmetic (static block boundaries, cyclic chunk-to-worker mapping, dynamic chunk generation, fewer-chunks-than-workers edge cases). Document the semantics so all models define chunks consistently.
- Output formatting: the single-run report (`STATUS`, `WORKLOAD`, ..., `VALID`) and the COMPARE table (speedup and efficiency, using `p - 1` for MPI dynamic).
- `Makefile` or build script with documented commands for the sequential, `std::thread`, OpenMP, and MPI builds, plus the MPI launcher command.

**Docs**
- Design document sections: LOOP workload, sequential and OpenMP implementations, scheduling explanations (semantics shared across models).
- Final report sections: **Amdahl's Law analysis** (serial fraction, maximum speedup, observed vs ideal, one workload far from ideal and why), why more threads do not always help, memory-bandwidth limits, OS thread scheduling.
- README sections: compiler version and compile commands.
- Development notebook: OpenMP and scheduler pseudocode and notes.

---

## Jonah: LOOP (`std::thread` + MPI) and MPI Infrastructure

**Code**
- LOOP `std::thread` implementation: static (contiguous blocks), cyclic (round-robin chunks), and dynamic (synchronized chunk counter). Thread creation and join. No OpenMP directives and no pthread calls.
- LOOP MPI implementation: static (contiguous blocks per rank), cyclic (round-robin chunks per rank), and dynamic manager-worker (rank 0 hands out chunks and does not compute; needs at least 2 ranks). Checksum combined with an MPI reduction.
- MPI setup shared by all workloads: `MPI_Init_thread` with a check of the provided thread-support level, clean `MPI_Finalize` on every path (including invalid input), rank 0 reads stdin and broadcasts the validated config, `PROCESSES` is checked against the communicator size, only rank 0 prints. All ranks follow the same control flow and error path.
- MPI timing: barrier before the measured work, then maximum elapsed time across ranks.
- Document the message tags, the termination protocol, and the behavior when there are fewer chunks than workers.

**Docs**
- Design document sections: LOOP `std::thread` and MPI implementations, MPI rank decomposition, communication.
- README section: MPI ranks, data decomposition, and communication, plus MPI compile and launch instructions.
- Final report sections: process vs thread, shared memory vs message passing, ranks/communicators/collectives, why MPI processes do not share ordinary memory.
- Development notebook: protocol diagrams, MPI bugs, and fixes.

---

## Owen: DOT (Sequential + `std::thread`), Race Demo, and Trace

**Code**
- `DotProductWorkload`: deterministic arrays generated from the global element index, with a documented floating-point type and tolerance (or overflow-safe integers).
- DOT sequential implementation (correctness and timing baseline).
- DOT `std::thread` implementation: static, cyclic, and dynamic scheduling. Use per-thread local partial sums with a final reduction (no shared global sum updated per iteration), and at least one correct variant using a mutex or atomic.
- `SyncUtils` (mutex wrapper, atomic counter, optional barrier or condition variable).
- Controlled race-condition demo, kept separate from the correct code: an unsafe shared-sum DOT, a corrected mutex version, and a corrected per-thread-partial-sum version. Include a ThreadSanitizer build target and a script that runs the unsafe version repeatedly to show nondeterminism.
- `TraceLog` and TRACE mode: real, not hard-coded, traces for `std::thread`, OpenMP, and MPI (create, assign range or chunk, start, partial, join, reduce). Thread-safe logging. Small inputs only, and never used for timing. Owen builds the log class; the other executors call it.

**Docs**
- Design document sections: DOT workload, sequential and `std::thread` implementations, synchronization choices, trace mode.
- Final report sections: thread creation and joining, shared vs private data, synchronization strategy, race-condition demonstration (explanation, evidence, why the fix is valid), process vs thread, shared address space, cost of excess locking.
- Development notebook: race observations, bugs, and fixes.

---

## Chase: DOT (OpenMP + MPI), Program Control, and Experiments

**Code**
- DOT OpenMP implementation: `parallel for` with `reduction(+:sum)` and static, cyclic (`static,c`), and dynamic (`dynamic,c`) schedules. Report the actual team size.
- DOT MPI implementation: process-local partial sums combined with `MPI_Reduce`, with static, cyclic, and dynamic manager-worker assignment. Uses Jonah's shared MPI setup.
- `main`: parse, dispatch by mode, and handle the SEQUENTIAL / STD_THREAD / OPENMP / MPI / COMPARE / TRACE modes. Check environment compatibility (MPI launcher, process count, at least 2 ranks for COMPARE) and return the correct status for unsupported modes.
- COMPARE mode: run all four models with the same schedule and chunk size and the same number of repetitions, with the documented synchronization point for non-zero ranks.
- `ExperimentRunner`: batch runs over at least 3 input sizes, at least 4 worker counts, at least 3 repetitions, and all three schedules, writing results to CSV.
- Graph scripts (for example Python/matplotlib) that read the CSV, with no hard-coded values: for each workload, time vs input size and time vs worker count across all four models, with error bars and labeled axes, and the machine, schedule, and chunk size stated on each graph.

**Docs**
- Design document sections: DOT OpenMP and MPI implementations, program control.
- README sections: run commands for each mode, output format, how to reproduce the graphs.
- Final report sections: timing tables, speedup/efficiency tables, graphs, model and schedule comparisons, measurement variability, cases where parallel is slower than sequential, machine description (compiler, OpenMP runtime, MPI implementation, cores, process placement).
- Demonstration script and materials for the final demo.
- Development notebook: timing results and timing anomalies.

---

## Shared Responsibilities (Everyone)

- Keep an individual, incremental development notebook (meeting notes, design decisions, bugs, fixes, failed and successful tests).
- Write your own part of the contribution statement.
- Review at least one other member's code before each milestone.
- Be ready to answer code-defense questions about the *entire* project, not just your own part, and to make a small live modification.
- Team-specific configuration sheet (when issued): each owner updates their own workload or model, and Dakota handles parsing changes.

## Interface Agreements

Agree on these early so modules plug together:
- Dakota publishes the headers for `Config`, `Timer`, and the result struct first.
- Jack publishes `WorkScheduler` function signatures first.
- Jonah publishes the MPI setup interface (init, broadcast, timing, finalize) first, since Chase's DOT MPI code and Dakota's PI MPI code call it.
- Owen publishes the `TraceLog` interface first so every executor can call it.
- Each executor exposes the same call shape, taking a `Config` and returning a result struct (value, checksum, per-run times, active workers).

## Milestone Ownership

| Milestone | Deliverable | Lead |
|-----------|-------------|------|
| 1: Design Proposal | Module design, partitioning strategies, validation and timing plans, OS concepts | All (each writes their own section; Dakota assembles) |
| 2: Sequential + Shared-Memory Baselines | Parsing, sequential, `std::thread`, OpenMP, basic validation and timing for all three workloads | Dakota, Jack, Jonah, Owen, Chase (their own models) |
| 3: MPI + Trace | MPI executors, config broadcast, reductions, trace mode | Jonah, Chase, Dakota, Owen |
| 4: Sync, Race Demo, Empirical Draft | Race demo, repeated trials, draft speedup tables, one slower-than-sequential case | Owen, Chase, Jack |
| 5: Final Project | Everything, plus README, design document, report, notebook, contribution statement, demo | All |
