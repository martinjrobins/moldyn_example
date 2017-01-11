cmake_minimum_required(VERSION 2.8)
project(MoldynExample)

set(Aboria_LOG_LEVEL 1 CACHE STRING "Logging level (1 = least, 3 = most)")
add_definitions(-DABORIA_LOG_LEVEL=${Aboria_LOG_LEVEL})

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/Aboria/cmake"
                        ${CMAKE_MODULE_PATH})

list(APPEND CMAKE_CXX_FLAGS "-Wno-deprecated -std=c++11")
list(APPEND CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined")

find_package(Boost 1.50.0 COMPONENTS program_options REQUIRED)

find_package(VTK REQUIRED)
add_definitions(-DHAVE_VTK)

option(Aboria_USE_GPERFTOOLS "Use Google Profiling tools" OFF)
if (Aboria_USE_GPERFTOOLS)
    find_package(Gperftools REQUIRED)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")
    set(GPERFTOOLS_LIBRARIES "-Wl,--no-as-needed ${GPERFTOOLS_LIBRARIES} -Wl,--as-needed") 
endif()


option(Aboria_USE_THRUST "Use CUDA Thrust library" OFF)
if (Aboria_USE_THRUST)
    find_package(Thrust REQUIRED)
    find_package(CUDA REQUIRED)

    option(Aboria_THRUST_USE_ALGORITHMS "Use CUDA Thrust library algorithms" ON)
    option(Aboria_THRUST_TEST_DEVICE_VECTOR "Turn on tests using thrust::device_vector" OFF)
    if (Aboria_THRUST_USE_ALGORITHMS)
        add_definitions(-D__aboria_use_thrust_algorithms__)
    endif()
    #http://stackoverflow.com/questions/34996295/trying-to-get-cuda-7-5-to-work-with-gcc-5-x

    add_definitions(-D_MWAITXINTRIN_H_INCLUDED)
    set(
    CUDA_NVCC_FLAGS
    ${CUDA_NVCC_FLAGS};
    --keep
    #--std=c++11
    #-gencode arch=compute_20,code=sm_20
    )
endif()

find_package(Eigen3 REQUIRED)
add_definitions(-DHAVE_EIGEN)

#export compiler flags for code completion engines
set( CMAKE_EXPORT_COMPILE_COMMANDS 1 )


include_directories(Aboria/src)
include_directories(SYSTEM ${CXXTEST_INCLUDES} ${EIGEN3_INCLUDE_DIR} ${VTK_INCLUDE_DIRS} ${Boost_INCLUDE_DIR}  ${PYTHON_INCLUDE_DIRS})

add_executable(moldyn_example moldyn_example.cpp)
target_link_libraries(moldyn_example ${VTK_LIBRARIES} ${Boost_LIBRARIES} ${CUDA_LIBRARIES} ${GPERFTOOLS_LIBRARIES})

