cmake_minimum_required(VERSION 3.6)

project(proto)

cmake_policy(SET CMP0078 OLD)
cmake_policy(SET CMP0086 OLD)

# SWIG, Python
if (SWIG_BINDINGS)
  find_package(SWIG REQUIRED)
endif()
find_package(PythonInterp 3 REQUIRED)
find_package(PythonLibs 3 REQUIRED)


include(UseSWIG)
include(GNUInstallDirs)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
# Finding eigen3 path flaky in some envs; use abs path
include_directories(/usr/local/include/eigen3)

# LTSSSolver
add_executable(LTSS_solver_demo LTSS_solver_demo.cpp)
target_link_libraries(LTSS_solver_demo LTSS)

# DPSolver
add_executable(DP_solver_demo DP_solver_demo.cpp)
target_link_libraries(DP_solver_demo DP LTSS pthread)

# solver_timer
add_executable(solver_timer solver_timer.cpp)
target_link_libraries(solver_timer DP timer)

set(CMAKE_SWIG_FLAGS "")

if (SWIG_BINDINGS)
  set_source_files_properties(proto.i PROPERTIES CPLUSPLUS ON)
  swig_add_library(proto LANGUAGE python OUTPUT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../python OUTFILE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../python SOURCES LTSS.cpp python_dpsolver.cpp DP.cpp python_ltsssolver.cpp proto.i)
  swig_link_libraries(proto ${PYTHON_LIBRARIES})
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${PYTHON_INCLUDE_PATH})

add_library(timer OBJECT timer.cpp)
add_library(DP OBJECT DP.cpp)
add_library(LTSS OBJECT LTSS.cpp)

target_link_libraries(DP LTSS)
