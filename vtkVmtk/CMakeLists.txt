project(VTK_VMTK)

if(NOT VMTK_USE_VTK9)
  set(VTK_COMPONENT_PREFIX "vtk")
  set(VTK_TARGET_PREFIX "vtk")
else()
  set(VTK_COMPONENT_PREFIX "")
  set(VTK_TARGET_PREFIX "VTK::")
endif()

include(${VMTK_SOURCE_DIR}/CMake/vmtkFunctions.cmake)

set(dirs
  Utilities
  Common
  ComputationalGeometry
  DifferentialGeometry
  IO
  Misc
  Segmentation
  )

if (VMTK_USE_RENDERING)
  list(APPEND dirs Rendering)
endif ()

option(VTK_VMTK_CONTRIB "Build and install classes in the vtkVmtk/Contrib directory." ON)

if (VTK_VMTK_CONTRIB)
  list(APPEND dirs Contrib)
endif ()

configure_file(
  ${VTK_VMTK_SOURCE_DIR}/vtkvmtkConfigure.h.in
  ${VTK_VMTK_BINARY_DIR}/vtkvmtkConfigure.h
  )

if(APPLE)
  set(CMAKE_SHARED_LINKER_FLAGS "-Wl,-dylib_file,/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib:/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib")
  set(CMAKE_EXE_LINKER_FLAGS "-Wl,-dylib_file,/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib:/System/Library/Frameworks/OpenGL.framework/Versions/A/Libraries/libGL.dylib")
endif()

option(VTK_VMTK_BUILD_TETGEN "Build TetGen and TetGen wrapper. Check TetGen license before you activate this." ON)

set(_VTK_VMTK_BUILD_STREAMTRACER_DEFAULT ON)
if(${VTK_VERSION} VERSION_GREATER_EQUAL "9.2")
  # Interpolated velocity fields were completely reworked in VTK-9.2
  # and VMTK is now incompatible.
  set(_VTK_VMTK_BUILD_STREAMTRACER_DEFAULT OFF)
endif()
option(VTK_VMTK_BUILD_STREAMTRACER "Build static temporal stream tracer." _VTK_VMTK_BUILD_STREAMTRACER_DEFAULT)

if (VTK_USE_COCOA)
  option(VTK_VMTK_USE_COCOA "Build the Cocoa vmtk classes." ON)
endif ()


include_directories (${VTK_VMTK_SOURCE_DIR})
include_directories (${VTK_VMTK_SOURCE_DIR}/Common)
include_directories (${VTK_VMTK_SOURCE_DIR}/ComputationalGeometry)
include_directories (${VTK_VMTK_SOURCE_DIR}/DifferentialGeometry)
include_directories (${VTK_VMTK_SOURCE_DIR}/IO)
include_directories (${VTK_VMTK_SOURCE_DIR}/Misc)
if (VMTK_USE_RENDERING)
  include_directories (${VTK_VMTK_SOURCE_DIR}/Rendering)
endif ()
include_directories (${VTK_VMTK_SOURCE_DIR}/Segmentation)
if (VTK_VMTK_CONTRIB)
  include_directories (${VTK_VMTK_SOURCE_DIR}/Contrib)
endif ()
include_directories (${VTK_VMTK_SOURCE_DIR}/Utilities/vtkvmtkITK)
include_directories (${VTK_VMTK_BINARY_DIR})

set(VTK_DOXYGEN_HOME ${VTK_VMTK_SOURCE_DIR}/Utilities/Doxygen)

file(GLOB files "${VTK_VMTK_BINARY_DIR}/vtkvmtkConfigure*.h")

install(FILES ${files}
  DESTINATION ${VMTK_INSTALL_INCLUDE_DIR}
  COMPONENT Development)


configure_file(${VTK_VMTK_SOURCE_DIR}/vtkvmtk.py ${VTK_VMTK_BINARY_DIR}/vtkvmtk.py COPYONLY)
install(
    FILES ${VTK_VMTK_BINARY_DIR}/vtkvmtk.py
    DESTINATION ${VMTK_MODULE_INSTALL_LIB_DIR}
    COMPONENT PythonRuntimeLibraries
)


set(TETGEN_SOURCE_DIR "${VTK_VMTK_SOURCE_DIR}/Utilities/tetgen1.4.3")

foreach(dir ${dirs})
  add_subdirectory(${dir})
endforeach()
