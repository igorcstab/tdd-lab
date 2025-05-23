cmake_minimum_required(VERSION 3.14...3.22)

project(TDD-labTests LANGUAGES CXX)

# ---- Options ----

option(ENABLE_TEST_COVERAGE "Enable test coverage" OFF)
option(TEST_INSTALLED_VERSION "Test the version found by find_package" OFF)

# --- Import tools ----

include(../cmake/tools.cmake)

# ---- Dependencies ----

include(../cmake/CPM.cmake)

CPMAddPackage("gh:catchorg/Catch2@3.8.0")
CPMAddPackage("gh:TheLartians/Format.cmake@1.7.3")

if(TEST_INSTALLED_VERSION)
  find_package(TDD-lab REQUIRED)
else()
  CPMAddPackage(NAME TDD-lab SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)
endif()

# ---- Create binary ----

file(GLOB sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)
add_executable(${PROJECT_NAME} ${sources})
target_link_libraries(${PROJECT_NAME} Catch2::Catch2WithMain TDD-lab::TDD-lab)
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 23)

# enable compiler warnings
if(NOT TEST_INSTALLED_VERSION)
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(TDD-lab PUBLIC -Wall -Wpedantic -Wextra -Werror)
  elseif(MSVC)
    target_compile_options(TDD-lab PUBLIC /W4 /WX)
    target_compile_definitions(${PROJECT_NAME} PUBLIC CATCH2_CONFIG_USE_STD_HEADERS)
  endif()
endif()

# ---- Add TDD-labTests ----

enable_testing()

# Note: doctest and similar testing frameworks can automatically configure CMake tests. For other
# testing frameworks add the tests target instead: add_test(NAME ${PROJECT_NAME} COMMAND
# ${PROJECT_NAME})

include(${Catch2_SOURCE_DIR}/extras/Catch.cmake)
catch_discover_tests(${PROJECT_NAME})

# ---- code coverage ----

if(ENABLE_TEST_COVERAGE)
  target_compile_options(TDD-lab PUBLIC -O0 -g -fprofile-arcs -ftest-coverage)
  target_link_options(TDD-lab PUBLIC -fprofile-arcs -ftest-coverage)
endif()
