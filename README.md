# cuml-installation-notes

This page summarizes 2 possible ways to install the GPU-accelerated ML libraries
offered by [cuML](https://github.com/rapidsai/cuml).

## Conda / Docker

Conda and Docker are at the moment the two officially supported release channels
for `cuML`. When installing `cuML` via Docker, RAPIDS components will be
installed from published Conda packages to the 'rapids' Conda environment (e.g.,
see
[here](https://github.com/rapidsai/docker/blob/branch-21.08/generated-dockerfiles/rapidsai-core_centos8-base.Dockerfile#L42-L43)).

## Build from source without `Conda` and without multi-GPU support

- See [this section](https://github.com/rapidsai/cuml/tree/branch-21.08/cpp#setup)
  for system dependencies of `cuML` libraries.

- Follow [this page](https://developer.nvidia.com/cuda-downloads) to install CUDA
  Toolkit 11.0+.

- Follow [this link](https://developer.nvidia.com/nccl/nccl-download) to install
  `libnccl`.

- Follow [this link](https://github.com/openucx/ucx/releases/tag/v1.10.1) to
  install `ucx`.

- And then run the following:

```
wget https://github.com/rapidsai/cuml/archive/refs/tags/v21.06.02.tar.gz
tar xvf v21.06.02.tar.gz
cd cuml-21.06.02/cpp
mkdir build
cd build
cmake .. -DENABLE_CUMLPRIMS_MG=OFF -DBUILD_CUML_TESTS=OFF -DBUILD_CUML_BENCH=OFF
make -j $(nproc)
```

to build `cuML` libraries from source.

NOTE: this is **not** an officially supported release channel (yet). Your
mileage may vary. Also, the build could take quite a while, even on an
8-core machine with modern CPUs.
