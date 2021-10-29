# cuml-installation-notes

This page summarizes 3 possible ways to install the GPU-accelerated ML libraries
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
cd cpp
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

### **EDIT**
It looks like for `cuML` 21.10 or above, the `spdlog` header files
(which are required by `rmm` and possibly other `cuML` dependencies) will not be
installed by default when you run `make install`. As a temp workaround for this,
you can do the following:
- Go to https://github.com/rapidsai/rapids-cmake, switch to
`branch-<the cuML version you want to install>`.
- See the required `spdlog` version in the `rapids-cmake/cpm/versions.json`
config file (e.g., something like this:
https://github.com/rapidsai/rapids-cmake/blob/e7a79a669f3ccec524f99c50f518c0879c33f19a/rapids-cmake/cpm/versions.json#L21).
- Build and install that required version
of `spdlog` from source using the commit tagged `v<required spdlog version>` from
the https://github.com/gabime/spdlog.git repo.

## Build from source without `Conda` and with multi-GPU support

Now this is even further away from the beaten paths and appears somewhat
brittle, but at least it has worked for the `branch-21.08` branch of the `cuML`
repo.

To build `cuML` successfully with multi-GPU support, one will need a library
named `libcumlprims` ("prims" meaning primitives). Because the author of this
note was unable to find any source code for `libcumlprims` on the internet, the
only viable alternative so far seems to be downloading and installing a pre-
built copy of `libcumlprims` instead of building it from source. So,

```
# NOTE: you should choose the most recent version of libcumlprims that matches
# the version of CUDA you intend to use. Looks like currently only x86-64 is
# supported.
#
# Also, the `| sudo tar -xjvf - -C /usr` part is probably not something you
# should run directly in production!!
wget -O - https://anaconda.org/nvidia/libcumlprims/21.06.00/download/linux-64/libcumlprims-21.06.00-cuda11.2_gfda2e6c_0.tar.bz2 | sudo tar -xjvf - -C /usr
```

and then, similar build instructions as above except for removal of the
`-DSINGLEGPU=ON` flag:

```
git clone https://github.com/rapidsai/cuml.git
cd cuml
# git checkout branch-21.08
mkdir build
cd build
cmake \
  -DBUILD_CUML_TESTS=OFF \
  -DBUILD_PRIMS_TESTS=OFF \
  -DBUILD_CUML_EXAMPLES=OFF \
  -DBUILD_CUML_BENCH=OFF \
  -DBUILD_CUML_PRIMS_BENCH=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr \
  ..
make -j $(nproc)
```

Also, see [comment](#edit) above re: how to intall `spdlog` header files that
are possibly missing.
