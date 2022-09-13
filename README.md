# tinygwas
![python package](https://github.com/bogdanlab/tinygwas/actions/workflows/python.yml/badge.svg)

## Install python version
```bash
pip install -e .
```
## Enable multithreading

### Background
`tinygwas` uses the [`Eigen`](https://eigen.tuxfamily.org/index.php?title=Main_Page) C++ library for linear algebra.
Therefore enabling multithreading in `tinygwas` boils down to enabling it in `Eigen`.

Eigen supports multithreading through several options as outlined in their [wiki](https://eigen.tuxfamily.org/dox/TopicMultiThreading.html).
The two options are
    1. Enabling `openmp` with the `-fopenmp` flag.
    2. Linking `BLAS\LAPACK` backend to replace the linear algebra routines of Eigen.
Note that while the second option enables multithreading across all operations that is supported by the system's `BLAS\LAPACK` routine,
the first option only enables multithreading partially as described in their wiki (the link above).

### TL;DR
As long as the system has `BLAS\LAPACK` installed, just run the installation command 
```bash
pip install e .
```
which has no change from the original installation procedure.
Note that when using Intel MKL, it is necessary to run `python` with `LD_PRELOAD` specified to the `libmkl_rt.so`.
```bash
LD_PRELOAD="/path/to/mkl/libmkl_rt.so"
```

In `conda` environment, it is usually found in the `anaconda3/envs/env_name/lib/` folder.
I'm currently under a workaround to resolve this problem.

### Details
The current version implements the second option above by linking the system's `BLAS\LAPACK` shared libraries to the `tinygwas` binary during compilation.
This can be done by modifying the `CMakeLists.txt` file with an additional `LAPACKE` linking extension (the `FindLAPACKE.cmake` file).

There are two major code blocks added to implement linking shared libraries.
```
    # BLAS
    find_package(BLAS REQUIRED)
    if(BLAS_FOUND)
        message("BLAS found.")
        include_directories(${BLAS_INCLUDE_DIRS})
        target_link_libraries(tinygwas PUBLIC ${BLAS_LIBRARIES})
        set(CMAKE_CXX_FLAGS "-D EIGEN_USE_BLAS=1")
    endif(BLAS_FOUND)
```
detects the system's `BLAS` and links it to `Eigen`.

```
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
```
detects the system's `LAPACK` and links it ot `Eigen`.
Note that `LAPACKE` is a C-wrapper around `LAPACK` which is originally written in Fortran.

## Contact
Kangcheng Hou (kangchenghou@gmail.com, The original author)

Hanbin Lee (hanbin973@snu.ac.kr, Maintainer of the current repo)
