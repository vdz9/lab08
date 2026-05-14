
cmake_minimum_required(VERSION 3.5)

project(print)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION
  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_TESTS "Build tests" OFF)

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

add_executable(example1 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)

target_link_libraries(example1 print)
target_link_libraries(example2 print)

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print)
install(TARGETS demo RUNTIME DESTINATION bin)

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check ${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()

include(CPackConfig.cmake)
