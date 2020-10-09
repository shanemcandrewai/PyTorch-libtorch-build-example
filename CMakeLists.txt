cmake_minimum_required(VERSION 3.10)

# set the project name
project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

option(RESET "Restore and clean PyTorch source tree")
option(NO_BUILD_SHARED_LIBS "Build libcaffe2.so" ON)
option(DEBUG "enable debug builds" ON)
option(USE_CUDA "disable compiling CUDA, to save compile time")
option(USE_DISTRIBUTED "disable distributed (c10d, gloo, mpi, etc.) build")
option(USE_MKLDNN "disable using MKL-DNN")
option(USE_FBGEMM "disable using FBGEMM (quantized 8-bit server operators)")
option(USE_NNPACK "disable compiling with NNPACK")
option(USE_QNNPACK "disable QNNPACK build (quantized 8-bit operators)")
option(USE_XNNPACK "disable compiling with XNNPACK")
option(BUILD_CUSTOM_PROTOBUF "Build and use Caffe2's own protobuf under third_party")
option(BUILD_PYTHON "Build Python binaries")
option(BUILD_CAFFE2_OPS "Build Caffe2 operators")

if(DEBUG)
  set(ENV{DEBUG} 1) #enable debug builds (-g -O0)
#set(ENV{REL_WITH_DEB_INFO} 1) #enable debug symbols with optimizations (-g -O3)
endif()

if(RESET)
  find_package(Git)
  # Restore PyTorch working tree from HEAD and cleans recursively without respecting .gitignore
  if(Git_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} restore :/ WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
#    execute_process(COMMAND ${GIT_EXECUTABLE} clean -dfx WORKING_DIRECTORY ${PYTORCH_SRC_DIR})

    file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
    string(REPLACE "include(cmake\/" "include(${PYTORCH_SRC_DIR}/cmake\/" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "file(READ version.txt" "file(READ ${PYTORCH_SRC_DIR}/version.txt" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "set(CAFFE2_CMAKE_BUILDING_WITH_MAIN_REPO ON)" "set(CAFFE2_CMAKE_BUILDING_WITH_MAIN_REPO)" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory(c10" "add_subdirectory(${PYTORCH_SRC_DIR}/c10 ${CMAKE_BINARY_DIR}/c10" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory(caffe2" "#add_subdirectory(${PYTORCH_SRC_DIR}/caffe2 ${CMAKE_BINARY_DIR}/caffe2" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")

    file(READ ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake FILE_CONTENTS)
    string(REPLACE "add_subdirectory($\{CMAKE_SOURCE_DIR}\/aten\/src\/ATen\/cpu\/tbb)" "add_subdirectory(${PYTORCH_SRC_DIR}\/aten\/src\/ATen\/cpu\/tbb ${CMAKE_BINARY_DIR}\/aten\/src\/ATen\/cpu\/tbb)" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory($\{CMAKE_SOURCE_DIR}\/third_party\/fmt" "add_subdirectory(${PYTORCH_SRC_DIR}\/third_party\/fmt ${CMAKE_BINARY_DIR}\/third_party\/fmt" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake "${FILE_CONTENTS}")
  endif()
  set(RESET OFF CACHE BOOL "Restore and clean PyTorch source tree" FORCE)
endif()

if(NO_BUILD_SHARED_LIBS)
  file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
  string(REPLACE "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\" ON)" "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\")" FILE_CONTENTS "${FILE_CONTENTS}")
  file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
  set(NO_BUILD_SHARED_LIBS OFF CACHE BOOL "Build libcaffe2.so" FORCE)
endif()

#find_package(Python REQUIRED)
#execute_process(COMMAND ${Python_EXECUTABLE} ${PYTORCH_SRC_DIR}/tools/build_libtorch.py)
include(${PYTORCH_SRC_DIR}/CMakeLists.txt)
