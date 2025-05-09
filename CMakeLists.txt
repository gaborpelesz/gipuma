cmake_minimum_required (VERSION 3.9)
project (gipuma)
# Enable C++20 globally
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Support IDEs: https://cliutils.gitlab.io/modern-cmake/chapters/features/ides.html
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "cmake-default-targets")

find_package(CUDA REQUIRED ) # For Cuda Managed Memory and c++11
find_package(Eigen3 REQUIRED )
find_package(OpenCV REQUIRED )
if(NOT DEFINED CMAKE_CUDA_STANDARD)
    set(CMAKE_CUDA_STANDARD 20)
    set(CMAKE_CUDA_STANDARD_REQUIRED ON)
endif()

include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(.)

# from https://en.wikipedia.org/wiki/CUDA#GPUs_supported
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 --use_fast_math --ptxas-options=-v --compiler-options -Wall)
#set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-O3 --use_fast_math --ptxas-options=-v -std=c++11 --compiler-options -Wall -gencode arch=compute_52,code=sm_52)

if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-Wall)
    add_definitions(-Wextra)
    add_definitions(-pedantic)
    add_definitions(-Wno-unused-function)
    add_definitions(-Wno-switch)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -Ofast -ffast-math -march=native") # extend release-profile with fast-math
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()


# For compilation ...
# Specify target & source files to compile it from
cuda_add_executable(
    gipuma
    cameraGeometryUtils.h
    vector_operations.h
    camera.h
    globalstate.h
    algorithmparameters.h
    cameraparameters.h
    linestate.h
    groundTruthUtils.h
    displayUtils.h
    fileIoUtils.h
    main.h
    gipuma.cu
    main.cpp
    )

# For linking ...
# Specify target & libraries to link it with
target_link_libraries(gipuma
    ${OpenCV_LIBS}
	OpenMP::OpenMP_CXX
    )

include(FeatureSummary)
feature_summary(WHAT ALL)

