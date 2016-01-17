project(PTSCOTCH VERSION ${SCOTCH_VERSION} LANGUAGES C)

if (NEED_HEADER_CREATION)
    ## ptdummysizes
    add_executable(ptdummysizes ../dummysizes.c)
    target_include_directories(ptdummysizes 
        PRIVATE .)
    target_compile_definitions(ptdummysizes
        PRIVATE SCOTCH_VERSION=${SCOTCH_VERSION_MAJOR}
        SCOTCH_RELEASE=${SCOTCH_VERSION_MINOR}
        SCOTCH_PATCHLEVEL=${SCOTCH_VERSION_PATCH}
    )
    add_custom_target(ptscotch_header
        COMMAND ptdummysizes 
        ${CMAKE_CURRENT_SOURCE_DIR}/../library_pt.h ptscotch.h
        COMMAND ${CMAKE_COMMAND} -E copy ptscotch.h ${SCOTCH_INC_DIR}
        COMMAND ptdummysizes 
        ${CMAKE_CURRENT_SOURCE_DIR}/../library_pt_f.h ptscotchf.h
        COMMAND ${CMAKE_COMMAND} -E copy ptscotchf.h ${SCOTCH_INC_DIR})    
endif()

SET(PTSCOTCH_SRC bdgraph.c
bdgraph_bipart_bd.c
bdgraph_bipart_df.c
bdgraph_bipart_ex.c
bdgraph_bipart_ml.c
bdgraph_bipart_sq.c
bdgraph_bipart_st.c
bdgraph_bipart_zr.c
bdgraph_check.c
bdgraph_gather_all.c
bdgraph_store.c
comm.c
dgraph.c
dgraph_allreduce.c
dgraph_band.c
dgraph_build.c
dgraph_build_grid3d.c
dgraph_build_hcub.c
dgraph_check.c
dgraph_coarsen.c
dgraph_fold.c
dgraph_fold_comm.c
dgraph_fold_dup.c
dgraph_gather.c
dgraph_gather_all.c
dgraph_ghst.c
dgraph_halo.c
dgraph_induce.c
dgraph_io_load.c
dgraph_io_save.c
dgraph_match.c
dgraph_match_sync_coll.c
dgraph_match_sync_ptop.c
dgraph_match_check.c
dgraph_redist.c
dgraph_scatter.c
dgraph_view.c
dmapping.c
dmapping_io.c
dorder.c
dorder_gather.c
dorder_io.c
dorder_io_block.c
dorder_io_tree.c
dorder_perm.c
dorder_tree_dist.c
hdgraph.c
hdgraph_check.c
hdgraph_fold.c
hdgraph_gather.c
hdgraph_induce.c
hdgraph_order_nd.c
hdgraph_order_si.c
hdgraph_order_sq.c
hdgraph_order_st.c
kdgraph.c
kdgraph_gather.c
kdgraph_map_rb.c
kdgraph_map_rb_map.c
kdgraph_map_rb_part.c
kdgraph_map_st.c
library_dgraph.c
library_dgraph_f.c
library_dgraph_band.c
library_dgraph_band_f.c
library_dgraph_build.c
library_dgraph_build_f.c
library_dgraph_build_grid3d.c
library_dgraph_build_grid3d_f.c
library_dgraph_check.c
library_dgraph_check_f.c
library_dgraph_coarsen.c
library_dgraph_coarsen_f.c
library_dgraph_gather.c
library_dgraph_gather_f.c
library_dgraph_grow.c
library_dgraph_halo.c
library_dgraph_halo_f.c
library_dgraph_induce.c
library_dgraph_induce_f.c
library_dgraph_io_load.c
library_dgraph_io_load_f.c
library_dgraph_io_save.c
library_dgraph_io_save_f.c
library_dgraph_map.c
library_dgraph_map_f.c
library_dgraph_map_view.c
library_dgraph_map_view_f.c
library_dgraph_order.c
library_dgraph_order_f.c
library_dgraph_order_gather.c
library_dgraph_order_gather_f.c
library_dgraph_order_io.c
library_dgraph_order_io_f.c
library_dgraph_order_io_block.c
library_dgraph_order_io_block_f.c
library_dgraph_order_perm.c
library_dgraph_order_perm_f.c
library_dgraph_order_tree_dist.c
library_dgraph_order_tree_dist_f.c
library_dgraph_redist.c
library_dgraph_redist_f.c
library_dgraph_scatter.c
library_dgraph_scatter_f.c
library_dgraph_stat.c
library_dgraph_stat_f.c
library_dmapping.c
library_dorder.c
vdgraph.c
vdgraph_check.c
vdgraph_gather_all.c
vdgraph_separate_bd.c
vdgraph_separate_df.c
vdgraph_separate_ml.c
vdgraph_separate_sq.c
vdgraph_separate_st.c
vdgraph_separate_zr.c
vdgraph_store.c
)
# Add one or the other error library
if (EXIT_ON_ERROR)
    list(APPEND PTSCOTCH_SRC library_error_exit.c) 
else()
    list(APPEND PTSCOTCH_SRC library_error.c)
endif()
ADD_PREFIX(../ PTSCOTCH_SRC)
    
add_library(ptscotch ${PTSCOTCH_SRC})
set_target_properties(ptscotch PROPERTIES OUTPUT_NAME ptscotch-${PTSCOTCH_VERSION})
if (NEED_HEADER_CREATION)
    add_dependencies(ptscotch ptscotch_header)
endif()
target_compile_definitions(ptscotch PRIVATE SCOTCH_PTSCOTCH)
target_link_libraries(ptscotch PUBLIC scotch ${MPI_C_LIBRARIES})
if (UNIX AND NOT APPLE)
    target_link_libraries(ptscotch PRIVATE rt)
endif()
if (COMPRESSION_LIBS)
    target_link_libraries(ptscotch PUBLIC ${COMPRESSION_LIBS})
endif()
if (USE_THREADS)
    target_link_libraries(ptscotch PUBLIC ${CMAKE_THREAD_LIBS_INIT})
endif()
# Add mpi interface target for non-wrapper cases (see findmpi.cmake)
if (NOT CMAKE_C_COMPILER STREQUAL MPI_C_COMPILER)
    target_link_libraries(ptscotch PUBLIC $<BUILD_INTERFACE:mpi>)
endif()
target_include_directories(ptscotch PRIVATE ..
    PUBLIC $<BUILD_INTERFACE:${SCOTCH_INC_DIR}>
)
    
install(TARGETS ptscotch EXPORT ptscotch-config
    DESTINATION lib
    INCLUDES DESTINATION include)
install(EXPORT ptscotch-config DESTINATION ${PACKAGE_CONFIG_DIR})
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/ptscotch-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
CREATE_DEPENDENCIES_FILE(ptscotch-config)
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/ptscotch-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/ptscotch-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})
