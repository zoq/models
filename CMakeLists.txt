cmake_minimum_required(VERSION 2.6)
project (models)

# Ensure that we have a C++11 compiler.  In newer versions of CMake, this is
# done with target_compile_features().
if ((${CMAKE_MAJOR_VERSION} LESS 3 OR
    (${CMAKE_MAJOR_VERSION} EQUAL 3 AND ${CMAKE_MINOR_VERSION} LESS 1))
    AND NOT FORCE_CXX11)
  # Older versions of CMake do not support target_compile_features(), so we have
  # to use something kind of hacky.
  include(CMake/CXX11.cmake)
  check_for_cxx11_compiler(HAS_CXX11)
  if(NOT HAS_CXX11)
    message(FATAL_ERROR "No C++11 compiler available!")
  endif()
  enable_cxx11()
endif ()

# Include modules in the CMake directory.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMake")

# Set the CFLAGS and CXXFLAGS.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
SET(CMAKE_CXX_FLAGS "-std=c++11 -O0 ${CMAKE_CXX_FLAGS}")

# If using clang, we have to link against libc++ depending on the
# OS (at least on some systems). Further, gcc sometimes optimizes calls to
# math.h functions, making -lm unnecessary with gcc, but it may still be
# necessary with clang.
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  IF (APPLE)
    # detect OS X version. Use '/usr/bin/sw_vers -productVersion' to
    # extract V from '10.V.x'.)
    EXEC_PROGRAM(/usr/bin/sw_vers ARGS
        -productVersion OUTPUT_VARIABLE MACOSX_VERSION_RAW)
    string(REGEX REPLACE
        "10\\.([0-9]+).*" "\\1"
        MACOSX_VERSION
        "${MACOSX_VERSION_RAW}")

     # OSX Lion (10.7) and OS X Mountain Lion (10.8) doesn't automatically
     # select the right stdlib.
    if(${MACOSX_VERSION} LESS 9)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
     endif(${MACOSX_VERSION} LESS 9)
  endif(APPLE)
endif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")

# Now, find the libraries we need to compile against.
find_package(Armadillo 3.6.0 REQUIRED)
find_package(Mlpack REQUIRED)

# Include directories for the previous dependencies.
set(MLPACK_INCLUDE_DIRS ${MLPACK_INCLUDE_DIR} ${ARMADILLO_INCLUDE_DIR})
set(MLPACK_LIBRARIES ${MLPACK_LIBRARIES} ${ARMADILLO_LIBRARIES})

include_directories(${MLPACK_INCLUDE_DIRS})

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/)

# Recurse into each model.
set(DIRS
  model1
  model2
)

foreach(dir ${DIRS})
  add_subdirectory(${dir})
endforeach()
