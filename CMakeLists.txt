cmake_minimum_required(VERSION 3.12)

project(mylib)

option(BUILD_DEPS "Build dependencies" ON)
option(SHARED_LIB "Generate a shared library" OFF)
option(USE_EXTENSION "Use extensions library" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)
message(STATUS "BUILD_DEPS: ${BUILD_DEPS}")
message(STATUS "SHARED_LIB: ${SHARED_LIB}")
message(STATUS "USE_EXTENSION: ${USE_EXTENSION}")
message(STATUS "BUILD_EXAMPLES: ${BUILD_EXAMPLES}")

if(USE_EXTENSION)
  add_compile_definitions(USE_EXTENSION)
endif()

if(BUILD_DEPS)
  include(FetchContent)
endif()

if(USE_EXTENSION)
  if(BUILD_DEPS)
    include(cmake/ExternalMylibExtended.cmake)
  else()
    find_package(mylib-e)
    if(NOT TARGET mylib-e)
      message(FATAL_ERROR "mylib-e not found")
    endif()
  endif()
endif()

if(SHARED_LIB)
  add_library(mylib SHARED
    src/mylib.cpp
  )
else()
  add_library(mylib STATIC
    src/mylib.cpp
  )
endif()

target_include_directories(mylib
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if(USE_EXTENSION)
  add_dependencies(mylib mylib-e)
  target_link_libraries(mylib PUBLIC mylib-e)
  #if(BUILD_DEPS)
  #  target_include_directories(mylib
  #    PRIVATE
  #      $<BUILD_INTERFACE:${mylib-e_SOURCE_DIR}/include>
  #  )
  #endif()
endif()

if(BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

install(TARGETS mylib
  EXPORT mylib-config
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
  DESTINATION include
)
export(TARGETS mylib
  FILE "${CMAKE_CURRENT_BINARY_DIR}/mylib_Targets.cmake"
)
install(
  FILES ${CMAKE_CURRENT_SOURCE_DIR}/cmake/mylib-config.cmake
  DESTINATION cmake
)
install(EXPORT mylib-config
  FILE mylib_Targets.cmake
  DESTINATION cmake
)
