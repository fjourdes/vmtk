cmake_minimum_required(VERSION 3.12...3.29.1)

project(VMTK)

include(GNUInstallDirs)

# vmtk versions number
set(VMTK_VERSION_MAJOR 1)
set(VMTK_VERSION_MINOR 5)
set(VMTK_VERSION_PATCH 0)
set(VMTK_VERSION
    "${VMTK_VERSION_MAJOR}.${VMTK_VERSION_MINOR}.${VMTK_VERSION_PATCH}")

set(VMTK_CMAKE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/CMake)
add_subdirectory(CMake)
list(INSERT CMAKE_MODULE_PATH 0
  ${VMTK_CMAKE_DIR}
  ${vtkAddon_CMAKE_DIR}
  )


find_package (Python3 COMPONENTS Interpreter Development)

if (NOT VMTK_PYTHON_VERSION)
  set(VMTK_PYTHON_VERSION "python${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}" CACHE STRING "" FORCE)
endif ()
mark_as_advanced(VMTK_PYTHON_VERSION)

set(PYTHON_INCLUDE_PATH ${Python3_INCLUDE_DIRS} CACHE STRING "" FORCE)


set(VTK_WRAP_PYTHON_FIND_LIBS OFF CACHE BOOL "" FORCE)
set(VTK_UNDEFINED_SYMBOLS_ALLOWED ON CACHE BOOL "" FORCE)

#-----------------------------------------------------------------------------
# ITK setup
option( USE_SYSTEM_ITK
  "Exclude ITK from SuperBuild and use an existing build instead." ON )
if( USE_SYSTEM_ITK )
  set(ITK_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/ITK-5.4" CACHE STRING "ITK cmake search directory")
  find_package( ITK REQUIRED )
  include( ${ITK_USE_FILE} )
endif()

set(VMTK_RENDERING_BACKEND "OpenGL2" CACHE STRING "Graphic backend for VTK")

#-----------------------------------------------------------------------------
# VTK setup
option( USE_SYSTEM_VTK
  "Exclude VTK from SuperBuild and use an existing build instead." ON )
if( USE_SYSTEM_VTK )
  set(VTK_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/vtk-9.1" CACHE STRING "VTK cmake search directory")
  find_package( VTK REQUIRED )
  if( ${VTK_VERSION_MAJOR} LESS 6 )
    include( ${VTK_USE_FILE} )
  endif()
endif()

message("VMTK_BINARY_DIR=${VMTK_BINARY_DIR}")
# Set up our directory structure for output libraries and binaries
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${VMTK_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${VMTK_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${VMTK_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

set(VMTK_INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR})
set(VMTK_INSTALL_BINDIR ${CMAKE_INSTALL_BINDIR})

set(VMTK_INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR})
set(VMTK_INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR})

set(VMTK_INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR})
set(VMTK_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR})


if (NOT DEFINED VMTK_PYTHON_SITE_PACKAGES_PREFIX)
    vtk_module_python_default_destination(VMTK_PYTHON_SITE_PACKAGES_PREFIX)
endif()
set(VMTK_MODULE_INSTALL_LIB_DIR ${VMTK_PYTHON_SITE_PACKAGES_PREFIX}/vmtk)
message("VMTK_MODULE_INSTALL_LIB_DIR=${VMTK_MODULE_INSTALL_LIB_DIR}")


# Append the library version information to the library target properties.
option(VMTK_WITH_LIBRARY_VERSION "Build with library version information" OFF)
if(VMTK_WITH_LIBRARY_VERSION)
  # This setting of SOVERSION assumes that any API change
  # will increment either the minor or major version number of vmtk.
  set(VMTK_LIBRARY_PROPERTIES
      VERSION "${VMTK_VERSION}"
      SOVERSION "${VMTK_VERSION_MAJOR}.${VMTK_VERSION_MINOR}"
     )
endif()


if (APPLE AND VMTK_BREW_PYTHON)
  if(VMTK_BREW_PYTHON)
    option(VMTK_BREW_PYTHON "Link against Homebrew Python" ON)
    set(PYTHON_SHEBANG ${PYTHON_EXECUTABLE})
  else()
    set(PYTHON_SHEBANG "/usr/bin/env python")
  endif()
  #if(VMTK_BREW_PYTHON)
  #  execute_process(
  #    COMMAND /usr/local/bin/python -c "from distutils import sysconfig; print sysconfig.get_python_inc()"
  #    OUTPUT_VARIABLE PYTHON_INCLUDE_DIR
  #    OUTPUT_STRIP_TRAILING_WHITESPACE)
  #  set(PYTHON_LIBRARY ${PYTHON_INCLUDE_DIR}../../lib/lib${VMTK_PYTHON_VERSION}.dylib)
  #endif()
elseif (UNIX)
  set(PYTHON_SHEBANG "/usr/bin/env python")
endif ()

option( VMTK_USE_X
  "Compile vmtk with X11." OFF )
if( VMTK_USE_X )
  set ( VTK_USE_X ON CACHE BOOL "" FORCE )
else ()
  set ( VTK_USE_X OFF CACHE BOOL "" FORCE )
endif()



# Superbuild setup
option( VMTK_USE_SUPERBUILD
  "Build VMTK and the projects it depends on via SuperBuild.cmake." OFF )

set(VMTK_TEST_DATA_SOURCE "git-submodule" CACHE STRING "git-submodule initializes the vmtk-test-data repository. in-place assumes test data is already available.")
set_property(CACHE VMTK_TEST_DATA_SOURCE PROPERTY STRINGS in-place git-submodule)

# Git setup
if (VMTK_USE_SUPERBUILD OR VMTK_TEST_DATA_SOURCE MATCHES "git-submodule")
  if( NOT GIT_EXECUTABLE )
    find_package( Git REQUIRED )
  endif()

  option( GIT_PROTOCOL_HTTPS
    "Use HTTPS for git access (useful if behind a firewall)" OFF )
  if( GIT_PROTOCOL_HTTPS )
    set( GIT_PROTOCOL "https" CACHE STRING "Https protocol for file transfer" FORCE )
  else()
    set( GIT_PROTOCOL "git" CACHE STRING "Git protocol for file transfer" FORCE )
  endif()
  mark_as_advanced( GIT_PROTOCOL )
endif()

option( VMTK_USE_ITK5
  "Compile vmtk against ITK 5. OFF is ITK 4.13" ON )

option( VMTK_USE_VTK9
  "Compile vmtk against VTK 9. OFF is VTK 8" ON )


if( NOT VMTK_USE_VTK9 )
  message(FATAL_ERROR "This version of VMTK only supports VTK9")
endif()

option( VMTK_BUILD_TESTING
  "Build the VMTK testing libraries" OFF)

if(VMTK_BUILD_TESTING)
  add_subdirectory(tests)
endif()

option( VMTK_BUNJEE_BUILD
  "Build VMTK and the projects it depends on ready for being linked from Bunjee." OFF )

option( BUILD_VMTK_DOCUMENTATION "Build the methods to build doxygen documentation for VMTK locally" OFF )

option( VMTK_USE_RENDERING "Build VMTK Rendering library." ON)

mark_as_advanced(BUILD_VMTK_DOCUMENTATION)
if(BUILD_VMTK_DOCUMENTATION)
  set(BUILD_DOCUMENTATION ON)
endif()


if ( NOT ITK_FOUND )
    find_package( ITK REQUIRED )
    include( ${ITK_USE_FILE} )
endif()

if ( NOT VTK_FOUND )
find_package( VTK REQUIRED )
    if( ${VTK_VERSION_MAJOR} LESS 6 )
      include( ${VTK_USE_FILE} )
    endif()
endif()


option(VMTK_SCRIPTS_ENABLED "Build and install vmtkScripts and Pypes" ON)
mark_as_advanced(VMTK_SCRIPTS_ENABLED)

if(VMTK_SCRIPTS_ENABLED)
  set(vmtk_directories
    PypeS
    vtkVmtk
    vmtkScripts
    )
else()
  set(vmtk_directories
    vtkVmtk
    )
endif()

foreach(vmtk_directory ${vmtk_directories})
  add_subdirectory(${vmtk_directory})
endforeach()

option(VMTK_MINIMAL_INSTALL "Do not install individual scripts, only vmtk.py" OFF)



if(VMTK_SCRIPTS_ENABLED)

    set(VMTK_SRCS
       vmtk.py
    )


  foreach (SCRIPT_FILE ${VMTK_SRCS})
    configure_file(${VMTK_SOURCE_DIR}/${SCRIPT_FILE} ${VMTK_BINARY_DIR}/${SCRIPT_FILE} COPYONLY)
  endforeach ()

  configure_file(${VMTK_SOURCE_DIR}/__init__.py.in ${VMTK_BINARY_DIR}/__init__.py COPYONLY)

  install(FILES __init__.py.in
    DESTINATION ${VMTK_MODULE_INSTALL_LIB_DIR}
    COMPONENT PythonRuntimeLibraries
    RENAME __init__.py)

  if (NOT WIN32)

    set (STRIPPED_SCRIPTS_SRCS )

    foreach (SCRIPT_FILE ${VMTK_SRCS})
      configure_file(${VMTK_SOURCE_DIR}/${SCRIPT_FILE} ${VMTK_BINARY_DIR}/${SCRIPT_FILE} COPYONLY)
      string(REGEX REPLACE ".py" "" STRIPPED_SCRIPT_FILE ${SCRIPT_FILE})
      configure_file(${VMTK_SOURCE_DIR}/${SCRIPT_FILE} ${VMTK_BINARY_DIR}/${STRIPPED_SCRIPT_FILE})
      set (STRIPPED_SCRIPTS_SRCS ${STRIPPED_SCRIPTS_SRCS} ${VMTK_BINARY_DIR}/${STRIPPED_SCRIPT_FILE})
    endforeach ()

    install(PROGRAMS ${STRIPPED_SCRIPTS_SRCS}
      DESTINATION ${VMTK_INSTALL_BIN_DIR}
      COMPONENT RuntimeExecutables)

  else ()

    foreach (SCRIPT_FILE ${VMTK_SRCS})
      configure_file(${VMTK_SOURCE_DIR}/${SCRIPT_FILE} ${VMTK_BINARY_DIR}/${SCRIPT_FILE} COPYONLY)
      string(REGEX REPLACE ".py" "-exe.py" STRIPPED_SCRIPT_FILE ${SCRIPT_FILE})
      configure_file(${VMTK_SOURCE_DIR}/${SCRIPT_FILE} ${VMTK_BINARY_DIR}/${STRIPPED_SCRIPT_FILE} COPYONLY)
      set (STRIPPED_SCRIPTS_SRCS ${STRIPPED_SCRIPTS_SRCS} ${VMTK_BINARY_DIR}/${STRIPPED_SCRIPT_FILE})
    endforeach ()
    
    message("VMTK_INSTALL_BIN_DIR=${VMTK_INSTALL_BIN_DIR}")
    install(PROGRAMS ${STRIPPED_SCRIPTS_SRCS}
      DESTINATION ${VMTK_INSTALL_BIN_DIR}
      COMPONENT RuntimeExecutables)

  endif ()

  # option(VMTK_ENABLE_DISTRIBUTION "Enable distribution targets." OFF)
  # if(VMTK_ENABLE_DISTRIBUTION)
  #   add_subdirectory(distribution)
  # endif()

  install(EXPORT VMTK-Targets DESTINATION ${VMTK_BINARY_DIR})

  get_property(_wrap_hierarchy_targets GLOBAL PROPERTY VMTK_WRAP_HIERARCHY_TARGETS)
  if(_wrap_hierarchy_targets)
    foreach(target ${_wrap_hierarchy_targets})
      set(VMTK_WRAP_HIERARCHY_FILES_CONFIG
  "${VMTK_WRAP_HIERARCHY_FILES_CONFIG}
  set(${target}_WRAP_HIERARCHY_FILE
    \"${${target}_WRAP_HIERARCHY_FILE}\")"
  )
    endforeach()
  endif()

  configure_file(
    ${VMTK_SOURCE_DIR}/CMake/VMTKConfig.cmake.in
    ${VMTK_BINARY_DIR}/VMTKConfig.cmake
  @ONLY IMMEDIATE
  )
  install(FILES ${VMTK_BINARY_DIR}/VMTKConfig.cmake
    TYPE LIB
  )

  configure_file(
    ${VMTK_SOURCE_DIR}/CMake/VMTKUse.cmake.in
    ${VMTK_BINARY_DIR}/VMTKUse.cmake
  @ONLY IMMEDIATE
  )
  install(FILES ${VMTK_BINARY_DIR}/VMTKUse.cmake
    TYPE LIB
  )

endif()
