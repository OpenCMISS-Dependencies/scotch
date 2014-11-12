cmake_minimum_required(VERSION 3.0)
project(SCOTCH VERSION 6.0.3 LANGUAGES C)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
option(BUILD_PTSCOTCH "Build PT-Scotch (parallel version)" ON)
option(USE_GZ "Enable use of .gz compression" ON)
option(USE_BZ2 "Enable use of .bz2 compression" OFF)
option(USE_LZMA "Enable use of .lzma compression" OFF)
option(USE_PTHREAD "Use pthread library" ON)

include(CMakePackageConfigHelpers)

find_package(MPI REQUIRED)

MACRO(ADD_PREFIX prefix rootlist)
  SET(outlist )
  FOREACH(root ${${rootlist}})
    LIST(APPEND outlist ${prefix}${root})
  ENDFOREACH(root)
  SET(${rootlist} ${outlist})
ENDMACRO(ADD_PREFIX)

# need either c99 or -Drestrict=__restrict definition
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")
add_definitions(-DCOMMON_RANDOM_FIXED_SEED 
    -DSCOTCH_RENAME
    -DSCOTCH_RENAME_PARSER
    #-Drestrict=__restrict
    #-DSCOTCH_PTHREAD
)
if (USE_PTHREAD)
    # Add the commons pthread definition to all targets.
    # Will only compile scotch with pthread enabled, ptscotch with pthread
    # seems somewhat unstable / error-prone to use (see INSTALL.txt)
    add_definitions(-DCOMMON_PTHREAD)
endif()
SET(COMPRESSION_LIBS )
if (USE_GZ)
    FIND_PACKAGE(ZLIB REQUIRED)
    add_definitions(-DCOMMON_FILE_COMPRESS_GZ)
    include_directories(${ZLIB_INCLUDE_DIRS})
    LIST(APPEND COMPRESSION_LIBS ${ZLIB_LIBRARIES})
endif()
if (USE_BZ2)
    FIND_PACKAGE(BZip2 REQUIRED)
    add_definitions(-DCOMMON_FILE_COMPRESS_BZ2)
    include_directories(${BZIP2_INCLUDE_DIRS})
    LIST(APPEND COMPRESSION_LIBS ${BZIP2_LIBRARIES})
endif()
if (USE_LZMA)
    FIND_PACKAGE(LZMA REQUIRED)
    add_definitions(-DCOMMON_FILE_COMPRESS_LZMA)
    include_directories(${LZMA_INCLUDE_DIRS})
    LIST(APPEND COMPRESSION_LIBS ${LZMA_LIBRARIES})
endif()

set(SCOTCH_INC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
if (NOT EXISTS ${SCOTCH_INC_DIR}/scotch.h)
    SET(NEED_HEADER_CREATION YES)
endif()

add_subdirectory(src/esmumps)
add_subdirectory(src/libscotch)
add_subdirectory(src/scotch)
install(DIRECTORY include/ DESTINATION include FILES_MATCHING PATTERN "*.h")

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(src/check)
endif()