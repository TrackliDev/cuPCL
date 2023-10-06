# Build CUDA segmentation library
add_library(cu_segmentation INTERFACE)

# Include directories and libraries
target_link_libraries(cu_segmentation INTERFACE 
                      cu_pcl_std_dep_lib
                      "${CMAKE_CURRENT_SOURCE_DIR}/lib/libcudasegmentation.so")
target_include_directories(cu_segmentation INTERFACE 
                           "${CMAKE_CURRENT_SOURCE_DIR}/lib/")

# Demonstration target
add_executable(cu_seg_demo "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
target_link_libraries(cu_seg_demo PRIVATE cu_segmentation)

# Copy .pcd file to build directory
set(CU_SEG_PCD_FILE "${CMAKE_CURRENT_SOURCE_DIR}/sample.pcd")
set(CU_SEG_PCD_DEST "${CMAKE_CURRENT_BINARY_DIR}/sample.pcd")
add_custom_command(
    OUTPUT ${CU_SEG_PCD_DEST}
    COMMAND ${CMAKE_COMMAND} -E copy ${CU_SEG_PCD_FILE} ${CU_SEG_PCD_DEST}
    DEPENDS ${CU_SEG_PCD_FILE}
)
add_custom_target(cu_seg_copy_pcd_file ALL DEPENDS ${CU_SEG_PCD_DEST})
add_dependencies(cu_seg_demo cu_seg_copy_pcd_file)
