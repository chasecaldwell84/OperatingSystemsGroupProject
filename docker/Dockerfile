# Dev environment for the Four-Model Parallel Processing Workbench
# (sequential, C++11 std::thread, OpenMP, MPI). Works on x86_64 and Apple Silicon.
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential g++ make cmake \
        libomp-dev libtsan2 \
        openmpi-bin libopenmpi-dev \
        gdb valgrind git vim nano time \
        python3 python3-pip python3-matplotlib python3-numpy python3-pandas \
    && rm -rf /var/lib/apt/lists/*

# Containers run as root: allow mpirun, and avoid shared-memory/CMA warnings.
ENV OMPI_ALLOW_RUN_AS_ROOT=1 \
    OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1 \
    OMPI_MCA_btl_vader_single_copy_mechanism=none \
    OMPI_MCA_rmaps_base_oversubscribe=0

WORKDIR /workspace
CMD ["bash"]
