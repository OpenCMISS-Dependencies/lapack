cmake_minimum_required(VERSION 3.0)
PROJECT(BLAS VERSION 3.5.0 LANGUAGES Fortran)

add_subdirectory(SRC)
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(TESTING)
endif(BUILD_TESTS)