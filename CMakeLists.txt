cmake_minimum_required(VERSION 3.3)
project(SCOTCH VERSION 6.0.3 LANGUAGES C)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
option(PACKAGE_CONFIG_DIR "Directory for package config files (relative to CMAKE_INSTALL_PREFIX)" "lib/cmake")
option(BUILD_PTSCOTCH "Build PT-Scotch (parallel version)" ON)
option(USE_ZLIB "Enable use of .gz compression" ON)
option(USE_BZ2 "Enable use of .bz2 compression" ON)
option(USE_LZMA "Enable use of .lzma compression" OFF)
option(EXIT_ON_ERROR "Use libscotcherrexit instead of libscotcherr" NO)
option(USE_THREADS "Use threading" ON)

include(CMakePackageConfigHelpers)

find_package(MPI REQUIRED)
if (USE_THREADS)
    find_package(Threads QUIET)
    if (NOT Threads_FOUND)
        message(WARNING "Threading was requested (USE_THREADS=${USE_THREADS}), but package could not be found. Disabling.")
        SET(USE_THREADS NO)    
    endif()
endif()

MACRO(ADD_PREFIX prefix rootlist)
  SET(outlist )
  FOREACH(root ${${rootlist}})
    LIST(APPEND outlist ${prefix}${root})
  ENDFOREACH(root)
  SET(${rootlist} ${outlist})
ENDMACRO(ADD_PREFIX)

if (MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /Qstd=c99")
    add_definitions(-DCOMMON_WINDOWS 
        -DHAVE_UINT_T
        -Drestrict=__restrict
        -DYY_NO_UNISTD_H
        -DCOMMON_STUB_FORK)
elseif(MINGW)
    # The extra definition "-D'pipe(pfds)=_pipe(pfds,1024,0x8000)'" is automatically defined in src/libscotch/common.h
    # if COMMON_WINDOWS is used
    add_definitions(-DCOMMON_WINDOWS -DCOMMON_STUB_FORK)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")    
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
endif()
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")
add_definitions(-DCOMMON_RANDOM_FIXED_SEED 
    -DSCOTCH_RENAME
    -DSCOTCH_RENAME_PARSER
)
if (APPLE)
    add_definitions(-DCOMMON_TIMING_OLD)
endif()
if (USE_THREADS)
    # Add the commons pthread definition to all targets.
    # Will only compile scotch with pthread enabled, ptscotch with pthread
    # seems somewhat unstable / error-prone to use (see INSTALL.txt)
    include(CheckIncludeFile)
    CHECK_INCLUDE_FILE("pthread.h" COMMON_PTHREAD)
    if (${COMMON_PTHREAD})
        add_definitions(-DCOMMON_PTHREAD)
    else()
        SET(USE_THREADS NO)
    endif()
    if (APPLE)
        add_definitions(-DCOMMON_PTHREAD_BARRIER)
    endif()
endif()

INCLUDE(CheckIncludeFiles)
SET(SEARCHHEADERS strings malloc stdint)
foreach(hdr ${SEARCHHEADERS})
    STRING(TOUPPER ${hdr} HDR)
    STRING(REPLACE "/" "_" HDR ${HDR})
    SET(VARNAME "HAVE_${HDR}_H")
    CHECK_INCLUDE_FILES("${hdr}.h" ${VARNAME})
    if (${${VARNAME}})
        add_definitions(-D${VARNAME})
    endif()
endforeach()

SET(COMPRESSION_LIBS )
if (USE_ZLIB)
    FIND_PACKAGE(ZLIB ${ZLIB_VERSION} REQUIRED)
    add_definitions(-DCOMMON_FILE_COMPRESS_GZ)
    LIST(APPEND COMPRESSION_LIBS zlib)
endif()
if (USE_BZ2)
    FIND_PACKAGE(BZip2 ${BZIP2_VERSION} REQUIRED)
    add_definitions(-DCOMMON_FILE_COMPRESS_BZ2)
    LIST(APPEND COMPRESSION_LIBS bz2)
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
if(MSVC)# WIN32
    include_directories(${SCOTCH_INC_DIR}/win32)
endif()

function(CREATE_DEPENDENCIES_FILE EXPORT_NAME)
    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake
        "include(CMakeFindDependencyMacro)\r\n"
        	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
    )
    if (USE_ZLIB)
        file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake "find_dependency(ZLIB ${ZLIB_VERSION})\r\n")
    endif()
    if (USE_BZ2)
        file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake "find_dependency(BZip2 ${BZIP2_VERSION})\r\n")
    endif()
    if (USE_LZMA)
        file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake "find_dependency(LZMA ${LZMA_VERSION})\r\n")
    endif()
    if (USE_THREADS)
        file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake "find_dependency(Threads)\r\n")
    endif()
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_NAME}-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
endfunction()

add_subdirectory(src/esmumps)
add_subdirectory(src/libscotch)
add_subdirectory(src/scotch)
if (MSVC)
    install(DIRECTORY include/ DESTINATION include FILES_MATCHING PATTERN "*.h")
else()
    install(FILES 
        include/ptscotch.h
        include/ptscotchf.h
        include/scotch.h
        include/scotchf.h 
        DESTINATION include)
endif()

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(src/check)
endif()