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

add_library(opusfile SHARED 
    src/info.c
    src/internal.c
    src/internal.h
    src/opusfile.c
    src/stream.c
    include/opusfile.h
)

set_target_properties(
    opusfile PROPERTIES
    PUBLIC_HEADER "include/opusfile.h"
)

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
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/opusfile
)
