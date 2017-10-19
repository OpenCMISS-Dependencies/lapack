cmake_minimum_required(VERSION 3.0)
PROJECT(LAPACK VERSION 3.5.0 LANGUAGES Fortran C)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
set(PACKAGE_CONFIG_DIR "lib/cmake" CACHE STRING "Directory for package config files (relative to CMAKE_INSTALL_PREFIX)")
set(BUILD_PRECISION sdcz CACHE STRING "${PROJECT_NAME} - Build precisions (sdcz)")

# Always build all precisions,
# as the different modules have test routines
# wit suitesparse library has demo-code for both that is hard to separate
#set(BUILD_PRECISION "${BUILD_PRECISION}dz")
set(BUILD_PRECISION sdcz)


# Configure the warning and code coverage suppression file
configure_file( 
  "${LAPACK_SOURCE_DIR}/CTestCustom.cmake.in"
  "${LAPACK_BINARY_DIR}/CTestCustom.cmake"
  COPYONLY
)

# Add the CMake directory for custon CMake modules
set(CMAKE_MODULE_PATH "${LAPACK_SOURCE_DIR}/CMAKE" ${CMAKE_MODULE_PATH})

if (UNIX)
   if ( "${CMAKE_Fortran_COMPILER}" MATCHES "ifort" )
  set( CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -fltconsistency -fp_port" )
   endif ()
   if ( "${CMAKE_Fortran_COMPILER}" MATCHES "xlf" )
  set( CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -qnosave -qstrict=none" )
   endif ()
# Delete libmtsk in linking sequence for Sun/Oracle Fortran Compiler.
# This library is not present in the Sun package SolarisStudio12.3-linux-x86-bin
   STRING(REPLACE \;mtsk\; \; CMAKE_Fortran_IMPLICIT_LINK_LIBRARIES "${CMAKE_Fortran_IMPLICIT_LINK_LIBRARIES}")
endif ()

# Get Python
#find_package(PythonInterp)
#message(STATUS "Looking for Python found - ${PYTHONINTERP_FOUND}")
#if (PYTHONINTERP_FOUND)
#   message(STATUS "Using Python version ${PYTHON_VERSION_STRING}")
#endif()
# --------------------------------------------------

# --------------------------------------------------
# Check for any necessary platform specific compiler flags
include( CheckLAPACKCompilerFlags )
CheckLAPACKCompilerFlags()

# Old way to detect mangling
#include(FortranMangling)
#FORTRAN_MANGLING(CDEFS)
#set(CDEFS ${CDEFS} CACHE STRING "Fortran Mangling" FORCE)
#MESSAGE(STATUS "Fortran Mangling: -D${CDEFS}")

# --------------------------------------------------
# Check second function

include(CheckTimeFunction)
set(TIME_FUNC NONE ${TIME_FUNC})
CHECK_TIME_FUNCTION(NONE TIME_FUNC)
CHECK_TIME_FUNCTION(INT_CPU_TIME TIME_FUNC)
CHECK_TIME_FUNCTION(EXT_ETIME TIME_FUNC)
CHECK_TIME_FUNCTION(EXT_ETIME_ TIME_FUNC)
CHECK_TIME_FUNCTION(INT_ETIME TIME_FUNC)
message(STATUS "--> Will use second_${TIME_FUNC}.f and dsecnd_${TIME_FUNC}.f as timing function.")

set(SECOND_SRC  ${LAPACK_SOURCE_DIR}/INSTALL/second_${TIME_FUNC}.f)
set(DSECOND_SRC  ${LAPACK_SOURCE_DIR}/INSTALL/dsecnd_${TIME_FUNC}.f)
set(prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX})
set(PKG_CONFIG_DIR ${libdir}/pkgconfig)

# --------------------------------------------------
# Subdirectories that need to be processed

#option(USE_OPTIMIZED_BLAS "Whether or not to use an optimized BLAS library instead of included netlib BLAS" OFF)


# Check the usage of the user provided BLAS libraries
#if(BLAS_LIBRARIES)
#  include(CheckFortranFunctionExists)
#  set(CMAKE_REQUIRED_LIBRARIES ${BLAS_LIBRARIES})
#  CHECK_FORTRAN_FUNCTION_EXISTS("dgemm" BLAS_FOUND)
#  unset( CMAKE_REQUIRED_LIBRARIES )
#  if(BLAS_FOUND)
#    message(STATUS "--> BLAS supplied by user is WORKING, will use ${BLAS_LIBRARIES}.")
#  else(BLAS_FOUND)
#    message(ERROR "--> BLAS supplied by user is not WORKING, CANNOT USE ${BLAS_LIBRARIES}.")
#    message(ERROR "-->     Will use REFERENCE BLAS (by default)")
#    message(ERROR "-->     Or Correct your BLAS_LIBRARIES entry ")
#    message(ERROR "-->     Or Consider checking USE_OPTIMIZED_BLAS")
#  endif(BLAS_FOUND)

# User did not provide a BLAS Library but specified to search for one
#elseif( USE_OPTIMIZED_BLAS )
#  find_package( BLAS )
#endif (BLAS_LIBRARIES)

# Neither user specified or optimized BLAS libraries can be used
#if(NOT BLAS_FOUND)
#  message(STATUS "Using supplied NETLIB BLAS implementation")
  add_subdirectory(BLAS)
  set( BLAS_LIBRARIES blas )
#else()
#  set( CMAKE_EXE_LINKER_FLAGS 
#    "${CMAKE_EXE_LINKER_FLAGS} ${BLAS_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for executables" FORCE)
#  set( CMAKE_MODULE_LINKER_FLAGS 
#    "${CMAKE_MODULE_LINKER_FLAGS} ${BLAS_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for modules" FORCE)
#  set( CMAKE_SHARED_LINKER_FLAGS 
#    "${CMAKE_SHARED_LINKER_FLAGS} ${BLAS_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for shared libs" FORCE)
#endif( NOT BLAS_FOUND )


# --------------------------------------------------
# CBLAS
#option(CBLAS "Build CBLAS" OFF)

#if(CBLAS)
#  add_subdirectory(CBLAS)
#endif(CBLAS)

# --------------------------------------------------
# XBLAS

#option(USE_XBLAS "Build extended precision (needs XBLAS)" OFF)
#if (USE_XBLAS)
#  find_library(XBLAS_LIBRARY NAMES xblas)
#endif(USE_XBLAS)
   
#option(USE_OPTIMIZED_LAPACK "Whether or not to use an optimized LAPACK library instead of included netlib LAPACK" OFF)

# --------------------------------------------------
# LAPACK
# User did not provide a LAPACK Library but specified to search for one
#if( USE_OPTIMIZED_LAPACK )
#  find_package( LAPACK )
#endif (USE_OPTIMIZED_LAPACK)

# Check the usage of the user provided or automatically found LAPACK libraries
#if(LAPACK_LIBRARIES)
#  include(CheckFortranFunctionExists)
#  set(CMAKE_REQUIRED_LIBRARIES ${LAPACK_LIBRARIES})
  # Check if new routine of 3.4.0 is in LAPACK_LIBRARIES
#  CHECK_FORTRAN_FUNCTION_EXISTS("dgeqrt" LATESTLAPACK_FOUND)
#  unset( CMAKE_REQUIRED_LIBRARIES )
#  if(LATESTLAPACK_FOUND)
#    message(STATUS "--> LAPACK supplied by user is WORKING, will use ${LAPACK_LIBRARIES}.")
#  else(LAPACK_FOUND)
#    message(ERROR "--> LAPACK supplied by user is not WORKING or is older than LAPACK 3.4.0, 	CANNOT USE ${LAPACK_LIBRARIES}.")
#    message(ERROR "-->     Will use REFERENCE LAPACK (by default)")
#    message(ERROR "-->     Or Correct your LAPACK_LIBRARIES entry ")
#    message(ERROR "-->     Or Consider checking USE_OPTIMIZED_LAPACK")
#  endif(LATESTLAPACK_FOUND)
#endif (LAPACK_LIBRARIES)

# Neither user specified or optimized LAPACK libraries can be used
#if(NOT LATESTLAPACK_FOUND)
#  message(STATUS "Using supplied NETLIB LAPACK implementation")
  set( LAPACK_LIBRARIES lapack )
  add_subdirectory(SRC)
#else()
#  set( CMAKE_EXE_LINKER_FLAGS 
#    "${CMAKE_EXE_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for executables" FORCE)
#  set( CMAKE_MODULE_LINKER_FLAGS 
#    "${CMAKE_MODULE_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for modules" FORCE)
#  set( CMAKE_SHARED_LINKER_FLAGS 
#    "${CMAKE_SHARED_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS}" 
#    CACHE STRING "Linker flags for shared libs" FORCE)
#endif( NOT LATESTLAPACK_FOUND )

if(BUILD_TESTS)
  enable_testing()
  include(CTest)
  add_subdirectory(TESTING)
endif(BUILD_TESTS)

# --------------------------------------------------
# LAPACKE
#option(LAPACKE "Build LAPACKE" OFF)

# LAPACKE has also the interface to some routines from tmglib,
# if LAPACKE_WITH_TMG is selected, we need to add those routines to LAPACKE
#option(LAPACKE_WITH_TMG "Build LAPACKE with tmglib routines" OFF)
#if (LAPACKE_WITH_TMG)
#  set(LAPACKE ON)
#  if(NOT BUILD_TESTS)
#     add_subdirectory(TESTING/MATGEN)
#  endif(NOT BUILD_TESTS)
#endif(LAPACKE_WITH_TMG)

#if(LAPACKE)
#  add_subdirectory(LAPACKE)
#endif(LAPACKE)

#if(NOT BLAS_FOUND)
#  set(ALL_TARGETS ${ALL_TARGETS} blas)
#endif(NOT BLAS_FOUND)

#if(NOT LATESTLAPACK_FOUND)
#  set(ALL_TARGETS ${ALL_TARGETS} lapack)
#endif(NOT LATESTLAPACK_FOUND)

#if(BUILD_TESTS OR LAPACKE_WITH_TMG)
#  set(ALL_TARGETS ${ALL_TARGETS} tmglib)
#endif(BUILD_TESTS OR LAPACKE_WITH_TMG)

# Export lapack targets, not including lapacke, from the
# install tree, if any.
#set(_lapack_config_install_guard_target "")
#if(ALL_TARGETS)
#  install(EXPORT lapack-targets
#    DESTINATION ${PACKAGE_CONFIG_DIR}/lapack-${LAPACK_VERSION})

  # Choose one of the lapack targets to use as a guard for
  # lapack-config.cmake to load targets from the install tree.
#  list(GET ALL_TARGETS 0 _lapack_config_install_guard_target)
#endif()
