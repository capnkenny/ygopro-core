cmake_minimum_required(VERSION 3.13 FATAL_ERROR)

project (ocgcore)

option(OCGCORE_INCLUDE_CONAN_PATHS "Include conan_paths toolchain" ON)

if(MSVC)
  # CMake by default appends /W3, so we need to strip it to prevent warnings (D9025)
  string(REGEX REPLACE " /W[0-4]" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
  string(REGEX REPLACE " /W[0-4]" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  string(REGEX REPLACE " /M[TD]d?" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
  string(REGEX REPLACE " /M[TD]d?" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
endif()

if(WIN32)
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
  set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDLL")
endif()

if (OCGCORE_INCLUDE_CONAN_PATHS)
  include(${CMAKE_BINARY_DIR}/conan_paths.cmake)
endif()

set(SOURCES
    card.cpp
    duel.cpp
    effect.cpp
    field.cpp
    group.cpp
    interpreter.cpp
    libcard.cpp
    libdebug.cpp
    libduel.cpp
    libeffect.cpp
    libgroup.cpp
    mem.cpp
    ocgapi.cpp
    operations.cpp
    playerop.cpp
    processor.cpp
    scriptlib.cpp
)

find_package(lua 5.3 REQUIRED)

add_library(ocgcore SHARED ${SOURCES})

if(WIN32)
  set_property(TARGET ocgcore PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreadedDLL")
endif()

target_compile_features(ocgcore PUBLIC cxx_std_14)
target_include_directories(ocgcore
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if(MSVC)
  target_compile_options(ocgcore
    PRIVATE
      /MP
  )
  get_target_property(opts ocgcore COMPILE_OPTIONS)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_compile_options(ocgcore
    PRIVATE
      -pedantic
      -pedantic-errors
      -Wall
      -Wextra
      -Werror
      -Wno-float-equal
      -Wno-padded
  )
else()
  target_compile_options(ocgcore
    PRIVATE
      -pedantic
      -pedantic-errors
      -Wall
      -Wextra
      -Werror
      -Wno-c++98-compat
      -Wno-c++98-compat-pedantic
      -Wno-float-equal
      -Wno-padded
      -Wno-reserved-id-macro
  )
endif()

target_link_libraries(ocgcore
  PRIVATE
    lua::lua
)

if(OCGCORE_INCLUDE_CONAN_PATHS)
    add_custom_command(TARGET ocgcore POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory
      ${CMAKE_BINARY_DIR}/thirdparty
      $<TARGET_FILE_DIR:ocgcore>)
  endif()