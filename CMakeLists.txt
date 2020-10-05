cmake_minimum_required(VERSION 3.10)

# set the project name
project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(BUILD_SHARED_LIBS "Build libtorch.so" ON)
option(RESET "Restore and clean PyTorch source tree")
option(DEBUG "will enable debug builds (-g -O0)" ON)
option(REL_WITH_DEB_INFO "will enable debug symbols with optimizations (-g -O3)" ON)
option(USE_DISTRIBUTED "will disable distributed (c10d, gloo, mpi, etc.) build.")
option(USE_MKLDNN "will disable using MKL-DNN.")
option(USE_CUDA "will disable compiling CUDA (in case you are developing on something not CUDA related), to save compile time.")
option(BUILD_TEST "will disable building C++ test binaries.")
option(USE_FBGEMM "will disable using FBGEMM (quantized 8-bit server operators).")
option(USE_NNPACK "will disable compiling with NNPACK.")
option(USE_QNNPACK "will disable QNNPACK build (quantized 8-bit operators).")
option(USE_XNNPACK "will disable compiling with XNNPACK.")

option(BUILD_CAFFE2_OPS "disable build of Caffe2 operators")


if(RESET)
  find_package(Git)
  # Restore PyTorch working tree from HEAD and cleans recursively without respecting .gitignore
  if(Git_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} restore :/ WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
    execute_process(COMMAND ${GIT_EXECUTABLE} clean -dfx WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
  endif()
endif()

find_package(PythonInterp REQUIRED)
execute_process(COMMAND ${PYTHON_EXECUTABLE} ${PYTORCH_SRC_DIR}/tools/build_libtorch.py)
