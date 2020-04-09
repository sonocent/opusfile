cmake_minimum_required(VERSION 3.1)
project(opusfile LANGUAGES C VERSION 0.11)

# Required modules
include(GNUInstallDirs)

# Disable in-source builds to prevent source tree corruption.
if(" ${CMAKE_SOURCE_DIR}" STREQUAL " ${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "FATAL: In-source builds are not allowed.
    You should create a separate directory for build files."
  )
endif()

option(BUILD_SHARED_LIBS "Build shared library" OFF)

if(APPLE)
  option(BUILD_FRAMEWORK "Build Framework bundle for Apple systems" OFF)
endif()

if(BUILD_FRAMEWORK)
  set(BUILD_SHARED_LIBS ON)
endif()

set(OPUSFILE_PUBLIC_HEADER include/opusfile.h)

add_library(opusfile 
  src/info.c
  src/internal.c
  src/internal.h
  src/opusfile.c
  src/stream.c
  ${OPUSFILE_PUBLIC_HEADER}
)

set_target_properties(
  opusfile PROPERTIES
  PUBLIC_HEADER ${OPUSFILE_PUBLIC_HEADER}
)

if(BUILD_FRAMEWORK)
  set_target_properties(opusfile PROPERTIES
    FRAMEWORK TRUE
    FRAMEWORK_VERSION ${PROJECT_VERSION}
    MACOSX_FRAMEWORK_IDENTIFIER org.xiph.opusfile
    MACOSX_FRAMEWORK_SHORT_VERSION_STRING ${PROJECT_VERSION}
    MACOSX_FRAMEWORK_BUNDLE_VERSION ${PROJECT_VERSION}
    XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
    OUTPUT_NAME Opusfile)
endif()

target_compile_options(opusfile PRIVATE -std=c89)

find_library(OGG_PATH ogg)
find_library(OPUS_PATH opus)
target_link_libraries(opusfile "${OGG_PATH}" "${OPUS_PATH}")

find_path(OPUS_MULTISTREAM_H "opus_multistream.h" PATH_SUFFIXES "opus")
find_path(OGG_H "ogg.h" PATH_SUFFIXES "ogg")
target_include_directories(opusfile PUBLIC 
  "${OPUS_MULTISTREAM_H}"
  "${OGG_H}/.."
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

install(TARGETS opusfile
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  FRAMEWORK DESTINATION ${CMAKE_INSTALL_PREFIX}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/opusfile
)
