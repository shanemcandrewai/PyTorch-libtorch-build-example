cmake_minimum_required(VERSION 3.10)

project(build_libtorch)

set(PYTORCH_SRC_DIR ../pytorch)

include(CMakeDependentOption)
option(RESET "Restore and clean PyTorch source tree")
CMAKE_DEPENDENT_OPTION(NO_BUILD_SHARED_LIBS "Build libcaffe2.so" ON RESET OFF)
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

if(RESET)
  find_package(Git)
  if(Git_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} restore :/ WORKING_DIRECTORY ${PYTORCH_SRC_DIR})
  endif()

    file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
    string(REPLACE "include(cmake\/" "include(${PYTORCH_SRC_DIR}/cmake\/" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "file(READ version.txt" "file(READ ${PYTORCH_SRC_DIR}/version.txt" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory(c10" "add_subdirectory(${PYTORCH_SRC_DIR}/c10 ${CMAKE_BINARY_DIR}/c10" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory(caffe2" "add_subdirectory(${PYTORCH_SRC_DIR}/caffe2 ${CMAKE_BINARY_DIR}/caffe2" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")

    file(READ ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake FILE_CONTENTS)
    string(REPLACE "add_subdirectory($\{CMAKE_SOURCE_DIR}\/aten\/src\/ATen\/cpu\/tbb)" "add_subdirectory(${PYTORCH_SRC_DIR}\/aten\/src\/ATen\/cpu\/tbb ${CMAKE_BINARY_DIR}\/aten\/src\/ATen\/cpu\/tbb)" FILE_CONTENTS "${FILE_CONTENTS}")
    string(REPLACE "add_subdirectory($\{CMAKE_SOURCE_DIR}\/third_party\/fmt" "add_subdirectory(${PYTORCH_SRC_DIR}\/third_party\/fmt ${CMAKE_BINARY_DIR}\/third_party\/fmt" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/cmake/Dependencies.cmake "${FILE_CONTENTS}")

  if(NO_BUILD_SHARED_LIBS)
    message("xxx NO_BUILD_SHARED_LIBS")
    file(READ ${PYTORCH_SRC_DIR}/CMakeLists.txt FILE_CONTENTS)
    string(REPLACE "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\" ON)" "option(BUILD_SHARED_LIBS \"Build libcaffe2.so\")" FILE_CONTENTS "${FILE_CONTENTS}")
    file(WRITE ${PYTORCH_SRC_DIR}/CMakeLists.txt "${FILE_CONTENTS}")
  endif()

endif()

include(${PYTORCH_SRC_DIR}/CMakeLists.txt)
