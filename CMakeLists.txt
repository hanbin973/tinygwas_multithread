cmake_minimum_required(VERSION 3.12)
project(tinygwas)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_STANDARD 14)

include_directories(${CMAKE_SOURCE_DIR}/thirdparty/)
set(SOURCE_FILES src/FileUtils.cpp src/FileUtils.h src/Utils.cpp src/Utils.h)

add_subdirectory(thirdparty/pybind11)
pybind11_add_module(tinygwas src/main.cpp ${SOURCE_FILES})

# BLAS
find_package(BLAS REQUIRED)
if(BLAS_FOUND)
    message("BLAS found.")
    include_directories(${BLAS_INCLUDE_DIRS})
    target_link_libraries(tinygwas PUBLIC ${BLAS_LIBRARIES})
    set(CMAKE_CXX_FLAGS "-D EIGEN_USE_BLAS=1")
endif(BLAS_FOUND)

# LAPACK
find_package(LAPACK REQUIRED)

# LAPACKE
list(APPEND CMAKE_MODULE_PATH "")
include(FindLAPACKE.cmake)
find_package(LAPACKE QUIET)
if(LAPACKE_FOUND)
    message("LAPACKE found.")
    include_directories(${LAPACK_INCLUDE_DIRS})
    target_link_libraries(tinygwas PUBLIC ${LAPACK_LIBRARIES})
    set(CMAKE_CXX_FLAGS "-D EIGEN_USE_LAPACKE=1")
endif(LAPACKE_FOUND)
