if (MPI_C_COMPILER)
    SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
endif()
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MPI_C_FLAGS}")

add_library(ptesmumps ${esmumps_src})
target_link_libraries(ptesmumps ptscotch)
set_target_properties(ptesmumps PROPERTIES OUTPUT_NAME ptesmumps-${SCOTCH_VERSION})
target_compile_definitions(ptesmumps PRIVATE SCOTCH_PTSCOTCH)
target_include_directories(ptesmumps
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../libscotch>
    $<BUILD_INTERFACE:${SCOTCH_SOURCE_DIR}/include>
    )

install(TARGETS ptesmumps EXPORT ptscotch-config
    DESTINATION lib
    INCLUDES DESTINATION include
)

add_executable(exptesmumps ../main_esmumps.c)
target_link_libraries(exptesmumps ptesmumps ptscotch)
target_compile_definitions(exptesmumps PRIVATE SCOTCH_PTSCOTCH)