# Docker Dev Environment

A Ubuntu 24.04 container with everything needed for the workbench: g++, make, cmake, OpenMP, ThreadSanitizer, Open MPI, gdb, valgrind, and Python (matplotlib, numpy, pandas). Works on x86_64 and Apple Silicon.


## Build

All commands are run from the `docker/` folder:

```bash
cd docker
docker compose build
```


## Use

Start an interactive shell in the container:

```bash
docker compose run --rm workbench
```

container. Changes are saved on both sides.

Run a single command without opening a shell:

```bash
docker compose run --rm workbench make
docker compose run --rm workbench mpirun -np 4 ./build/my_program
```

Leave the container with `exit` 
