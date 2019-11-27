cmake_minimum_required(VERSION 3.1)
project(opusfile LANGUAGES C VERSION 0.11)

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
    src/stream.c)

target_compile_options(opusfile PRIVATE -std=c89)

find_library(OGG_PATH ogg)
find_library(OPUS_PATH opus)
target_link_libraries(opusfile "${OGG_PATH}" "${OPUS_PATH}")

find_path(OPUS_MULTISTREAM_H "opus_multistream.h")
target_include_directories(opusfile PUBLIC include "${OPUS_MULTISTREAM_H}")

install(TARGETS opusfile
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/opusfile)
