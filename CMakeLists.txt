cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)

if(${CMAKE_VERSION} VERSION_LESS "3.8.0")
    project(sudoku)
else()
    project(sudoku LANGUAGES CXX CUDA)
    # https://stackoverflow.com/questions/53859271/cmake-cannot-find-cuda-could-not-find-cmake-module-file-cmakedeterminecudacom
endif()

find_package(TensorFlow REQUIRED)
find_package(CUDA 10 REQUIRED)
find_package(Boost)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${SSE_FLAGS} -march=native -fopenmp -D_GLIBCXX_USE_CXX11_ABI=${TensorFlow_ABI}")
set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -fPIC --shared -D_GLIBCXX_USE_CXX11_ABI=${TensorFlow_ABI}" )
# ref: http://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/
# Workaround: error: constexpr function return is non-constant https://github.com/tensorflow/tensorflow/issues/22766
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11 --expt-relaxed-constexpr
    -DGOOGLE_CUDA=1 -DNDEBUG=1
    -gencode arch=compute_60,code=sm_60
    -gencode arch=compute_61,code=sm_61
    -gencode arch=compute_61,code=compute_61
    -gencode arch=compute_70,code=sm_70
    -D_GLIBCXX_USE_CXX11_ABI=${TensorFlow_ABI}" )

get_filename_component(TF_CUDA ${CUDA_INCLUDE_DIRS} DIRECTORY)
get_filename_component(TF_CUDA ${TF_CUDA} DIRECTORY)
include_directories(SYSTEM ${TF_CUDA})
include_directories(SYSTEM "${TensorFlow_INCLUDE_DIR}")
include_directories(SYSTEM "includes")

# add_tensorflow_gpu_operation("sudoku")

# Tests
if(Boost_FOUND)
    set(CMAKE_EXE_LINKER_FLAGS  "-D_GLIBCXX_USE_CXX11_ABI=${TensorFlow_ABI}" )
    add_executable(test_sudoku "tests/test_sudoku.cc")
    target_link_libraries(test_sudoku LINK_PUBLIC sudoku_op)
endif()
