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