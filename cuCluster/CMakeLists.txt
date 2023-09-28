cmake_minimum_required(VERSION 3.0)
project(your_project_name)

# Execute shell command to get CUDA version
# execute_process(
#     COMMAND "dpkg -l | grep cuda-core | sed -e \"s/ \\{1,\\}/ /g\" | cut -d ' ' -f 3 | cut -d '.' -f 1,2 | sed -e \"s/-.*//g\" | sort -n | tail -n 1"
#     OUTPUT_VARIABLE CUDAVERSION
#     OUTPUT_STRIP_TRAILING_WHITESPACE
# )
set(CUDAVERSION cuda)

# Check if CUDA version is found
if (NOT CUDAVERSION)
    message(FATAL_ERROR "Failed to find CUDA version")
endif()

# Print CUDA version
message("CUDA version: ${CUDAVERSION}")

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

set(CCFLAGS "-D_REENTRANT")
set(CXXFLAGS "-std=c++14")

set(NVCCFLAGS "")
# set(NVCCFLAGS "-G --ptxas-options=-v")

# Execute shell command to get CUDA_VERSION from cuda.h file
set(ASASA "cat /usr/local/${CUDAVERSION}/targets/${TARGET_ARCH}-linux/include/cuda.h | grep \"define CUDA_VERSION\" | awk '{print \$3}'")
message("ASASA: ${ASASA}")
execute_process(
    COMMAND "cat /usr/local/${CUDAVERSION}/targets/${TARGET_ARCH}-linux/include/cuda.h | grep \"define CUDA_VERSION\" | awk '{print \$3}'"
    OUTPUT_VARIABLE CUDA_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Print later CUDA version
message("Later CUDA version: ${CUDA_VERSION}")

# Set SMS variable based on CUDA_VERSION and target architecture
set(SMS "")
if (${CUDA_VERSION} EQUAL 8000)
  set(SMS_VOLTA "")
else()
  if (NOT ${TARGET_ARCH} STREQUAL "ppc64le")
    if (${CUDA_VERSION} EQUAL 9000)
      set(SMS_VOLTA "70")
    else()
      set(SMS_VOLTA "70 72")
    endif()
  else()
    set(SMS_VOLTA "70")
  endif()
endif()

if (${TARGET_ARCH} STREQUAL "aarch64")
    if (${CUDA_VERSION} EQUAL 9000)
      set(SMS_VOLTA "62 70")
    endif()
endif()

if (${CUDA_VERSION} EQUAL 10020)
set(SMS_TURING "75")
endif()

if (${CUDA_VERSION} EQUAL 11040)
set(SMS_AMPERE "87")
endif()

# Gencode arguments
set(SMS "30 35 37 50 52 53 60 61 62 ${SMS_VOLTA} ${SMS_TURING} ${SMS_AMPERE}")
message("SMS: ${SMS}")

set(GENCODE_FLAGS "")
foreach(sm ${SMS})
    list(APPEND GENCODE_FLAGS "-gencode arch=compute_${sm},code=sm_${sm}")
endforeach()

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
