cmake_minimum_required(VERSION 3.5)

option(ENABLE_TESTING "Build SQLite Query Builder tests" OFF)
option(ENABLE_SQLCIPHER_TESTS "enable sqlchipher test")

# Creates the file compile_commands.json in the build directory.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 17)

if (ENABLE_TESTING)
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")
  include("cmake/Catch.cmake")
endif()

project(SqliteModernCpp)

find_package(SQLite3 REQUIRED)

if (ENABLE_TESTING)
  find_package(Catch2 REQUIRED)
  set(TEST_SOURCE_DIR             ${CMAKE_CURRENT_SOURCE_DIR}/tests)
  file(GLOB TEST_SOURCES          ${TEST_SOURCE_DIR}/*.cc)

  IF(NOT ENABLE_SQLCIPHER_TESTS)
    list(REMOVE_ITEM TEST_SOURCES ${TEST_SOURCE_DIR}/sqlcipher.cc)
  ENDIF(NOT ENABLE_SQLCIPHER_TESTS)

  enable_testing()
endif()

add_library (sqlite_modern_cpp INTERFACE)
target_include_directories(sqlite_modern_cpp INTERFACE hdr/)

if (ENABLE_TESTING)
  add_executable(tests_runner ${TEST_SOURCES})
  target_include_directories(tests_runner INTERFACE ${SQLITE3_INCLUDE_DIRS})
  if(ENABLE_SQLCIPHER_TESTS)
      target_link_libraries(tests_runner Catch2::Catch2 sqlite_modern_cpp SQLite::SQLite3 -lsqlcipher)
  else()
      target_link_libraries(tests_runner Catch2::Catch2 sqlite_modern_cpp SQLite::SQLite3)
  endif()

  catch_discover_tests(tests_runner)
  target_compile_options(tests_runner PUBLIC $<$<CXX_COMPILER_ID:MSVC>:/Zc:__cplusplus> )

  # Place the file in the source directory, permitting us to place a single configuration file for YCM there.
  # YCM is the code-completion engine for (neo)vim https://github.com/Valloric/YouCompleteMe
  IF(EXISTS "${CMAKE_BINARY_DIR}/compile_commands.json")
  EXECUTE_PROCESS( COMMAND ${CMAKE_COMMAND} -E copy_if_different
    ${CMAKE_BINARY_DIR}/compile_commands.json
    ${CMAKE_SOURCE_DIR}/compile_commands.json
  )
  ENDIF()
endif()
