cmake_minimum_required(VERSION 3.0)
project(your_project_name)

# Execute shell command to get CUDA version
execute_process(
    COMMAND dpkg -l | grep cuda-core | sed -e "s/ \{1,\}/ /g" | cut -d ' ' -f 3 | cut -d '.' -f 1,2 | sed -e "s/-.*//g" | sort -n | tail -n 1
    OUTPUT_VARIABLE CUDA_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Check if CUDA version is found
if (NOT CUDA_VERSION)
    message(FATAL_ERROR "Failed to find CUDA version")
endif()

# Print CUDA version
message("CUDA version: ${CUDA_VERSION}")

# Execute shell command to get target architecture
execute_process(
    COMMAND uname -m
    OUTPUT_VARIABLE TARGET_ARCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Check if target architecture is found
if (NOT TARGET_ARCH)
    message(FATAL_ERROR "Failed to find target architecture")
endif()

# Print target architecture
message("Target architecture: ${TARGET_ARCH}")

# Set CC and NVCC variables based on target architecture
set(CC "g++")
if (${TARGET_ARCH} STREQUAL "aarch64")
    set(NVCC "/usr/local/${CUDAVERSION}/bin/nvcc -m64 -ccbin ${CC}")
elseif (${TARGET_ARCH} STREQUAL "x86_64")
    set(NVCC "/usr/local/${CUDAVERSION}/bin/nvcc -m64")
else()
    message(FATAL_ERROR "Unsupported target architecture: ${TARGET_ARCH}")
endif()

# Set CUDA flags and libraries
set(CUDA_CFLAGS "-I/usr/local/${CUDAVERSION}/include")
set(CUDA_LIBS "-L/usr/local/${CUDAVERSION}/lib64 -lcudart_static -lrt -ldl -lpthread -lcudart")

# Set additional include directories
set(INCLUDE "")
list(APPEND INCLUDE ${CUDA_CFLAGS})
list(APPEND INCLUDE "-I/usr/local/include")
list(APPEND INCLUDE "-I/usr/include/eigen3/")
list(APPEND INCLUDE "-I/usr/include/pcl-1.10/")
list(APPEND INCLUDE "-I/usr/include/vtk-6.3/")

# Set additional libraries
set(LIBRARIES "")
list(APPEND LIBRARIES "-L/usr/lib")
list(APPEND LIBRARIES "-L/usr/local/lib")
list(APPEND LIBRARIES ${CUDA_LIBS})
list(APPEND LIBRARIES "-lpthread")
list(APPEND LIBRARIES "-L/usr/lib/aarch64-linux-gnu/")
list(APPEND LIBRARIES "-lboost_system")
list(APPEND LIBRARIES "-lpcl_common")
list(APPEND LIBRARIES "-lpcl_io")
list(APPEND LIBRARIES "-lpcl_recognition")
list(APPEND LIBRARIES "-lpcl_features")
list(APPEND LIBRARIES "-lpcl_sample_consensus")
list(APPEND LIBRARIES "-lpcl_octree")
list(APPEND LIBRARIES "-lpcl_search")
list(APPEND LIBRARIES "-lpcl_filters")
list(APPEND LIBRARIES "-lpcl_kdtree")
list(APPEND LIBRARIES "-lpcl_segmentation")
list(APPEND LIBRARIES "-lpcl_visualization")

# Set object directory
set(OBJ_DIR "obj")

# Collect source files
file(GLOB CPP_FILES "*.cpp")
file(GLOB CU_FILES "*.cu")
file(GLOB LIBRARY_FILES "./lib/*.so")

# Create object directory
file(MAKE_DIRECTORY ${OBJ_DIR})

# Compile C++ files
foreach(CPP_FILE ${CPP_FILES})
    get_filename_component(FILE_NAME ${CPP_FILE} NAME_WE)
    set(OBJ_FILE "${OBJ_DIR}/${FILE_NAME}.o")
    add_custom_command(
        OUTPUT ${OBJ_FILE}
        COMMAND ${CC} ${INCLUDE} ${CCFLAGS} ${CXXFLAGS} -fPIC -o ${OBJ_FILE} -c ${CPP_FILE}
        DEPENDS ${CPP_FILE}
    )
    list(APPEND OBJ_FILES_CPP ${OBJ_FILE})
endforeach()

# Compile CUDA files
foreach(CU_FILE ${CU_FILES})
    get_filename_component(FILE_NAME ${CU_FILE} NAME_WE)
    set(OBJ_FILE "${OBJ_DIR}/${FILE_NAME}.o")
    add_custom_command(
        OUTPUT ${OBJ_FILE}
        COMMAND mkdir -p ${OBJ_DIR}
        COMMAND ${NVCC} ${INCLUDE} ${CXXFLAGS} ${CCFLAGS} ${NVCCFLAGS} -lineinfo ${GENCODE_FLAGS} -Xcompiler -fPIC -c ${CU_FILE} -o ${OBJ_FILE}
        DEPENDS ${CU_FILE}
    )
    list(APPEND OBJ_FILES_CU ${OBJ_FILE})
endforeach()

# Build target
add_executable(${TARGET} ${OBJ_FILES_CPP} ${OBJ_FILES_CU})
target_link_libraries(${TARGET} PRIVATE ${LIBRARIES} ${LIBRARY_FILES})

# Clean target
add_custom_target(clean-all COMMAND rm -rfv ${OBJ_DIR} ${TARGET})
