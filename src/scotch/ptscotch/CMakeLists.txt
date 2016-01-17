macro(ADD_EXEC_PT NAME)
    add_executable(${NAME} ../${NAME}.c)
    target_link_libraries(${NAME} PRIVATE ptscotch ${MPI_C_LIBRARIES})
    target_include_directories(${NAME} 
        PRIVATE ../../libscotch ../ 
            ${SCOTCH_INC_DIR} ${MPI_C_INCLUDE_PATH})
endmacro()

ADD_EXEC_PT(dggath)
ADD_EXEC_PT(dgmap)
ADD_EXEC_PT(dgord)
add_executable(dgpart ../dgmap.c)
target_link_libraries(dgpart PRIVATE ptscotch ${MPI_C_LIBRARIES})
target_include_directories(dgpart 
    PRIVATE ../../libscotch ../ ${SCOTCH_INC_DIR})
target_compile_definitions(dgpart PRIVATE SCOTCH_COMPILE_PART)
ADD_EXEC_PT(dgscat)
ADD_EXEC_PT(dgtst)