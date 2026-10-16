CSI 4337 Fall 2026 Group Project Version 2
Jump to navigationJump to search
Four-Model Parallel Processing Workbench

Practical Resources

LLNL HPC Tutorials - OpenMP
MPI Tutorial
Project Overview

In this group project, students will design, implement, analyze, test, and defend a computational workbench that compares four execution models: sequential execution, C++11 std::thread, OpenMP, and MPI. The project connects shared-memory threading and distributed-memory message passing to operating-system concepts.
The project is motivated by three introductory OpenMP examples:
A parallel loop
A parallel dot product using reduction
A Monte Carlo estimate of pi using independent random trials
These examples show that some computations can be parallelized easily, while others require careful treatment of shared data, synchronization, reduction, scheduling, and per-thread state.
Students will begin with these examples as conceptual models, but the project requires a deeper operating-systems treatment. For each required workload, each group must implement all four models: a sequential baseline, a C++11 std::thread version, an OpenMP version, and an MPI version. Groups must compare correctness and performance, explain the operating-system concepts involved, and defend the design.
A correct final program alone is not sufficient for full credit. Students must demonstrate that they understand thread creation, shared memory, race conditions, synchronization, scheduling, reductions, timing, scalability, and the interaction between user-level code and operating-system thread scheduling.
Purpose

The purpose of this project is to connect parallel programming constructs to the underlying operating-system concepts that make parallel execution possible.
This project emphasizes:
Processes and threads
Shared-memory models versus distributed-memory MPI
Shared address spaces
C++11 thread creation and joining
MPI processes, ranks, communicators, and message passing
OpenMP parallel regions
Distributed-memory data partitioning and MPI collectives
Work distribution across threads
Race conditions
Critical sections
Mutexes
Reductions
Per-thread local storage
Thread scheduling and MPI work distribution
Load balancing
Timing and performance measurement
Speedup and efficiency
Amdahl's Law
Modular C++ design
Group software development
Testing, debugging, and code defense
Motivating Example

The project is motivated by the following OpenMP ideas:
#pragma omp parallel for
distributes independent loop iterations across multiple threads.
reduction(+:sum)
safely combines partial results from multiple threads.
In a Monte Carlo computation, each trial is largely independent, but random-number generation must be handled carefully so that threads do not interfere with one another.
The goal of this project is to compare what each model provides and what the programmer must manage: OpenMP directives, explicit std::thread coordination, and MPI communication among processes with separate address spaces.
Required Workloads

Each group must implement the following three workloads in all four computational models: sequential, C++11 std::thread, OpenMP, and MPI.
Workload 1: Independent Loop Computation
The program must fill an array using a computation where each index is independent.
For example:
data[i] = (i * i + 3 * i + 7) % M;
The exact formula may be assigned by the instructor as part of the team-specific configuration. Use an integer type and arithmetic that avoid signed overflow; define any checksum accumulation and overflow behavior explicitly.
This workload demonstrates:
Independent loop iterations
Static work partitioning
Minimal synchronization
Memory access patterns
Worker-thread creation or MPI rank participation overhead for small tasks
Workload 2: Dot Product with Reduction
The program must compute the dot product of two arrays:
sum = a[0] * b[0] + a[1] * b[1] + ... + a[n-1] * b[n-1]
The implementation must include all four models:
A sequential version
An OpenMP version using a reduction
A C++11 std::thread version using manual partial sums
An MPI version using process-local partial sums and a collective reduction
The std::thread version must avoid updating a shared global sum on every iteration. The MPI version must compute process-local partial sums and combine them with an MPI collective such as MPI_Reduce.
This workload demonstrates:
Shared data
Reduction
Partial sums
Synchronization avoidance
False sharing concerns
Numerical consistency issues
Workload 3: Monte Carlo Estimate of Pi
The program must estimate pi using randomly generated points in the unit square.
The implementation must include all four models:
A sequential version
An OpenMP version
A C++11 std::thread version
An MPI version using process-local trial counts and a collective reduction
Each worker must generate points from an independent, deterministic stream keyed by the base seed and global trial index. No mutable random-number state may be shared among workers.
Use a deterministic random-number mapping from the base seed and global trial index so all four models generate the same points, independent of thread or rank assignment. Do not use a shared mutable random-number generator.
This workload demonstrates:
Embarrassingly parallel computation
Per-thread state
Reduction of counts
Random-number generation in parallel programs
Scalability under independent work
Required Computational Models

Every workload must have four correct implementations. These are the four required computational models; they are not optional alternatives.
Model	Required Meaning
SEQUENTIAL	One process and one execution thread. This is the correctness baseline and the timing baseline.
STD_THREAD	One process creates and joins C++11 std::thread workers. Work partitioning and synchronization are implemented explicitly by the group.
OPENMP	One process uses OpenMP directives and runtime scheduling. The group must identify the OpenMP constructs and schedule used.
MPI	Multiple MPI processes, each with a separate address space, divide the work and communicate results using MPI operations.
The sequential implementation must not create worker threads or MPI workers. The std::thread implementation must use C++11 std::thread, not POSIX thread functions such as pthread_create. The OpenMP implementation must not create std::thread or MPI workers. The MPI implementation must use MPI processes and MPI communication; it may not substitute local threads for its required distributed work.
The std::thread and OpenMP implementations must support static, cyclic, and dynamic work assignment. The MPI implementation must support static and cyclic rank assignment and a dynamic manager-worker chunk assignment. For MPI dynamic assignment, rank 0 acts as the manager and distributes chunks to worker ranks; this design requires at least two MPI processes, and rank 0 does not compute workload chunks. Use the same workload inputs and define scheduling/chunk semantics consistently across models where applicable.
Required Scheduling Strategies

Strategy	Required Meaning
STATIC	Divide the iteration range into predetermined contiguous blocks, with at most one block per worker.
CYCLIC	Assign iterations or fixed-size chunks in round-robin order to workers.
DYNAMIC	Assign the next available fixed-size chunk at run time. std::thread and OpenMP use a synchronized counter or runtime schedule; MPI uses a manager-worker message protocol.
MPI Execution Requirements

The MPI implementation uses separate processes. Each process has its own address space; data is not shared automatically. The program must initialize MPI before using MPI routines and finalize MPI after all MPI work is complete. Use MPI_Init_thread with at least MPI_THREAD_FUNNELED when the executable also runs std::thread or OpenMP work under MPI; only the main thread may call MPI routines, and the program must verify the provided thread-support level. Rank 0 reads standard input and broadcasts the validated configuration to the other ranks so that all processes follow the same control flow and enter collectives in the same order.
For static MPI assignment, divide the global index range into contiguous blocks across ranks. For cyclic assignment, assign fixed-size chunks to ranks in round-robin order. For dynamic assignment, rank 0 distributes chunks to worker ranks using MPI messages and gathers their partial results. Rank 0 does not compute chunks in dynamic mode; account for the resulting p - 1 compute workers when reporting efficiency. Document the message tags, termination protocol, and behavior when the input has fewer chunks than workers. Rank 0 is the only process that prints the final report. Do not time input, output, trace printing, or MPI initialization/finalization as part of the workload; include the synchronization and communication needed to complete the measured MPI computation.
Use MPI collectives such as MPI_Reduce or MPI_Allreduce where they fit the computation. For the loop checksum and dot product, combine integer or floating-point partial results with a documented reduction operation. For Monte Carlo pi, reduce the total number of points inside the circle and divide by the total number of trials. Make global array data available only when needed; do not assume that an MPI process can read another process's local memory.
Required Operating-System Concepts

The final report and code defense must address the following operating-system concepts:
Difference between a process and a thread
Difference between shared-memory threading and MPI processes
MPI ranks, communicators, message passing, and collective operations
Why threads in the same process share an address space
What data is shared among threads
What data should be private to each thread
How thread creation and joining work
Why a race condition occurs
How mutexes protect critical sections
Why excessive locking may reduce performance
How reduction reduces synchronization overhead
How the operating system schedules runnable threads
Why more threads do not always produce better performance
How load balancing affects completion time
How memory bandwidth may limit parallel speedup
Why timing results vary between runs
How Amdahl's Law limits speedup
Required Input Format

The program must read a test configuration from standard input. In MPI runs, rank 0 reads it and broadcasts the configuration. All ranks must take the same error path for invalid input and must finalize MPI cleanly.
The required input format is:
WORKLOAD workload
MODE mode
[MODEL model]
SIZE n
THREADS t
PROCESSES p
SCHEDULE schedule
CHUNK c
SEED s
REPETITIONS r
where:
workload is one of LOOP, DOT, or PI.
model is required only when mode is TRACE; it is one of STD_THREAD, OPENMP, or MPI.
mode is one of SEQUENTIAL, STD_THREAD, OPENMP, MPI, COMPARE, or TRACE. The SCHEDULE field selects the scheduling strategy for applicable parallel modes.
n is the number of loop iterations, array elements, or Monte Carlo points.
t is the requested number of C++11 threads or OpenMP threads.
p is the MPI communicator size required for MPI and comparison modes. MPI dynamic scheduling requires at least two ranks; comparison mode also requires at least two ranks. For non-MPI modes, this field is ignored.
schedule is one of STATIC, CYCLIC, or DYNAMIC.
c is a positive chunk size for cyclic or dynamic scheduling. Static block size is determined by the workload size and worker count; CHUNK is ignored for static scheduling.
s is the base random seed.
r is the number of repeated timing runs. SIZE n gives the number of Monte Carlo samples for workload PI.
Some fields may not be relevant for all modes, but the parser must handle the full configuration consistently. In MPI and comparison modes, rank 0 reads the complete configuration and broadcasts it. All ranks must validate that the requested process count equals the MPI communicator size. MODEL is omitted for modes other than TRACE.
Example Input

WORKLOAD DOT
MODE COMPARE
SIZE 10000000
THREADS 4
PROCESSES 4
SCHEDULE STATIC
CHUNK 1000
SEED 12345
REPETITIONS 5
Required Output Format

The program must print a clear report from rank 0 only when running under MPI; a non-MPI run prints from its sole process.
For a single run, the output must include the model-specific worker count. Use THREADS for OpenMP or std::thread, and PROCESSES for MPI. Report average, best, and worst elapsed time over REPETITIONS runs. The output must include:
STATUS: OK
WORKLOAD: DOT
MODE: STD_THREAD
SIZE: 10000000
WORKERS: 4
SCHEDULE: STATIC
RESULT: value
CHECKSUM: value
AVG_TIME_MS: value
BEST_TIME_MS: value
WORST_TIME_MS: value
REPETITIONS: r
VALID: YES
For comparison mode, the output must include one report line for each implementation version. Use the same schedule and chunk size for all applicable models. Report p active workers for static/cyclic MPI and p - 1 for dynamic MPI:
VERSION          ACTIVE_WORKERS   RESULT        CHECKSUM      AVG_MS   BEST_MS   WORST_MS   SPEEDUP   EFFICIENCY
SEQUENTIAL       1         value         value         value   value     value      1.000     1.000
STD_THREAD       t         value         value         value   value     value      value     value
OPENMP           t         value         value         value   value     value      value     value
MPI              p         value         value         value   value     value      value     value
If the input is invalid, the program must print:
STATUS: INVALID_INPUT
If the requested mode is not supported, the program must print:
STATUS: UNSUPPORTED_MODE
Required Program Modes

Mode	Required Behavior
SEQUENTIAL	Run the selected workload using one process and one thread.
STD_THREAD	Run the workload using C++11 std::thread workers with the selected scheduling strategy.
OPENMP	Run the workload using OpenMP with the selected schedule and chunk size.
MPI	Run the workload using MPI processes and the selected rank-level work assignment.
COMPARE	Run and compare all four models. Under MPI, all ranks participate in the MPI run; rank 0 runs and times the sequential, std::thread, and OpenMP implementations while the other ranks wait at a documented synchronization point.
TRACE	Print an actual execution trace for the selected parallel model, including thread IDs or MPI ranks and work assignments. Use small inputs only.
The program must validate that the requested mode is compatible with its execution environment. For MPI and COMPARE, the program must be launched with an MPI launcher, and the configured process count must match the MPI communicator size. COMPARE must use at least two MPI ranks so that all four models can be compared. For SEQUENTIAL, STD_THREAD, and OPENMP, run one process. Invalid or unavailable modes must produce the specified status rather than silently running another model.
Required Trace Mode

The project must include a trace mode that demonstrates work assignment and completion in the selected parallel model. When MODE TRACE is used, the required MODEL field selects STD_THREAD, OPENMP, or MPI.
For small inputs, trace mode must show:
Worker-thread creation or MPI rank participation
Thread ID or logical thread number, or MPI rank
Assigned work range or assigned chunk
Start of work
Completion of work
Local partial result, when applicable
Final reduction or result combination
Thread join or MPI completion/collective
Example trace output:
TRACE: CREATE thread 0
TRACE: CREATE thread 1
TRACE: ASSIGN thread 0 range [0, 499]
TRACE: ASSIGN thread 1 range [500, 999]
TRACE: START thread 0
TRACE: START thread 1
TRACE: PARTIAL thread 0 value 128391
TRACE: PARTIAL thread 1 value 129004
TRACE: JOIN thread 0
TRACE: JOIN thread 1
TRACE: REDUCE final value 257395
Trace output must reflect actual work assignment performed by the program.
Hard-coded trace output is not permitted. An MPI trace must identify ranks and the actual chunks assigned to each rank.
Required Race-Condition Demonstration

Each group must include a controlled race-condition demonstration.
This demonstration must be separate from the correct implementations.
The race-condition demonstration should show what happens when multiple threads update shared data incorrectly.
For example, an unsafe dot-product version may update a shared sum without a mutex or reduction.
The demonstration must include:
An explanation of why the code is unsafe
Evidence that the result may be incorrect or nondeterministic
A corrected version using either a mutex or per-thread partial sums
A discussion of why the corrected version is valid
The unsafe version must not be used for the final reported correct workload result. Compile and run it only in a controlled demonstration configuration; use a thread sanitizer where available, and do not claim that every run must produce a wrong result.
Required Synchronization Techniques

Each group must use and explain at least two of the following:
Mutex
Condition variable
Barrier
Atomic operation
Per-thread local storage with final reduction
MPI point-to-point messages and collective reductions
At least one correct std::thread implementation must use a mutex or atomic operation. The controlled race-condition demonstration must use shared-memory threads; MPI processes do not share ordinary variables, so an MPI communication error is not a data race.
At least one correct std::thread implementation must avoid frequent locking by using per-thread local results. The MPI implementation must combine process-local results using MPI communication.
Required Timing Methodology

The program must measure execution time using a consistent timing method. Run the same number of repetitions for every model in a comparison.
The timing report must include:
Input size
Number of active workers (threads for std::thread/OpenMP; ranks for MPI)
Version used
Number of repeated timing runs
Average execution time
Best execution time
Worst execution time
Speedup relative to the sequential version
Parallel efficiency, computed as speedup divided by active worker count (for MPI dynamic mode, use p - 1)
Speedup is defined using average times across the same number of repetitions:
speedup = sequential_average_time / parallel_average_time
Parallel efficiency is defined as:
efficiency = speedup / active_worker_count
Timing must not include time spent printing large amounts of output.
Trace mode must not be used for performance measurement.
Required Empirical Study

Each group must conduct an empirical study.
The empirical study must include:
At least three input sizes for each workload
At least four worker counts, when available, such as 1, 2, 4, and 8 threads for std::thread/OpenMP or MPI ranks
At least three repeated timing trials for each configuration
A comparison of sequential, C++11 std::thread, OpenMP, and MPI implementations
A comparison of static, cyclic, and dynamic scheduling for std::thread, OpenMP, and MPI
A discussion of speedup and efficiency
A discussion of cases where parallel execution is slower than sequential execution
A discussion of workload size and thread-creation overhead
A discussion of load imbalance
A discussion of measurement variability
Graphical displays of the measured results for each workload, with all four computational models represented
For each workload, include at least two labeled graphs:
Execution time versus input size, comparing the sequential, C++11 std::thread, OpenMP, and MPI models at a fixed worker count and scheduling policy where applicable.
Execution time versus worker count, comparing the four models while varying C++11/OpenMP thread count or MPI rank count for one or more fixed input sizes.
Use consistent units and clearly labeled axes. Show variability across repeated runs with error bars or another clearly explained summary. If scheduling strategies are compared, show the strategies as separate series or panels. State the workload, schedule, chunk size, worker counts, input sizes, and machine configuration represented in each graph. For MPI dynamic scheduling, distinguish the MPI rank count from the active worker count (p - 1). The graphs must be generated from the measured data, not illustrative or hard-coded values.
The empirical study must be performed on a consistent test machine or server, and the machine used must be identified in the report. Identify the compiler, OpenMP runtime, MPI implementation, processor/core count, and MPI process placement. Compare the four models on the same host unless the instructor approves a documented multi-node experiment.
Required Amdahl's Law Analysis

Each group must include an Amdahl's Law discussion.
The discussion must include:
The theoretical meaning of the serial fraction
Why the maximum possible speedup is limited
A comparison between observed speedup and ideal linear speedup for the relevant worker count
At least one workload where the observed speedup is far from ideal
A reasonable explanation for the gap between theoretical and observed performance
The group does not need to prove an exact serial fraction, but the explanation must be consistent with the measured data.
Numeric and Reproducibility Requirements

Generate workload inputs from the global element or trial index so that all implementations use the same input data. For the loop workload, define the checksum as a documented reduction over the generated array; use unsigned arithmetic or explicit modular arithmetic so its behavior is defined and independent of scheduling. Generate both dot-product arrays deterministically from each global element index. Choose input ranges that prevent integer overflow, or use a documented floating-point type and tolerance. Floating-point reductions may differ slightly because addition is not associative; validation must use an explicit tolerance rather than require bit-for-bit equality.
For Monte Carlo pi, every implementation must process exactly the requested total number of samples, given by SIZE. Derive reproducible random values from the base seed and global trial index, using a documented counter-based method or equivalent mapping so dynamic assignment does not change the samples. State which reproducibility guarantee is provided. The sequential and parallel versions must use the same point samples for a given seed and trial count, allowing direct correctness comparison.
Required Correctness and Validation

The project must include a validation strategy for each workload. The same deterministic input data must be used by all four models.
Loop Workload Validation
The loop workload must produce a checksum of the output array.
The checksum must be independent of thread count and scheduling method.
Dot Product Validation
The dot-product result from each parallel version must be compared against the sequential version.
For floating-point results, the comparison may use a documented tolerance.
Monte Carlo Pi Validation
The Monte Carlo estimate is approximate, but the same seed and sample count must generate the same points across all four models.
The validation must check that:
Each version completes the requested number of trials.
Each version derives the same point samples from the base seed and trial index without shared mutable random state.
The estimate agrees with the sequential result when the same samples are used; also discuss expected sampling error.
Repeated runs with the same seed and configuration are reproducible according to the documented random-stream method.
Required Architecture

The implementation must be modular.
At minimum, the project must include logical separation among:
Input parsing
Workload generation
Sequential implementations
OpenMP implementations
C++11 std::thread implementations
MPI implementations
Work scheduling
Synchronization utilities
Result validation
Timing
Trace generation
Output formatting
Experiment execution
Main program control
Suggested classes or modules include:
class Config;
class Timer;
class Workload;
class LoopWorkload;
class DotProductWorkload;
class MonteCarloPiWorkload;
class SequentialExecutor;
class OpenMPExecutor;
class StdThreadExecutor;
class MPIExecutor;
class WorkScheduler;
class TraceLog;
class ResultValidator;
class ExperimentRunner;
These names are suggestions, not requirements. However, the final design must demonstrate clear separation of responsibility.
Required Build and Execution Environment

The submission must include a build procedure for a C++11 compiler, an OpenMP-capable compiler/runtime, and an MPI implementation. A single compiler wrapper may not enable both OpenMP and MPI on every platform; document the exact commands used on the group's test machine and verify that the resulting executable actually includes all required models. The README must state how to compile and launch sequential, std::thread, OpenMP, and MPI modes, including an MPI launcher command and the required process count.
Use a monotonic clock for elapsed-time measurements. Start timing immediately before workload execution and stop after the result is complete, including thread creation/join, scheduling, synchronization, and required MPI communication. Exclude input parsing, test-data setup, MPI initialization/finalization, and output. For MPI timing, synchronize ranks immediately before the measured work and report the maximum elapsed time across ranks, since the parallel job completes when its slowest rank finishes. Explain any timing setup that differs between models. Thread counts refer to worker threads in one process; MPI process count refers to ranks. Use the actual OpenMP team size in the report and efficiency calculation. Avoid oversubscription in the primary comparison and report any deviation.
Required Design Constraints

The final project must satisfy the following design constraints:
No hard-coded timing results.
No hard-coded correctness results.
No artificial sleep calls to create misleading timing behavior.
No global shared result updated unsafely in correct implementations.
No shared random-number generator in the Monte Carlo correct implementation.
No OpenMP directives inside the std::thread implementation.
No std::thread creation inside the OpenMP implementation.
No std::thread or OpenMP workers substituted for MPI processes in the MPI implementation.
No duplicated validation logic for each mode.
No program behavior that depends on the name of the input file.
No hidden files required for compilation.
No use of AI tools.
No use of outside code unless explicitly authorized by the instructor.
The program must compute results from the input configuration, not from assumptions about instructor test cases.
Team-Specific Configuration Requirement

Each group will receive a short configuration sheet from the instructor after the design proposal is submitted.
The configuration sheet may specify requirements such as:
A required loop formula
A required checksum formula
A required random-number generator
A required thread-count set
A required scheduling policy
A required chunk-size rule
A required trace format
A required malformed-input case
A required race-condition demonstration
A required synchronization primitive
A required performance experiment
A required additional workload
The team-specific configuration must be implemented and documented.
Failure to implement the team-specific configuration will result in a substantial deduction, even if the general program works.
AI-Resistance Design

This project includes several requirements intended to make it difficult to complete successfully without genuine understanding of operating-system concepts.
Each group must complete:
A team-specific configuration requirement assigned by the instructor.
Multiple implementation versions of the same workloads.
Required trace mode reflecting actual thread behavior.
Required race-condition demonstration.
Required performance measurements on a real machine.
Required empirical study.
Required Amdahl's Law discussion.
Multiple milestone submissions.
A development notebook documenting design choices, bugs, timing anomalies, and fixes.
An individual code-defense interview.
A live modification task during the final demonstration.
A program that merely produces plausible output will not receive full credit unless the group can explain, test, measure, analyze, and modify the implementation.
Required Development Milestones

The project must be submitted in stages.
Milestone 1: Design Proposal
Submit:
Team name
Names of all group members
Proposed class or module design
Proposed workload representation
Proposed thread and rank partitioning strategies
Proposed validation strategy
Proposed timing methodology
Initial explanation of relevant operating-system concepts
Planned division of responsibilities
No code is required for this milestone.
Milestone 2: Sequential and Shared-Memory Baselines
Submit a version that compiles and runs.
This version must support:
Input parsing
Sequential implementation of all three workloads
C++11 std::thread implementation of all three workloads
OpenMP implementation of all three workloads
Basic validation
Basic timing output
MPI implementation is not required for this milestone.
Milestone 3: MPI and Trace Mode
Submit a version that supports:
MPI initialization and finalization
MPI configuration broadcast and rank-level work assignment
MPI result reduction for all three workloads
Trace mode for small inputs
Validation against the sequential version
The race-condition demonstration and full empirical study are not required for this milestone.
Milestone 4: Synchronization, Race Demonstration, and Empirical Study Draft
Submit a version that supports:
Controlled race-condition demonstration
Corrected synchronization strategy
Repeated timing trials
Initial speedup and efficiency tables
Initial discussion of timing results
At least one case where parallel execution is slower than sequential execution
Milestone 5: Final Project
Submit the complete project with:
Sequential implementations
OpenMP implementations
C++11 std::thread implementations
MPI implementations
Static, cyclic, and dynamic scheduling in each required parallel model
Correct synchronization
Trace mode
Race-condition demonstration
Validation system
Timing system
Team-specific configuration
Documentation
Empirical study
Amdahl's Law analysis
Development notebook
Contribution statement
Demonstration materials
Development Notebook Requirement

Each group must maintain a development notebook.
The notebook may be handwritten and scanned or maintained electronically.
The notebook must include:
Meeting notes
Design decisions
Thread-partitioning sketches
Shared-data diagrams
Synchronization diagrams
Pseudocode for each workload
Pseudocode for each scheduling strategy
Race-condition observations
Bugs encountered
Fixes attempted
Failed test cases
Successful test cases
Timing results
Timing anomalies
Notes from each milestone
Individual contribution notes
The notebook must show evidence of incremental development.
A notebook created only at the end of the project will receive little or no credit.
Required Test Cases

Each group must create a test folder containing at least:
Three small and three large loop-workload tests
Three small and three large dot-product tests
Three Monte Carlo pi tests
Three invalid-input tests
Three thread-count or MPI-process-count edge cases
Tests for static, cyclic, and dynamic scheduling in each applicable model
At least one trace-mode test for each supported trace model
At least one controlled race-condition demonstration case
At least one team-configuration-specific test case
Each test case must include its input, expected status, expected checksum or validation result when applicable, and a statement of what it verifies.
Required README

The README must include:
Team name
Names of all group members
Compiler version used
Full compile command
Instructions for running each mode
Explanation of the input format
Explanation of the output format
Explanation of the workloads
Explanation of the scheduling options
MPI compile and launcher commands
Explanation of MPI ranks, data decomposition, and communication
Explanation of the trace mode
Description of the empirical graphs and how to reproduce them
Explanation of the race-condition demonstration
Known limitations
List of files included in the submission
Required Design Document

The design document must include:
Team name
Names of all group members
Description of the project
Explanation of each workload
Explanation of the sequential implementations
Explanation of the OpenMP implementations
Explanation of the C++11 std::thread implementations
Explanation of the MPI implementation, rank decomposition, and communication
Explanation of static scheduling
Explanation of cyclic scheduling
Explanation of dynamic scheduling
Explanation of synchronization choices
Explanation of validation
Explanation of timing methodology
Description of the team-specific configuration
Known limitations, if any
Instructions for running the program
Required Final Report

The final report must include:
Design overview
Explanation of relevant operating-system concepts
Description of thread creation and joining
Description of shared and private data
Description of synchronization strategy
Description of race-condition demonstration
Timing tables
Graphs comparing the execution-time effects of data size and worker count across all four computational models
Speedup and efficiency tables
Comparison of sequential, C++11 std::thread, OpenMP, and MPI versions
Comparison of scheduling strategies in std::thread, OpenMP, and MPI
Amdahl's Law discussion
Discussion of measurement variability
Discussion of cases where parallel execution performed poorly
Development summary
Group contribution summary
Code Defense Requirement

Each student must be prepared to answer questions about the entire project.
During the code defense, the instructor may ask any group member to explain:
How a C++11 std::thread is created
How thread arguments are passed and how MPI ranks receive the configuration
How the main thread joins worker threads and how MPI processes coordinate completion
What memory is shared among threads
What data is private to each thread
Why a race condition occurs
How the race-condition demonstration works
How the corrected version avoids the race condition
How the dot-product reduction is implemented
How the Monte Carlo random-number state is handled per thread and per rank
How MPI data is distributed and partial results are reduced
Why MPI processes do not share ordinary memory
How static scheduling differs from cyclic scheduling in std::thread, OpenMP, and MPI
How dynamic scheduling assigns work to threads and MPI worker ranks
Why too many threads may reduce performance
Why timing results vary
How speedup and efficiency are computed
How Amdahl's Law applies to the results
What part of the code the student personally wrote or modified
How a specific bug was found and fixed
The instructor may also ask a student to make a small live modification, such as:
Change the number of threads
Add a new chunk size
Change the loop formula
Change the checksum formula
Add an additional trace field
Replace a mutex-based reduction with a per-thread partial-sum reduction
Add a validation check
Disable dynamic scheduling and explain the result
Explain and correct a deliberately introduced small race condition
A student who cannot explain the submitted code may receive a reduced individual grade, even if the group project produces correct output.
Demonstration Requirement

The final demonstration must include:
Running the loop workload in all four models.
Running the dot-product workload in all four models.
Running the Monte Carlo pi workload in all four models.
Demonstrating trace mode on a small input.
Demonstrating the controlled race condition.
Demonstrating the corrected synchronization strategy.
Showing timing results for at least two thread counts and two MPI process counts, when the test environment permits.
Presenting the required graphs and explaining the effects of input size and worker count for all four models.
Showing one case where parallel execution is faster than sequential execution.
Showing one case where parallel execution is slower than sequential execution.
Explaining one empirical result.
Answering code-defense questions.
Grading Rubric

Category	Points
Correct sequential implementations	6
Correct OpenMP implementations	8
Correct C++11 std::thread implementations	10
Correct MPI implementations	10
Correct static, cyclic, and dynamic scheduling across required models	8
Correct synchronization and reduction design	8
Race-condition demonstration and correction	8
Trace mode and thread/rank behavior reporting	6
Validation and correctness testing	7
Timing methodology, graphical analysis, speedup, and efficiency	8
Operating-system concept analysis	7
Code organization, modularity, and style	5
Design document, README, notebook, and report	5
Demonstration, contribution statement, and code defense	4
Total	100
Individual Grade Adjustment

The group receives a project score. Individual grades may be adjusted based on:
Contribution statement
Development notebook evidence
Milestone participation
Commit or file-history evidence, if requested
Demonstration participation
Code-defense performance
A student who cannot explain the submitted project may receive a substantially lower individual grade than the group score.
Major Penalties

The following may result in substantial point deductions:
Submitting code that does not compile
Failing to implement the sequential baseline
Failing to implement the OpenMP versions
Failing to implement the C++11 std::thread versions
Failing to implement the MPI versions
Hard-coding timing results
Hard-coding correctness results
Using unsafe shared data in the correct implementation
Using a shared random-number generator incorrectly
Failing to include trace mode
Failing to include the race-condition demonstration
Reporting speedup without a valid sequential baseline
Including printing time in performance measurements
Providing an analysis inconsistent with the code
Failing to implement the team-specific configuration
Failing to participate meaningfully in the code defense
Academic Integrity and Collaboration

This is a group project. Students are expected to collaborate within their assigned group.
Each student must understand the full design and be able to explain the operating-system concepts, parallel-programming techniques, synchronization strategy, testing strategy, timing methodology, and performance analysis used by the group.
Use of outside code is not permitted unless explicitly authorized by the instructor.
Use of AI tools is not permitted.
Students may be asked to explain any part of the submitted code. A correct program that the students cannot explain will not receive full credit.
Submission Checklist

Before submitting, confirm that:
Complete?	Item
☐	The project compiles successfully with all four required models enabled.
☐	The program reads a configuration from standard input.
☐	The program supports the loop workload in all four models.
☐	The program supports the dot-product workload in all four models.
☐	The program supports the Monte Carlo pi workload in all four models.
☐	Sequential versions are implemented.
☐	OpenMP versions are implemented.
☐	C++11 std::thread versions are implemented.
☐	MPI versions are implemented and tested with multiple ranks.
☐	Static scheduling is implemented for each required parallel model.
☐	Cyclic scheduling is implemented for each required parallel model.
☐	Dynamic scheduling is implemented for each required parallel model.
☐	Results are validated against the sequential baseline where appropriate.
☐	The Monte Carlo workload uses identical deterministic samples across all four models.
☐	Trace mode is implemented.
☐	The race-condition demonstration is included.
☐	Timing excludes large-output printing.
☐	Speedup and efficiency are reported for all four models.
☐	Graphs show execution time versus input size and worker count for all four models.
☐	The team-specific configuration is implemented.
☐	The README is included, with build and launch commands for all four models.
☐	The design document is included.
☐	The empirical study is included, with labeled graphs comparing all four models across input sizes and worker counts.
☐	The Amdahl's Law discussion is included.
☐	The development notebook is included.
☐	The group contribution statement is included.
☐	The test cases are included.
☐	Each student is prepared for the code defense.
Instructor Notes

This assignment connects introductory parallel-programming examples to operating-system concepts. OpenMP simplifies parallel execution, but the operating system still creates, schedules, and manages threads. C++11 std::thread requires students to manage more of that work explicitly, while MPI requires them to distribute data and communicate results among processes.
Students should understand what OpenMP automates, what C++11 std::thread requires the programmer to manage explicitly, and how MPI differs because processes do not share ordinary memory. Synchronization, scheduling, data distribution, communication, and measurement methodology are central to correct parallel processing.
The central assessment goal is to determine whether students can design, implement, measure, analyze, and defend parallel programs using operating-system concepts.
