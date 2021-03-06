
cmake_minimum_required(VERSION 2.8 )
project(MATH)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "-fPIC")
set(CMAKE_BUILD_TYPE "Debug")
set(MATH_VERSION_MAJOR 1)
set(MATH_VERSION_MINOR 5)
set(MATH_VERSION_PATCH 1)
set(MATH_VERSION ${MATH_VERSION_MAJOR}.${MATH_VERSION_MINOR}.${MATH_VERSION_PATCH})
message("当前MATH版本为:${MATH_VERSION}")

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmakemodule)

configure_file(
        "${PROJECT_SOURCE_DIR}/config/CmakeHelper.cmake.in"
        "${PROJECT_SOURCE_DIR}/cmakemodule/CmakeHelper.cmake" @ONLY
)
include(cmakemodule/CmakeHelper.cmake)

configure_file(
        "${PROJECT_SOURCE_DIR}/config/MATHConfig.h.in"
        "${PROJECT_SOURCE_DIR}/include/MATH/MATHConfig.h"
)

#以同步方式，在同一时间内执行同一条指令
option(SIMD_ENABLED "是否使用SIMD指令集进行优化" ON)
#用于共享内存并行系统的多处理器程序设计的一套指导性编译处理方案
option(OPENMP_ENABLED "是否使用OPenMP 进行并行优化" ON)

option(CUDA_ENABLED "是否使用CUDA加速" ON)

option(TESTS_ENABLED "是否编译测试二进制文件" OFF)

####################################################
######################TEST##########################
####################################################
set(TESTS_ENABLED ON)
if (TESTS_ENABLED)
    message(STATUS "允许编译测试文件")
    enable_testing()
else()
    message(STATUS "禁止编译测试文件")
endif ()

####################################################
######################CUDA##########################
####################################################
set(CUDA_MIN_VERSION "9.0")
if (CUDA_ENABLED)
    find_package(CUDA ${CUDA_MIN_VERSION} QUIET)
    if (CUDA_FOUND)
        message("CUDA已经安装")
    else(CUDA_FOUND)
        message(STATUS "未找到 CUDA")
        message(STATUS "  请安装不低于${CUDA_MIN_VERSION}的CUDA版本")
    endif (CUDA_FOUND)
endif ()

if (CUDA_FOUND)
    if (CUDA_ENABLED)
        add_definitions("-DCUDA_ENABLED")
        message(STATUS "使能CUDA支持（版本：，"
                "架构：）")
    endif ()
else()
    set(CUDA_FOUND OFF)
    message(STATUS "  禁用CUDA支持")
endif ()

####################################################
#####################BOOST##########################
####################################################

find_package(Boost REQUIRED COMPONENTS
        program_options
        filesystem
        graph
        regex
        system
        unit_test_framework
        )

if (Boost_FOUND)
    message(STATUS "  找到Boost")
else()
    message(STATUS "  未找到Boost")
    message(STATUS "  请安装Boost")
endif()

####################################################
#######################SIMD#########################
####################################################

if (SIMD_ENABLED)
    message(STATUS "使能SIMD支持")
else()
    message(STATUS "禁用SIMD支持")
endif ()

####################################################
######################OpenMP########################
####################################################

if (OPENMP_ENABLED)
    message(STATUS "使能OPENMP支持")
    add_definitions("-DOPENMP_ENABLED")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
else()
    message(STATUS "禁用OPENMP支持")
endif ()

add_subdirectory(examples)
add_subdirectory(src)
add_subdirectory(TestCase)

configure_file(
        "${PROJECT_SOURCE_DIR}/config/MATHConfig.cmake.in"
        "${PROJECT_SOURCE_DIR}/cmakemodule/MATHConfig.cmake" @ONLY
)

configure_file(
        "${PROJECT_SOURCE_DIR}/config/MATHConfigVersion.cmake.in"
        "${PROJECT_SOURCE_DIR}/cmakemodule/MATHConfigVersion.cmake" @ONLY
)

install(FILES cmakemodule/MATHConfig.cmake
        DESTINATION share/MATH)
install(FILES cmakemodule/MATHConfigVersion.cmake
        DESTINATION share/MATH)
install(DIRECTORY include/MATH/
        DESTINATION include/MATH)