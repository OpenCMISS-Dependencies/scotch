if (MPI_C_COMPILER)
    SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
endif()

# Scotch tests
if (USE_THREADS)
    add_executable(test_common_thread test_common_thread.c)
    target_link_libraries(test_common_thread scotch)
    add_test(test_common_thread test_common_thread)
endif()

add_executable(test_strat_seq test_strat_seq.c)
target_link_libraries(test_strat_seq scotch)
add_test(test_strat_seq test_strat_seq)

add_executable(test_scotch_graph_color test_scotch_graph_color.c)
target_link_libraries(test_scotch_graph_color scotch)
add_test(NAME test_scotch_graph_color_direct
    COMMAND test_scotch_graph_color bump.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
    
add_executable(test_scotch_graph_map test_scotch_graph_map.c)
target_link_libraries(test_scotch_graph_map PRIVATE scotch)
if (UNIX)
    target_link_libraries(test_scotch_graph_map PRIVATE m)
endif()
add_test(NAME test_scotch_graph_map4x4
    COMMAND test_scotch_graph_map m4x4.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
add_test(NAME test_scotch_graph_map4x4b1
    COMMAND test_scotch_graph_map m4x4_b1.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)       
add_test(NAME test_scotch_graph_map16x16
    COMMAND test_scotch_graph_map m16x16.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)   
add_test(NAME test_scotch_graph_map16x16_b1
    COMMAND test_scotch_graph_map m16x16_b1.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)   
    
add_executable(test_scotch_graph_order test_scotch_graph_order.c)
target_link_libraries(test_scotch_graph_order scotch)
add_test(NAME test_scotch_graph_order
    COMMAND test_scotch_graph_order bump.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
add_test(NAME test_scotch_graph_order_b1 
    COMMAND test_scotch_graph_order bump_b1.grf
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
    
add_executable(test_scotch_graph_part_ovl test_scotch_graph_part_ovl.c)
target_link_libraries(test_scotch_graph_part_ovl scotch)
add_test(NAME test_scotch_graph_part_ovl
    COMMAND test_scotch_graph_part_ovl 4 m16x16.grf /dev/null
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
add_test(NAME test_scotch_graph_part_ovl_b1 
    COMMAND test_scotch_graph_part_ovl 4 m16x16_b1.grf /dev/null
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)
    
add_test(NAME check_prog_gpart_remap
    COMMAND gpart 32 bump_imbal_32.grf /dev/null -robump_old.map -vmt
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data)    

# PT-Scotch tests
if (BUILD_PTSCOTCH)
    add_executable(test_strat_par test_strat_par.c)
    target_link_libraries(test_strat_par ptscotch)
    add_test(NAME test_strat_par 
        COMMAND ${MPIEXEC} -n 4 $<TARGET_FILE:test_strat_par>)
    
    # The following binaries build in debug/interactive mode, so automated testing is not possible
    # straightaway.
    # Will build them anyways to have at least that tested
    add_executable(test_scotch_dgraph_check test_scotch_dgraph_check.c)
    target_link_libraries(test_scotch_dgraph_check ptscotch)
    add_executable(test_scotch_dgraph_band test_scotch_dgraph_band.c)
    target_link_libraries(test_scotch_dgraph_band ptscotch)
    add_executable(test_scotch_dgraph_coarsen test_scotch_dgraph_coarsen.c)
    target_link_libraries(test_scotch_dgraph_coarsen ptscotch)
    add_executable(test_scotch_dgraph_grow test_scotch_dgraph_grow.c)
    target_link_libraries(test_scotch_dgraph_grow ptscotch)
    add_executable(test_scotch_dgraph_redist test_scotch_dgraph_redist.c)
    target_link_libraries(test_scotch_dgraph_redist ptscotch)
endif()