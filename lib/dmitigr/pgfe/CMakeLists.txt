# -*- cmake -*-
# Copyright (C) Dmitry Igrishin
# For conditions of distribution and use, see files LICENSE.txt or pgfe.hpp

# ------------------------------------------------------------------------------
# Dependencies
# ------------------------------------------------------------------------------

set(CMAKE_MODULE_PATH ${pgfe_SOURCE_DIR}/cmake)

# FindPq.cmake module will use these settings to find libpq.
set(DMITIGR_LIBPQ_LIB_PREFIX "" CACHE PATH
  "Installation prefix (with \"lib\" subdirectory) of the libpq binary")
set(DMITIGR_LIBPQ_INCLUDE_PREFIX "" CACHE PATH
  "Installation prefix (with \"include\" subdirectory) of the libpq includes")

find_package(Pq REQUIRED)
message("libpq includes to use: ${Pq_INCLUDE_DIRS}")
message("Suggested libpq binary to use: ${Suggested_Pq_LIBRARIES}")
include_directories(${Pq_INCLUDE_DIRS})

# ------------------------------------------------------------------------------
# Sources
# ------------------------------------------------------------------------------

set(dmitigr_pgfe_sources
  array_conversions.hpp
  array_conversions.hxx
  array_conversions.tcc
  basic_conversions.hpp

  basics.cmake
  basics.cpp
  basics.hpp.in

  completion.hpp
  completion.hxx

  compositional.hpp
  compositional.hxx

  connection.cpp
  connection.hpp
  connection.hxx

  connection_options.cpp
  connection_options.cxx.in
  connection_options.hpp
  connection_options.hxx

  conversions_api.hpp
  conversions.hpp
  conversions.hxx

  data.cpp
  data.hpp
  data.hxx

  errc.cmake
  errc.cxx
  errc.hpp.in
  errc.hxx

  error.hpp
  error.hxx

  exceptions.hpp
  exceptions.hxx

  message.hpp

  misc.cpp
  misc.hpp

  notice.hpp
  notice.hxx

  notification.hpp
  notification.hxx

  parameterizable.hpp
  parameterizable.hxx

  prepared_statement.cxx
  prepared_statement.hpp
  prepared_statement.hxx

  problem.cpp
  problem.hpp
  problem.hxx

  response.hpp

  row.hpp
  row.hxx

  row_info.hpp
  row_info.hxx

  server_message.hpp

  signal.hpp

  sql_string.cpp
  sql_string.cxx
  sql_string.hpp
  sql_string.hxx

  std_system_error.cpp
  std_system_error.hpp

  types_fwd.hpp

  net.cxx
  net.hxx

  pq.hxx

  sql.cxx
  sql.hxx

  internal/debug.hxx
  internal/enums.hxx
  internal/macros.hxx
  internal/net/inet.cpp
  internal/net/inet.hxx
  internal/std/cstring.hxx
  internal/std/memory/conditional_delete.hxx
  internal/std/string/to_string/numeric.hxx
  )

set_source_files_properties(
  basics.cmake
  basics.hpp.in
  connection_options.cxx.in
  errc.cmake
  errc.hpp.in

  PROPERTIES
  HEADER_FILE_ONLY ON
  )

# ------------------------------------------------------------------------------
# Target tuning
# ------------------------------------------------------------------------------

# Note: Multi-configuration generators (VS, Xcode) append a per-configuration
# subdirectory to PGFE_LIBRARY_OUTPUT_DIR unless a generator expression is used.
set(PGFE_LIBRARY_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}")

add_library(dmitigr_pgfe ${dmitigr_pgfe_sources})

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(dmitigr_pgfe PRIVATE
      -pedantic
      -Wall
      -Wextra
      -Winline
      -Winit-self
      -Wuninitialized
      -Wmaybe-uninitialized
      -Woverloaded-virtual
      -Wsuggest-override
      -Wlogical-op
      -Wswitch)
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(dmitigr_pgfe PRIVATE
      -pedantic
      -Weverything)
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(dmitigr_pgfe PRIVATE
      /Zc:throwingNew,referenceBinding,noexceptTypes
      /errorReport:none
      /nologo
      /utf-8)
  endif()
endif()

if (UNIX)
  target_link_libraries(dmitigr_pgfe ${Suggested_Pq_LIBRARIES} stdc++fs)
elseif (WIN32)
  target_link_libraries(dmitigr_pgfe ${Suggested_Pq_LIBRARIES} Ws2_32.lib)
endif()

set_target_properties(dmitigr_pgfe
  PROPERTIES
  LINKER_LANGUAGE "CXX"
  VERSION ${PGFE_VERSION_PART1}.${PGFE_VERSION_PART2}
  DEBUG_POSTFIX "d")

# ------------------------------------------------------------------------------
# Installations
# ------------------------------------------------------------------------------

install(FILES
  array_conversions.hpp
  array_conversions.hxx
  array_conversions.tcc
  basic_conversions.hpp
  basics.hpp
  completion.hpp
  completion.hxx
  compositional.hpp
  compositional.hxx
  connection.hpp
  connection.hxx
  connection_options.hpp
  connection_options.hxx
  conversions_api.hpp
  conversions.hpp
  conversions.hxx
  data.hpp
  data.hxx
  errc.hpp
  errc.hxx
  error.hpp
  error.hxx
  exceptions.hpp
  exceptions.hxx
  message.hpp
  misc.hpp
  net.hxx
  notice.hpp
  notice.hxx
  notification.hpp
  notification.hxx
  parameterizable.hpp
  parameterizable.hxx
  pq.hxx
  prepared_statement.hpp
  prepared_statement.hxx
  problem.hpp
  problem.hxx
  response.hpp
  row.hpp
  row.hxx
  row_info.hpp
  row_info.hxx
  server_message.hpp
  signal.hpp
  sql.hxx
  sql_string.hpp
  sql_string.hxx
  std_system_error.hpp
  types_fwd.hpp

  DESTINATION ${PGFE_INCLUDES_INSTALL_DIR}/pgfe)

install(TARGETS dmitigr_pgfe
  ARCHIVE DESTINATION ${PGFE_LIBRARY_INSTALL_DIR}
  LIBRARY DESTINATION ${PGFE_LIBRARY_INSTALL_DIR}
  RUNTIME DESTINATION ${PGFE_LIBRARY_INSTALL_DIR})

# ------------------------------------------------------------------------------
# Preprocessing
# ------------------------------------------------------------------------------

# Generated headers (.hpp) should go to the CMAKE_CURRENT_SOURCE_DIR, but
# generated implementations (.cxx) should go to the CMAKE_CURRENT_BINARY_DIR.

include("basics.cmake")
configure_file(basics.hpp.in ${CMAKE_CURRENT_SOURCE_DIR}/basics.hpp @ONLY
  NEWLINE_STYLE UNIX)

include("errc.cmake")
configure_file(errc.hpp.in ${CMAKE_CURRENT_SOURCE_DIR}/errc.hpp @ONLY
  NEWLINE_STYLE UNIX)

configure_file(connection_options.cxx.in ${CMAKE_CURRENT_BINARY_DIR}/connection_options.cxx @ONLY
  NEWLINE_STYLE UNIX)

# ------------------------------------------------------------------------------

add_subdirectory(internal)
add_subdirectory(tests)
