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