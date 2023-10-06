# Build CUDA cluster library
add_library(cu_cluster INTERFACE)

# Include directories and libraries
target_link_libraries(cu_cluster INTERFACE 
                      cu_pcl_std_dep_lib
                      "${CMAKE_CURRENT_SOURCE_DIR}/lib/libcudacluster.so")
target_include_directories(cu_cluster INTERFACE 
                           "${CMAKE_CURRENT_SOURCE_DIR}/lib/")

# Demonstration target
add_executable(cu_clu_demo "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
target_link_libraries(cu_clu_demo PRIVATE cu_cluster)

# Copy .pcd file to build directory
set(CU_CLU_PCD_FILE "${CMAKE_CURRENT_SOURCE_DIR}/sample.pcd")
set(CU_CLU_PCD_DEST "${CMAKE_CURRENT_BINARY_DIR}/sample.pcd")
add_custom_command(
    OUTPUT ${CU_CLU_PCD_DEST}
    COMMAND ${CMAKE_COMMAND} -E copy ${CU_CLU_PCD_FILE} ${CU_CLU_PCD_DEST}
    DEPENDS ${CU_CLU_PCD_FILE}
)
add_custom_target(cu_clu_copy_pcd_file ALL DEPENDS ${CU_CLU_PCD_DEST})
add_dependencies(cu_clu_demo cu_clu_copy_pcd_file)
