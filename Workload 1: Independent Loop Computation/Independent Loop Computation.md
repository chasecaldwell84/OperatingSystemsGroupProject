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