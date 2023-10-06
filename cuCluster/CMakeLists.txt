cmake_minimum_required(VERSION 3.0)
project(your_project_name)

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

# Set link directories
set(LINK_DIRS "")
list(APPEND LINK_DIRS "/usr/lib")
list(APPEND LINK_DIRS "/usr/local/lib")
list(APPEND LINK_DIRS "/usr/local/cuda/lib64")
# list(APPEND LINK_DIRS "/usr/lib/aarch64-linux-gnu/")

# Collect source files
file(GLOB LIBRARY_FILES "./lib/*.so")

# Build target
add_executable(demo "./main.cpp"
                    "./lib/cudaCluster.h")
# Compile setup
target_compile_definitions(demo PRIVATE -D_REENTRANT)
target_compile_options(demo PRIVATE -std=c++14 -fPIC)
target_compile_features(demo PUBLIC cxx_std_14)

# Includes and libraries
target_link_directories(demo PRIVATE ${LINK_DIRS})
target_link_libraries(demo PRIVATE ${LIBRARIES} ${LIBRARY_FILES})
target_include_directories(demo PRIVATE ${INCLUDE})

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

