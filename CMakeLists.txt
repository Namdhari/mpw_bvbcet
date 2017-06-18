# This is the CMake script for compiling the FileReader demo.

project( E1_FileReader )

cmake_minimum_required(VERSION 2.8.11)
if(POLICY CMP0053)
  cmake_policy(SET CMP0053 OLD)
endif()
if(POLICY CMP0043)
  cmake_policy(SET CMP0043 OLD)
endif()


include_directories( ./ )

# Find CGAL and CGAL Qt5
find_package(CGAL COMPONENTS Qt5)
include( ${CGAL_USE_FILE} )

# Find Qt5 itself
find_package(Qt5 QUIET COMPONENTS Xml Script OpenGL)

# Find OpenGL
find_package(OpenGL)

# Find QGLViewer
if(Qt5_FOUND)
  find_package(QGLViewer )
endif(Qt5_FOUND)

if(CGAL_Qt5_FOUND AND Qt5_FOUND AND OPENGL_FOUND AND QGLVIEWER_FOUND)

  include_directories ( ${QGLVIEWER_INCLUDE_DIR} )

  qt5_wrap_ui( UI_FILES MainWindow.ui )

  include(AddFileDependencies)

  qt5_generate_moc( "MainWindow.h" "${CMAKE_CURRENT_BINARY_DIR}/MainWindow_moc.cpp" )
  add_file_dependencies( MainWindow_moc.cpp "${CMAKE_CURRENT_SOURCE_DIR}/MainWindow.h" )

  qt5_generate_moc( "Viewer.h" "${CMAKE_CURRENT_BINARY_DIR}/Viewer_moc.cpp" )
  add_file_dependencies( Viewer_moc.cpp "${CMAKE_CURRENT_SOURCE_DIR}/Viewer.h" )

  qt5_add_resources ( CGAL_Qt5_RESOURCE_FILES FileReader.qrc )

    add_file_dependencies( FileReader.cpp "${CMAKE_CURRENT_BINARY_DIR}/MainWindow_moc.cpp"
                                         "${CMAKE_CURRENT_BINARY_DIR}/Viewer_moc.cpp" )
    add_executable  ( FileReader FileReader.cpp ${UI_FILES} ${CGAL_Qt5_RESOURCE_FILES} ${CGAL_Qt5_MOC_FILES})
    qt5_use_modules( FileReader Xml Script OpenGL)
  # Link with Qt libraries
  target_link_libraries( FileReader ${QT_LIBRARIES} )

  # Link with CGAL
  target_link_libraries( FileReader ${CGAL_LIBRARIES} ${CGAL_3RD_PARTY_LIBRARIES} )

  # Link with libQGLViewer, OpenGL
  target_link_libraries( FileReader ${QGLVIEWER_LIBRARIES} ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY})

  add_to_cached_list( CGAL_EXECUTABLE_TARGETS FileReader )


else (CGAL_Qt5_FOUND AND Qt5_FOUND AND OPENGL_FOUND AND QGLVIEWER_FOUND)

  set(FileReader_MISSING_DEPS "")

  if(NOT CGAL_Qt5_FOUND)
    set(FileReader_MISSING_DEPS "the CGAL Qt5 library, ${FileReader_MISSING_DEPS}")
  endif()

  if(NOT Qt5_FOUND)
    set(FileReader_MISSING_DEPS "Qt5, ${FileReader_MISSING_DEPS}")
  endif()

  if(NOT OPENGL_FOUND)
    set(FileReader_MISSING_DEPS "OpenGL, ${FileReader_MISSING_DEPS}")
  endif()

  if(NOT QGLVIEWER_FOUND)
    set(FileReader_MISSING_DEPS "QGLViewer, ${FileReader_MISSING_DEPS}")
  endif()

  message(STATUS "NOTICE: This demo requires ${FileReader_MISSING_DEPS} and will not be compiled.")

endif (CGAL_Qt5_FOUND AND Qt5_FOUND AND OPENGL_FOUND AND QGLVIEWER_FOUND)
