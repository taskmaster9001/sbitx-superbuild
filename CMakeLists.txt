cmake_minimum_required (VERSION 2.8.10 FATAL_ERROR)

project (sbitx-superbuild)

# Set Versions and Branches to build for components
#
set (FFTW3_VER 3.3.9
      CACHE STRING "FFTW3 version to build from officially released source."
)

# Find Pre-requisites
#
find_package(Git REQUIRED)

# Work out what we want to build
#
set (__default_branch main)
set (sbitx_repo https://github.com/afarhan/sbitx)
set (SBITX_TAG ${__default_branch})
set (target_dir sbitx)
message (STATUS "Building SBIT-X branch: ${SBITX_TAG}")

# change default install location to /usr instead of /usr/local
if (CMAKE_INSTALL_PREFIX STREQUAL "/usr/local")
  set (CMAKE_INSTALL_PREFIX "/usr")
endif ()

include (ExternalProject)

# build and install fftw3 locally so it can be referenced by the
# SBIT-X build
ExternalProject_Add (fftw3
  URL http://fftw.org/fftw-${FFTW3_VER}.tar.gz
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND
    <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --enable-shared --enable-threads ${EXTRA_FLAGS}
  BUILD_COMMAND $(MAKE) V=1
  INSTALL_COMMAND
    $(MAKE) install-strip V=1
  STEP_TARGETS build install
)

ExternalProject_Add (fftw3f
  URL http://fftw.org/fftw-${FFTW3_VER}.tar.gz
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND
    <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --enable-shared --enable-float --enable-threads ${EXTRA_FLAGS}
  BUILD_COMMAND $(MAKE) V=1
  INSTALL_COMMAND
    $(MAKE) install-strip V=1
  STEP_TARGETS build install
)

ExternalProject_Add (sbitx
  GIT_REPOSITORY ${sbitx_repo}
  GIT_TAG ${SBITX_TAG}
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CMAKE_ARGS
    ${CMAKE_ARGS}
    ${SBITX_EXTRA_CMAKE_OPTIONS}
  STEP_TARGETS build install
)

ExternalProject_Get_Property (sbitx BINARY_DIR)
ExternalProject_Add_Step (sbitx uninstall
  WORKING_DIRECTORY ${BINARY_DIR}
  COMMAND $(MAKE) uninstall
)
ExternalProject_Add_StepTargets (sbitx uninstall)

ExternalProject_Get_Property (fftw3 BINARY_DIR)
ExternalProject_Add_Step (fftw3 uninstall
  WORKING_DIRECTORY ${BINARY_DIR}
  COMMAND $(MAKE) uninstall
)
ExternalProject_Add_StepTargets (fftw3 uninstall)

ExternalProject_Get_Property (fftw3f BINARY_DIR)
ExternalProject_Add_Step (fftw3f uninstall
  WORKING_DIRECTORY ${BINARY_DIR}
  COMMAND $(MAKE) uninstall
)
ExternalProject_Add_StepTargets (fftw3f uninstall)

set_target_properties (fftw3 PROPERTIES EXCLUDE_FROM_ALL 1)
set_target_properties (fftw3f PROPERTIES EXCLUDE_FROM_ALL 1)
set_target_properties (sbitx PROPERTIES EXCLUDE_FROM_ALL 1)

# Export traditional targets
add_custom_target (build ALL DEPENDS sbitx-build)
add_custom_target (install DEPENDS sbitx-install)
add_custom_target (uninstall DEPENDS sbitx-uninstall)
