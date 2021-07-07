# cuml-installation-notes

This page summarizes 2 possible ways to install the GPU-accelerated ML libraries
offered by [cuML](https://github.com/rapidsai/cuml).

## Conda / Docker

See [https://rapids.ai/start.html#get-rapids](https://rapids.ai/start.html#get-rapids).

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
git clone https://github.com/rapidsai/cuml.git
cd cuml
# git checkout branch-21.08
mkdir build
cd build
cmake \
  -DSINGLEGPU=ON \
  -DBUILD_CUML_TESTS=OFF \
  -DBUILD_PRIMS_TESTS=OFF \
  -DBUILD_CUML_EXAMPLES=OFF \
  -DBUILD_CUML_BENCH=OFF \
  -DBUILD_CUML_PRIMS_BENCH=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr \
  ..
make -j $(nproc)
```

to build `cuML` libraries from source.

NOTE: even though in principle this *should* work, at least on a release branch
of the `cuml` repo, this, unlike Conda/Docker, is **not** an officially
supported release channel of `cuml` (yet) AFAIK. So, your mileage may vary.
Also, the build could take a non-negligible amount of time, even on an 8-core
machine with modern CPUs.
