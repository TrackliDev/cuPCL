cmake_minimum_required(VERSION 3.0)
project(cu_cluster_example)

# Set CUDA flags and libraries
set(CUDA_CFLAGS "/usr/local/cuda/include")
set(CUDA_LIBS "cudart_static" 
              "rt" 
              "dl" 
              "pthread" 
              "cudart")

# Set additional include directories
set(INCLUDE "")
list(APPEND INCLUDE ${CUDA_CFLAGS})
list(APPEND INCLUDE "/usr/local/include")
list(APPEND INCLUDE "/usr/include/eigen3/")
list(APPEND INCLUDE "/usr/include/pcl-1.10/")
list(APPEND INCLUDE "/usr/include/vtk-6.3/")
list(APPEND INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/lib/")

# Set additional libraries
set(LIBRARIES "")
list(APPEND LIBRARIES ${CUDA_LIBS})
list(APPEND LIBRARIES "pthread")
list(APPEND LIBRARIES "boost_system")
list(APPEND LIBRARIES "pcl_common")
list(APPEND LIBRARIES "pcl_io")
list(APPEND LIBRARIES "pcl_recognition")
list(APPEND LIBRARIES "pcl_features")
list(APPEND LIBRARIES "pcl_sample_consensus")
list(APPEND LIBRARIES "pcl_octree")
list(APPEND LIBRARIES "pcl_search")
list(APPEND LIBRARIES "pcl_filters")
list(APPEND LIBRARIES "pcl_kdtree")
list(APPEND LIBRARIES "pcl_segmentation")
list(APPEND LIBRARIES "pcl_visualization") 
list(APPEND LIBRARIES "${CMAKE_CURRENT_SOURCE_DIR}/lib/libcudacluster.so")

# Set link directories
set(LINK_DIRS "")
list(APPEND LINK_DIRS "/usr/lib")
list(APPEND LINK_DIRS "/usr/local/lib")
list(APPEND LINK_DIRS "/usr/local/cuda/lib64")
# list(APPEND LINK_DIRS "/usr/lib/aarch64-linux-gnu/")

# Build library
add_library(cu_cluster INTERFACE)

# Compile setup
target_compile_definitions(cu_cluster INTERFACE -D_REENTRANT)
target_compile_options(cu_cluster INTERFACE -std=c++14 -fPIC)
target_compile_features(cu_cluster INTERFACE cxx_std_14)

# Includes and libraries
target_link_directories(cu_cluster INTERFACE ${LINK_DIRS})
target_link_libraries(cu_cluster INTERFACE ${LIBRARIES})
target_include_directories(cu_cluster INTERFACE ${INCLUDE})

# Build demonstration target
add_executable(demo "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
target_link_libraries(demo PRIVATE cu_cluster)

# Copy .pcd file to build directory
set(PCD_FILE "${CMAKE_CURRENT_SOURCE_DIR}/sample.pcd")
set(PCD_DEST "${CMAKE_CURRENT_BINARY_DIR}/sample.pcd")
add_custom_command(
    OUTPUT ${PCD_DEST}
    COMMAND ${CMAKE_COMMAND} -E copy ${PCD_FILE} ${PCD_DEST}
    DEPENDS ${PCD_FILE}
)
add_custom_target(copy_pcd_file ALL DEPENDS ${PCD_DEST})
add_dependencies(demo copy_pcd_file)
