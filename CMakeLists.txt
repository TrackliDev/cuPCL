cmake_minimum_required(VERSION 3.0)
project(cu_pcl)

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

# Standard dependencies used by all subpackages
add_library(cu_pcl_std_dep_lib INTERFACE)

# Standard compile setup
target_compile_definitions(cu_pcl_std_dep_lib INTERFACE -D_REENTRANT)
target_compile_options(cu_pcl_std_dep_lib INTERFACE -std=c++14 -fPIC)
target_compile_features(cu_pcl_std_dep_lib INTERFACE cxx_std_14)

# Standard include directories and libraries
target_link_directories(cu_pcl_std_dep_lib INTERFACE ${LINK_DIRS})
target_link_libraries(cu_pcl_std_dep_lib INTERFACE ${LIBRARIES})
target_include_directories(cu_pcl_std_dep_lib INTERFACE ${INCLUDE})

# Add all subdirectories
add_subdirectory(cuCluster)
add_subdirectory(cuSegmentation)
