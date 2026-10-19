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

Monte Carlo Pi Validation
The Monte Carlo estimate is approximate, but the same seed and sample count must generate the same points across all four models.
The validation must check that:
Each version completes the requested number of trials.
Each version derives the same point samples from the base seed and trial index without shared mutable random state.
The estimate agrees with the sequential result when the same samples are used; also discuss expected sampling error.
Repeated runs with the same seed and configuration are reproducible according to the documented random-stream method.