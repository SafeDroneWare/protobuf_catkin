cmake_minimum_required(VERSION 2.8.12)
project(protobuf_catkin)

find_package(catkin REQUIRED)

# Build options
catkin_package_xml()
set(protobuf_VERSION ${${PROJECT_NAME}_VERSION})  # set by catkin_package()
set(protobuf_SOVERSION 14)  # from protobuf/src/Makefile.am
set(protobuf_CMAKE_ARGS
  #-Dprotobuf_VERSION=${protobuf_VERSION}  # would case CMake warning if policy CMP0048 is not set
  -Dprotobuf_SOVERSION=${protobuf_SOVERSION}
  -Dprotobuf_BUILD_TESTS=OFF
  -DBUILD_SHARED_LIBS=ON
  -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
)

###################################
## catkin specific configuration ##
###################################
catkin_package(
  CFG_EXTRAS protobuf-generate-cpp.cmake protobuf-extras.cmake
)

include(ExternalProject)

##############
## Download ##
##############
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/protobuf/cmake/CMakeLists.txt")
  set(protobuf_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/protobuf")
  add_custom_target(protobuf_src COMMENT "Using protobuf from submodule in ${protobuf_SOURCE_DIR}. Skipping download step.")
else()
  set(protobuf_SOURCE_DIR "${CMAKE_CURRENT_BINARY_DIR}/protobuf/src/protobuf")
  ExternalProject_Add(protobuf_src
    PREFIX protobuf
    URL https://github.com/google/protobuf/archive/v3.4.0.tar.gz
    SOURCE_DIR "protobuf/src/protobuf"
  #  PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings.patch &&
  #                patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings_wire_format.patch
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
  )
endif()

###########
## Build ##
###########
ExternalProject_Add(protobuf_build
  PREFIX protobuf
  DOWNLOAD_COMMAND ""
  SOURCE_DIR "${protobuf_SOURCE_DIR}/cmake"
  BINARY_DIR "protobuf/build"
  CMAKE_ARGS ${protobuf_CMAKE_ARGS}
  INSTALL_COMMAND ""
)
add_dependencies(protobuf_build protobuf_src)

# "install" in devel-space unless devel-space is inside the build directory
if(CATKIN_DEVEL_PREFIX AND NOT CATKIN_DEVEL_PREFIX STREQUAL "${CMAKE_BINARY_DIR}/devel")
  ExternalProject_Add(protobuf_devel
    PREFIX protobuf
    DOWNLOAD_COMMAND ""
    SOURCE_DIR "${protobuf_SOURCE_DIR}/cmake"
    BINARY_DIR "protobuf/build"
    BUILD_COMMAND ""
    INSTALL_DIR "${CATKIN_DEVEL_PREFIX}"
    CMAKE_ARGS ${protobuf_CMAKE_ARGS} "-DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>"
  )
  add_dependencies(protobuf_devel protobuf_build)
endif()

#############
## Install ##
#############

install(CODE "
  execute_process(
    COMMAND ${CMAKE_COMMAND} \"-DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}\" .
    COMMAND ${CMAKE_MAKE_PROGRAM} install
    WORKING_DIRECTORY \"${CMAKE_CURRENT_BINARY_DIR}/protobuf/build\"
  )
")
