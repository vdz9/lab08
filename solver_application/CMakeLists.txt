
cmake_minimum_required(VERSION 3.5)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(solver equation.cpp)
include_directories(${CMAKE_SOURCE_DIR}/formatter_lib)
include_directories(${CMAKE_SOURCE_DIR}/formatter_ex_lib)
include_directories(${CMAKE_SOURCE_DIR}/solver_lib)
target_link_libraries(solver formatter_ex solver_lib formatter)
