cmake_minimum_required(VERSION 3.10)

# set the project name
project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include(CMakeDependentOption)
option(RESET "Restore and clean PyTorch source tree")
CMAKE_DEPENDENT_OPTION(NO_BUILD_CAFFE2 "Master flag to build Caffe2" ON "RESET" OFF)
CMAKE_DEPENDENT_OPTION(NO_BUILD_SHARED_LIBS "Build libcaffe2.so" ON "RESET" OFF)
option(DEBUG "enable debug builds" ON)

set(ENV{USE_DISTRIBUTED} 0) #disable distributed (c10d, gloo, mpi, etc.) build.
set(ENV{USE_MKLDNN} 0) #disable using MKL-DNN.
set(ENV{USE_CUDA} 0) #disable compiling CUDA, to save compile time)
set(ENV{BUILD_TEST} 0) #disable building C++ test binaries.
set(ENV{USE_FBGEMM} 0) #disable using FBGEMM (quantized 8-bit server operators).
set(ENV{USE_NNPACK} 0) #disable compiling with NNPACK.
set(ENV{USE_QNNPACK} 0) #disable QNNPACK build (quantized 8-bit operators).
set(ENV{USE_XNNPACK} 0) #disable compiling with XNNPACK.

if(DEBUG)
  set(ENV{DEBUG} 1) #enable debug builds (-g -O0)
#set(ENV{REL_WITH_DEB_INFO} 1) #enable debug symbols with optimizations (-g -O3)
endif()

if(RESET)
  find_package(Git)
  # Restore PyTorch working tree from HEAD and cleans recursively without respecting .gitignore
  if(Git_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} restore :/ WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
    execute_process(COMMAND ${GIT_EXECUTABLE} clean -dfx WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
  endif()
  set(RESET OFF CACHE BOOL "Restore and clean PyTorch source tree" FORCE)
endif()

if(NO_BUILD_CAFFE2)
  file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
  string(REPLACE "option(BUILD_CAFFE2 \"Master flag to build Caffe2\" ON)" "option(BUILD_CAFFE2 \"Master flag to build Caffe2\")" FILE_CONTENTS "${FILE_CONTENTS}")
  file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
endif()

if(NO_BUILD_SHARED_LIBS)
  file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
  string(REPLACE "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\" ON)" "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\")" FILE_CONTENTS "${FILE_CONTENTS}")
  file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
endif()

find_package(Python REQUIRED)
execute_process(COMMAND ${Python_EXECUTABLE} ${PYTORCH_SRC_DIR}/tools/build_libtorch.py)
