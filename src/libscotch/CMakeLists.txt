if (NEED_HEADER_CREATION)
    ## dummysizes
    add_executable(dummysizes dummysizes.c)
    target_include_directories(dummysizes 
        PRIVATE .)
    target_compile_definitions(dummysizes
        PRIVATE SCOTCH_VERSION=${SCOTCH_VERSION_MAJOR}
        SCOTCH_RELEASE=${SCOTCH_VERSION_MINOR}
        SCOTCH_PATCHLEVEL=${SCOTCH_VERSION_PATCH}
    )
    add_custom_target(scotch_header
        COMMAND dummysizes 
        ${CMAKE_CURRENT_SOURCE_DIR}/library.h scotch.h
        COMMAND ${CMAKE_COMMAND} -E copy_if_different scotch.h ${SCOTCH_INC_DIR}
        COMMAND dummysizes 
        ${CMAKE_CURRENT_SOURCE_DIR}/library_f.h scotchf.h
        COMMAND ${CMAKE_COMMAND} -E copy_if_different scotchf.h ${SCOTCH_INC_DIR})

    ## parser stuff
    FIND_PACKAGE(FLEX QUIET)
    SET(parser_ll ${CMAKE_CURRENT_SOURCE_DIR}/parser_ll.c)
    IF(FLEX_FOUND)
        ADD_CUSTOM_COMMAND(
          OUTPUT ${parser_ll}
          COMMAND ${FLEX_EXECUTABLE} -Pscotchyy --outfile=parser_ll.c
              ${CMAKE_CURRENT_SOURCE_DIR}/parser_ll.l
          COMMAND ${CMAKE_COMMAND} -E copy_if_different parser_ll.c ${parser_ll}
          WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
          COMMENT "Flexing parser_ll.c"
        )
    ELSE()
        message(WARNING "No flex generator found. Using last-resort version.")
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/last_resort/parser_ll.c
            ${parser_ll} COPYONLY)
    ENDIF()
    
    FIND_PACKAGE(BISON QUIET)
    SET(parser_yy ${CMAKE_CURRENT_SOURCE_DIR}/parser_yy.c)
    SET(parser_ly ${CMAKE_CURRENT_SOURCE_DIR}/parser_ly.h)
    IF(BISON_FOUND)
        ADD_CUSTOM_COMMAND(
          OUTPUT ${parser_yy} ${parser_ly}
          COMMAND ${BISON_EXECUTABLE} -pscotchyy -y -b y -d -v
              ${CMAKE_CURRENT_SOURCE_DIR}/parser_yy.y
          COMMAND ${CMAKE_COMMAND} -E copy_if_different y.tab.c ${parser_yy}
          COMMAND ${CMAKE_COMMAND} -E copy_if_different y.tab.h ${parser_ly}
          WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
          COMMENT "Running bison to generate files parser_yy.c/parser_ly.h"
        )
    ELSE()
        message(WARNING "No bison generator found. Using last-resort version.")
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/last_resort/parser_yy.c
            ${parser_yy} COPYONLY)
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/last_resort/parser_ly.h
            ${parser_ly} COPYONLY)
    ENDIF()
endif()

SET(SCOTCH_SRC arch.c
arch_build.c
arch_cmplt.c
arch_cmpltw.c
arch_deco.c
arch_dist.c
arch_hcub.c
arch_mesh.c
arch_tleaf.c
arch_torus.c
arch_vcmplt.c
arch_vhcub.c
bgraph.c
bgraph_bipart_bd.c
bgraph_bipart_df.c
bgraph_bipart_ex.c
bgraph_bipart_fm.c
bgraph_bipart_gg.c
bgraph_bipart_gp.c
bgraph_bipart_ml.c
bgraph_bipart_st.c
bgraph_bipart_zr.c
bgraph_check.c
bgraph_store.c
common.c
common_file.c
common_file_compress.c
common_file_uncompress.c
common_integer.c
common_memory.c
common_string.c
common_stub.c
common_thread.c
fibo.c
gain.c
geom.c
graph.c
graph_base.c
graph_band.c
graph_check.c
graph_coarsen.c
graph_induce.c
graph_io.c
graph_io_chac.c
graph_io_habo.c
graph_io_mmkt.c
graph_io_scot.c
graph_list.c
graph_match.c
hall_order_hd.c
hall_order_hf.c
hall_order_hx.c
hgraph.c
hgraph_check.c
hgraph_induce.c
hgraph_order_bl.c
hgraph_order_cp.c
hgraph_order_gp.c
hgraph_order_hd.c
hgraph_order_hf.c
hgraph_order_hx.c
hgraph_order_kp.c
hgraph_order_nd.c
hgraph_order_si.c
hgraph_order_st.c
hmesh.c
hmesh_check.c
hmesh_hgraph.c
hmesh_induce.c
hmesh_mesh.c
hmesh_order_bl.c
hmesh_order_cp.c
hmesh_order_gr.c
hmesh_order_gp.c
hmesh_order_hd.c
hmesh_order_hf.c
hmesh_order_hx.c
hmesh_order_nd.c
hmesh_order_si.c
hmesh_order_st.c
kgraph.c
kgraph_band.c
kgraph_check.c
kgraph_map_bd.c
kgraph_map_cp.c
kgraph_map_df.c
kgraph_map_ex.c
kgraph_map_fm.c
kgraph_map_ml.c
kgraph_map_rb.c
kgraph_map_rb_map.c
kgraph_map_rb_part.c
kgraph_map_st.c
kgraph_store.c
library_arch.c
library_arch_f.c
library_arch_build.c
library_arch_build_f.c
library_common_f.c
library_geom.c
library_geom_f.c
library_graph.c
library_graph_f.c
library_graph_base.c
library_graph_base_f.c
library_graph_check.c
library_graph_check_f.c
library_graph_coarsen.c
library_graph_coarsen_f.c
library_graph_color.c
library_graph_color_f.c
library_graph_io_chac.c
library_graph_io_chac_f.c
library_graph_io_habo.c
library_graph_io_habo_f.c
library_graph_io_mmkt.c
library_graph_io_mmkt_f.c
library_graph_io_scot.c
library_graph_io_scot_f.c
library_graph_map.c
library_graph_map_f.c
library_graph_map_io.c
library_graph_map_io_f.c
library_graph_map_view.c
library_graph_map_view_f.c
library_graph_order.c
library_graph_order_f.c
library_graph_part_ovl.c
library_graph_part_ovl_f.c
library_mapping.c
library_memory.c
library_memory_f.c
library_mesh.c
library_mesh_f.c
library_mesh_graph.c
library_mesh_graph_f.c
library_mesh_io_habo.c
library_mesh_io_habo_f.c
library_mesh_io_scot.c
library_mesh_io_scot_f.c
library_mesh_order.c
library_mesh_order_f.c
library_order.c
library_parser.c
library_parser_f.c
library_random.c
library_random_f.c
library_strat.c
library_version.c
library_version_f.c
mapping.c
mapping_io.c
mesh.c
mesh_check.c
mesh_coarsen.c
mesh_graph.c
mesh_induce_sepa.c
mesh_io.c
mesh_io_habo.c
mesh_io_scot.c
order.c
order_check.c
order_io.c
parser.c
parser_ll.c
parser_yy.c
vgraph.c
vgraph_check.c
vgraph_separate_bd.c
vgraph_separate_df.c
vgraph_separate_es.c
vgraph_separate_fm.c
vgraph_separate_gg.c
vgraph_separate_gp.c
vgraph_separate_ml.c
vgraph_separate_st.c
vgraph_separate_th.c
vgraph_separate_vw.c
vgraph_separate_zr.c
vgraph_store.c
vmesh.c
vmesh_check.c
vmesh_separate_fm.c
vmesh_separate_gg.c
vmesh_separate_gr.c
vmesh_separate_ml.c
vmesh_separate_zr.c
vmesh_separate_st.c
vmesh_store.c
wgraph.c
wgraph_check.c
wgraph_part_fm.c
wgraph_part_gg.c
wgraph_part_gp.c
wgraph_part_ml.c
wgraph_part_rb.c
wgraph_part_st.c
wgraph_part_zr.c
wgraph_store.c)

SET(ERROR_LIB_SRC library_error.c) 
if (EXIT_ON_ERROR)
    SET(ERROR_LIB_SRC library_error_exit.c)
endif()

add_library(scotch ${SCOTCH_SRC} ${parser_yy} ${parser_ll} ${ERROR_LIB_SRC})
set_target_properties(scotch PROPERTIES OUTPUT_NAME scotch-${SCOTCH_VERSION})
if (NEED_HEADER_CREATION)
    add_dependencies(scotch scotch_header)
endif()
if (UNIX)
    target_link_libraries(scotch PRIVATE m)
    if (NOT APPLE)
        target_link_libraries(scotch PRIVATE rt)
    endif()
endif()
if (COMPRESSION_LIBS)
    target_link_libraries(scotch PUBLIC ${COMPRESSION_LIBS})
endif()
if (USE_THREADS)
    target_link_libraries(scotch PUBLIC ${CMAKE_THREAD_LIBS_INIT})
    target_compile_definitions(scotch PRIVATE SCOTCH_PTHREAD)
endif()
target_include_directories(scotch PRIVATE .
    PUBLIC $<BUILD_INTERFACE:${SCOTCH_INC_DIR}>
)

install(TARGETS scotch EXPORT scotch-config
    DESTINATION lib
    INCLUDES DESTINATION include
)
install(TARGETS scotch EXPORT ptscotch-config
    DESTINATION lib
    INCLUDES DESTINATION include
)
install(EXPORT scotch-config DESTINATION ${PACKAGE_CONFIG_DIR})
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/scotch-config-version.cmake COMPATIBILITY AnyNewerVersion)
CREATE_DEPENDENCIES_FILE(scotch-config)
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/scotch-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/scotch-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})


if (BUILD_PTSCOTCH)
    add_subdirectory(libptscotch)
endif()