
cmake_minimum_required(VERSION 3.5)
project(formatter_ex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter_ex STATIC formatter_ex.cpp)
include_directories(${CMAKE_SOURCE_DIR}/formatter_lib)
target_link_libraries(formatter_ex formatter)
