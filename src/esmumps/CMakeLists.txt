file(GLOB esmumps_src *.c)
add_library(esmumps ${esmumps_src})
target_link_libraries(esmumps scotch)
set_target_properties(esmumps PROPERTIES OUTPUT_NAME esmumps-${SCOTCH_VERSION})
target_include_directories(esmumps
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../libscotch>
    $<BUILD_INTERFACE:${SCOTCH_SOURCE_DIR}/include>
    )

install(TARGETS esmumps EXPORT scotch-config
    DESTINATION lib
    INCLUDES DESTINATION include
)
install(TARGETS esmumps EXPORT ptscotch-config
    DESTINATION lib
    INCLUDES DESTINATION include
)

if (BUILD_PTSCOTCH)
    add_subdirectory(ptesmumps)
endif()

add_executable(exesmumps main_esmumps.c)
target_link_libraries(exesmumps esmumps scotch)