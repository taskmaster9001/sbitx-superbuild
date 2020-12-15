cmake_minimum_required (VERSION 2.8.10 FATAL_ERROR)

project (sbitx-superbuild)

# Set Versions and Branches to build for components
#
set (FFTW3_VER 3.3.9
      CACHE STRING "FFTW3 version to build from officially released source."
      )

# Find Pre-requisites
#
find_program(PATCH_EXECUTABLE patch REQUIRED)
find_package(Git REQUIRED)

# Auxiliary Functions
#
function (read_md5sum md5sum_file md5sum_result)
  file (READ ${md5sum_file}.md5sum md5sum)
  string (REGEX REPLACE "^([^ ]+).*$" "\\1" md5sum ${md5sum})
  set (${md5sum_result} ${md5sum} PARENT_SCOPE)
endfunction (read_md5sum)

# work out what we want to build
#
set (__default_branch main)
set (sbitx_repo https://github.com/taskmaster9001/sbitx)
set (SBITX_TAG cmake)
set (target_dir sbitx)
message (STATUS "Building SBIT-X branch: ${SBITX_TAG}")

# if(EXISTS src/fftw3.tar.gz.md5sum)
#   read_md5sum(src/fftw3.tar.gz.md5sum fftw3_md5sum)
# endif()

# if(EXISTS src/sbitx.tar.gz.md5sum)
#   read_md5sum(src/sbitx.tar.gz.md5sum sbitx_md5sum)
# endif()

include (ExternalProject)


# build and install fftw3 locally so it can be referenced by the
# SBIT-X build
#

ExternalProject_Add (fftw3
  URL http://fftw.org/fftw-${FFTW3_VER}.tar.gz
  # URL_HASH MD5=${fftw3_md5sum}
  # UPDATE_COMMAND ./bootstrap.sh
  PATCH_COMMAND ${PATCH_EXECUTABLE} -p1 -N < ${CMAKE_CURRENT_SOURCE_DIR}/fftw3.patch
  CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --enable-threads --with-combined-threads --enable-static ${EXTRA_FLAGS} # LIBUSB_LIBS=${USB_LIBRARY}
  BUILD_COMMAND $(MAKE) V=1
  INSTALL_COMMAND $(MAKE) install-strip V=1 DESTDIR=""
  STEP_TARGETS update install
  )

#
# custom target to make a fftw3 source tarball
#
add_custom_target (fftw3_sources
  COMMAND ${GIT_EXECUTABLE} archive --format=tgz --prefix=fftw3/ --remote=${fftw3_repo} ${fftw3_TAG} -o ${CMAKE_CURRENT_BINARY_DIR}/fftw3.tar.gz
  COMMENT "Generating the fftw3 upstream source tarball"
  )


# build a list of command line arguments to pass on to the SBIT-X build
#
get_cmake_property (CACHE_VARS CACHE_VARIABLES)
foreach (CACHE_VAR ${CACHE_VARS})
  get_property (CACHE_VAR_HELPSTRING CACHE ${CACHE_VAR} PROPERTY HELPSTRING)
  if (CACHE_VAR_HELPSTRING STREQUAL "No help, variable specified on the command line.")
    get_property (CACHE_VAR_TYPE CACHE ${CACHE_VAR} PROPERTY TYPE)
    if (CACHE_VAR_TYPE STREQUAL "UNINITIALIZED")
      set (CACHE_VAR_TYPE)
    else ()
      set (CACHE_VAR_TYPE :${CACHE_VAR_TYPE})
    endif ()
    list (APPEND CMAKE_ARGS "-D${CACHE_VAR}${CACHE_VAR_TYPE}=${${CACHE_VAR}}")
  endif ()
endforeach ()


# build and optionally install SBIT-X using the fftw3 package built
# above
#
ExternalProject_Get_Property (fftw3 INSTALL_DIR)

ExternalProject_Add (sbitx
  GIT_REPOSITORY ${sbitx_repo}
  GIT_TAG ${SBITX_TAG}
  URL ${CMAKE_CURRENT_SOURCE_DIR}/src/sbitx.tar.gz
  URL_HASH MD5=${sbitx_md5sum}
  PATCH_COMMAND ${PATCH_EXECUTABLE} -p1 -N < ${CMAKE_CURRENT_SOURCE_DIR}/sbitx.patch
  CMAKE_ARGS
  ${CMAKE_ARGS}
  -DCMAKE_PREFIX_PATH=${INSTALL_DIR}
  -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
  ${SBITX_EXTRA_CMAKE_OPTIONS}
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  STEP_TARGETS update configure build install package
  )
ExternalProject_Add_Step (sbitx package
  COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --target package
  COMMAND echo "Built fftw3 from branch/tag: ${fftw3_TAG}"
  COMMAND echo "Built SBIT-X from: ${SBITX_TAG}"
  COMMAND echo "Package(s) generated in ${BINARY_DIR}."
  COMMENT "Generating SBIT-X package."
  DEPENDEES build
  )

set_target_properties (fftw3 PROPERTIES EXCLUDE_FROM_ALL 1)
set_target_properties (sbitx PROPERTIES EXCLUDE_FROM_ALL 1)

add_dependencies (sbitx-configure fftw3-install)
add_dependencies (sbitx-build fftw3-install)
add_dependencies (sbitx-install fftw3-install)
add_dependencies (sbitx-package fftw3-install)

# export traditional targets
add_custom_target (build ALL DEPENDS sbitx-build)
add_custom_target (install DEPENDS sbitx-install)
add_custom_target (package DEPENDS sbitx-package)


#
# custom target to make SBIT-X source tarball
#
add_custom_target (sbitx_sources
  COMMAND ${GIT_EXECUTABLE} archive --format=tgz --prefix=sbitx/ --remote=${sbitx_repo} ${SBITX_TAG} -o ${CMAKE_CURRENT_BINARY_DIR}/sbitx.tar.gz
  COMMENT "Generating the SBIT-X upstream source tarball"
  )


#
# custom target to build a self-contained source tarball suitable for
# a local build
#
add_custom_target (source
  COMMAND ${CMAKE_COMMAND} -E make_directory ${target_dir}
  COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR} ${target_dir}
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${target_dir}/.svn
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${target_dir}/.git
  COMMAND ${CMAKE_COMMAND} -E make_directory ${target_dir}/src
  COMMAND ${CMAKE_COMMAND} -E rename fftw3.tgz ${target_dir}/src/fftw3.tgz
  COMMAND ${CMAKE_COMMAND} -E rename sbitx.tgz ${target_dir}/src/sbitx.tgz
  COMMAND ${CMAKE_COMMAND} -E md5sum ${target_dir}/src/fftw3.tgz > ${target_dir}/src/fftw3.tgz.md5sum
  COMMAND ${CMAKE_COMMAND} -E md5sum ${target_dir}/src/sbitx.tgz > ${target_dir}/src/sbitx.tgz.md5sum
  COMMAND ${CMAKE_COMMAND} -E tar czf ${target_dir}.tgz ${target_dir}
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${target_dir}
  WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
  DEPENDS fftw3_sources sbitx_sources
  COMMENT "

Generating the SBIT-X superbuild self-contained source tarball ${target_dir}.tgz

To use this tarball, transfer it to the target build host, un-compress
and un-tar it;  then configure it including  a CMAKE_INSTALL_PREFIX if
required  (the default  is /usr/local),  then build  and install.  For
example:

    $ tar xzf ${target_dir}.tgz
    $ mkdir build
    $ cd build
    $ cmake <extra-args> ../${target_dir}
    $ cmake --build .
    $ sudo cmake --build . --target install

where <extra-args>  are CMake  command line arguments  that are  to be
passed  onto  the  SBIT-X  CMake  configuration.  The  sort  of  extra
arguments you might want to pass are for example:

  -D SBIT_MANPAGE_DESTINATION=.

which   changes    the   install   path   for    the   manpages   from
<install-prefix>/share to  <install-prefix>.  This  particular example
might be useful on FreeBSD where manpages are expected to reside under
/usr/local/man.

A test install may be carried out in a local user directory by passing
the DESTDIR variable to the build step e.g.:

    $ cmake --build . --target install -- -j2 DESTDIR=$HOME/local

which would install into ~/local/usr/local.

" )
