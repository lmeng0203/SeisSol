SYCL
====

.. _installing_SYCL:

SeisSol utilizes both native GPU and SYCL programming models. The former one is mainly used in performance critical
parts of the application which typically contain some generated code. The latter one is used in places where we need to
achieve GPU source code portability (e.g., Dynamic Rupture).

SYCL is a standard for cross-platform programming of heterogeneous processors. Below are the instructions for
installing the *hipSYCL* implementation of the SYCL standard. If you want to use DPC++ then refer to
`the followig instructions <https://intel.github.io/llvm-docs/GetStartedGuide.html#create-dpc-workspace>`_.


Installing LLVM
---------------

A preparation step:

.. code-block:: bash

  wget https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-12.0.0.tar.gz
  tar -xvf ./llvmorg-12.0.0.tar.gz
  mkdir -p llvm-project-llvmorg-12.0.0/build && cd llvm-project-llvmorg-12.0.0/build

LLVM comes with many backends. The most common CPU ones are X86, ARM, PowerPC. Below are the instructions for
installing LLVM with X86 and NVPTX backends for CPUs and GPUs, respectively.

.. code-block:: bash

  GCC_ROOT=$(dirname $(dirname $(which gcc)))
  CUDA_ROOT=${CUDA_PATH} # or CUDA_ROOT=$(dirname $(dirname $(which nvcc)))
  cmake ../llvm -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;openmp;libunwind;polly" \
  -DGCC_INSTALL_PREFIX="${GCC_ROOT}" \
  -DCUDA_TOOLKIT_ROOT_DIR="${CUDA_ROOT}" \
  -DCMAKE_INSTRALL_PREFIX="${HOME}"" \
  -DLLVM_TARGETS_TO_BUILD="X86;NVPTX"
  make -j $(nproc)
  make install
  cd ../..

Follow the instructions listed below if you need to configure LLVM with AMD GPU backend.

.. code-block:: bash

  GCC_ROOT=$(dirname $(dirname $(which gcc)))
  CUDA_ROOT=${CUDA_PATH} # or CUDA_ROOT=$(dirname $(dirname $(which nvcc)))
  cmake ../llvm -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;openmp;polly" \
  -DGCC_INSTALL_PREFIX="${GCC_ROOT}" \
  -DCMAKE_INSTRALL_PREFIX="${HOME}"" \
  -DLLVM_TARGETS_TO_BUILD="X86;AMDGPU"

  make -j $(nproc)
  make install

  cd ../..


Installing Boost
----------------

hipSYCL depends on 1.69.0 version of the Boost library.

.. code-block:: bash

  wget https://boostorg.jfrog.io/artifactory/main/release/1.69.0/source/boost_1_69_0.tar.gz
  tar -xvf ./boost_1_69_0.tar.gz
  cd boost_1_69_0

  GCC_EXEC=$(which g++)
  ./bootstrap.sh --prefix="${HOME}" \
  --with-toolset=gcc \
  --with-libraries=serialization,wave,date_time,iostreams,locale,math,random,context,regex,program_options,atomic,timer,log,fiber,chrono,thread,exception,system,test,graph,filesystem

  echo "using gcc : : ${GCC_EXEC} ;" > user-config.jam

  ./b2 install threading=multi variant=release toolset=gcc link=shared \
  cxxflags="-std=c++17" visibility=hidden -j $(nproc) --user-config="user-config.jam"

  cd ..


Installing hipSYCL
------------------

SeisSol requires 0.9.3 version of hipSYCL for a correct multi GPU-setup.

.. code-block:: bash

  git clone --depth 1 --branch v0.9.3 https://github.com/illuhad/hipSYCL.git
  cd hipSYCL

Perform the following steps to configure and install hipSYCL for Nvidia GPUs

.. code-block:: bash

  export CUDA_PATH=$CUDA_HOME
  CLANG_DIR=$(dirname $(dirname $(which clang)))
  CLANG_EXE=$(which clang++)

  CC=gcc CXX=g++ cmake .. \
  -DCMAKE_BUILD_TYPE:STRING=Release \
  -DCMAKE_INSTALL_PREFIX="${HOME}" \
  -DWITH_CPU_BACKEND:Bool=TRUE \
  -DWITH_ROCM_BACKEND:Bool=FALSE \
  -DWITH_CUDA_BACKEND:Bool=TRUE \
  -DWITH_ACCELERATED_CPU=OFF \
  -DLLVM_DIR:String="${CLANG_DIR}/lib/cmake/llvm" \
  -DCLANG_INCLUDE_PATH:String="${CLANG_DIR}/lib/clang/12.0.0/include" \
  -DCLANG_EXECUTABLE_PATH:String="${CLANG_EXE}" \
  -DCUDA_TOOLKIT_ROOT_DIR:String="${CUDA_PATH}" \
  -DBoost_NO_BOOST_CMAKE=TRUE \
  -DBoost_NO_SYSTEM_PATHS=TRUE \
  -DBOOST_ROOT:PATHNAME="${HOME}" \
  -DBoost_LIBRARY_DIRS:FILEPATH="${HOME}/lib"

  make -j $(nproc)
  make install

  cd ../..


The following steps describe the steps to configure and install hipSYCL for AMD GPUs. Note `ROCM_PATH` is typically
set by system administrators. Please, makes sure that this environment variable is not empty.

.. code-block:: bash

  CLANG_DIR=$(dirname $(dirname $(which clang)))
  CLANG_EXEC=$(which clang++)

  cmake .. -DCMAKE_BUILD_TYPE:STRING=Release \
  -DCMAKE_INSTALL_PREFIX="${HOME}" \
  -DWITH_CPU_BACKEND:Bool=TRUE \
  -DWITH_ROCM_BACKEND:Bool=TRUE \
  -DWITH_CUDA_BACKEND:Bool=FALSE \
  -DWITH_ACCELERATED_CPU=OFF \
  -DLLVM_DIR:String="${CLANG_DIR}/lib/cmake/llvm" \
  -DCLANG_INCLUDE_PATH:String="${CLANG_DIR}/lib/clang/12.0.0/include" \
  -DCLANG_EXECUTABLE_PATH:String="${CLANG_EXEC}" \
  -DROCM_PATH:String="${ROCM_PATH}" \
  -DBoost_NO_BOOST_CMAKE=TRUE \
  -DBoost_NO_SYSTEM_PATHS=TRUE \
  -DBOOST_ROOT:PATHNAME="${HOME}" \
  -DBoost_LIBRARY_DIRS:FILEPATH="${HOME}"/lib

  make -j $(nproc)
  make install

  cd ../..

Add the following durint the CMake configuration step if you want to enable the OpenMP backend of SYCL device kernels:
`-DWITH_ACCELERATED_CPU=ON`


