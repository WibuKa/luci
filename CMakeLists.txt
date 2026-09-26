cmake_minimum_required(VERSION 3.12)

project(luci C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Build FreeType as a static library from its bundled CMakeLists.txt.
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(SKIP_INSTALL_ALL ON CACHE BOOL "" FORCE)
set(FT_DISABLE_HARFBUZZ ON CACHE BOOL "" FORCE)
set(FT_DISABLE_BZIP2 ON CACHE BOOL "" FORCE)
set(FT_DISABLE_PNG ON CACHE BOOL "" FORCE)
set(FT_DISABLE_BROTLI ON CACHE BOOL "" FORCE)
set(FT_DISABLE_ZLIB ON CACHE BOOL "" FORCE)
add_subdirectory(external/freetype)

# OpenGL loader (glad).
add_library(glad STATIC external/glad/src/glad.c)
target_include_directories(glad PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/external
  ${CMAKE_CURRENT_SOURCE_DIR}/external/glad/include
)
# glad does (void* -> func*) casts via loader; ISO C pedantic warning is expected — silence for this external target
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
  target_compile_options(glad PRIVATE -Wno-pedantic)
endif()

# Main library. Other *.c files are pulled in via luci.c.
add_library(luci ${CMAKE_CURRENT_SOURCE_DIR}/luci.c)
set_target_properties(luci PROPERTIES
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
  ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_CURRENT_SOURCE_DIR}"
  ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_CURRENT_SOURCE_DIR}"
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
  LIBRARY_OUTPUT_DIRECTORY_DEBUG "${CMAKE_CURRENT_SOURCE_DIR}"
  LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_CURRENT_SOURCE_DIR}"
)
target_compile_definitions(luci PRIVATE LUCI_SOURCE_DIR="${CMAKE_CURRENT_SOURCE_DIR}")
target_include_directories(luci PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/external
)
target_link_libraries(luci PRIVATE
  glad
  freetype
)

if(UNIX AND NOT APPLE)
  target_link_libraries(luci PUBLIC m X11 Xrandr)
elseif(WIN32)
  target_link_libraries(luci PUBLIC gdi32)
elseif(APPLE)
  target_link_libraries(luci PUBLIC
    "-framework Cocoa" "-framework CoreVideo" "-framework IOKit")
endif()

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/examples/CMakeLists.txt")
  add_subdirectory(examples)
endif()
