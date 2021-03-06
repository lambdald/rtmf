# Copyright (c) 2015, Julian Straub <jstraub@csail.mit.edu> Licensed
# under the MIT license. See the license file LICENSE.

project(rtmf)
cmake_minimum_required(VERSION 2.8.1)

set(POD_NAME rtmf)
include(cmake/pods.cmake)

if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


set(CMAKE_COLOR_MAKEFILE ON)
IF(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug) # Release; debug; ...
  set(CMAKE_BUILD_TYPE Release) # Release; debug; ...
ENDIF()

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "default install path" FORCE )
endif()


# turn different compile modes on and off
set(LCM OFF)
set(DRC OFF)
set(OPENNI ON)

set(OPENNI2 OFF)  # untested hence off
set(REALSENSE ON)

find_package (Eigen3 REQUIRED)
find_package (VTK REQUIRED)
find_package (CUDA REQUIRED)
FIND_PACKAGE (Boost 1.54 REQUIRED)
#FIND_PACKAGE (OpenMP)

if (OPENMP_FOUND)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# add flags depending on build type
set(CMAKE_CXX_FLAGS_RELEASE "-g -Wall -Wno-deprecated -O2 -finline-functions -DNDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "-g -Wno-deprecated")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-gencode arch=compute_20,code=sm_20 -gencode arch=compute_30,code=sm_30 -gencode arch=compute_35,code=\"sm_35,compute_35\")

include_directories(
  include
  ${EIGEN3_INCLUDE_DIR}
  ${Boost_INCLUDE_DIRS}
  ${CUDA_TOOLKIT_INCLUDE}
  ${VTK_INCLUDE_DIRS}
)

link_directories(
  ${Boost_LIBRARY_DIRS}
)

if(NOT DRC)
  message("-- compiling for OpenCV")
  find_package (OpenCV REQUIRED)
  find_package (PCL REQUIRED)
  message(${OpenCV_LIBS})
  message(${OpenCV_INCLUDE_DIRS})
  message(${PCL_INCLUDE_DIRS})

  include_directories(
    ${PCL_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
    )

  link_directories(
    ${PCL_LIBRARY_DIRS}
    ${OpenCV_LIBRARIES} 
    )

endif()

if(OPENNI)
  message("-- compiling for OpenNI")
  cuda_add_executable(realtimeMF ./src/realtimeMF.cpp
    )
  target_link_libraries (realtimeMF
    jsCore manifold mmf cudaPcl
    boost_system boost_thread boost_program_options
    pcl_common pcl_io pcl_visualization pcl_filters pcl_features
    pcl_search
    OpenNI XnVNite_1_5_2 
     vtkCommon vtkFiltering vtkRendering vtkGraphics 
    opencv_core opencv_highgui opencv_imgproc
    opencv_contrib
    )
  pods_install_executables(realtimeMF)
  pods_install_headers(
    ./include/rtmf/rtmf.hpp
    ./include/rtmf/realtimeMF_openni.hpp
    DESTINATION rtmf)
endif()

if(OPENNI2)
  message("-- compiling for OpenNi2")
  cuda_add_executable(realtimeMF_openni2 ./src/realtimeMF_openni2.cpp
    )
  target_link_libraries (realtimeMF_openni2
    jsCore manifold mmf cudaPcl
    rgbdGrabber
    boost_system boost_thread boost_program_options
    pcl_common pcl_io pcl_visualization pcl_filters pcl_features
    pcl_search
    OpenNI XnVNite_1_5_2 
     vtkCommon vtkFiltering vtkRendering vtkGraphics 
    opencv_core opencv_highgui opencv_imgproc
    opencv_contrib
    )
  pods_install_executables(realtimeMF_openni2)
  pods_install_headers(
    ./include/rtmf/rtmf.hpp
    ./include/rtmf/realtimeMF_openni2.hpp
    DESTINATION rtmf)
endif()

if(REALSENSE)
  message("-- compiling for RealSense")
  cuda_add_executable(realtimeMF_realsense ./src/realtimeMF_realsense.cpp
    )
  target_link_libraries (realtimeMF_realsense
    jsCore manifold mmf cudaPcl
    rgbdGrabber
    boost_system boost_thread boost_program_options
    pcl_common pcl_io pcl_visualization pcl_filters pcl_features
    pcl_search
    OpenNI XnVNite_1_5_2 
     vtkCommon vtkFiltering vtkRendering vtkGraphics 
    opencv_core opencv_highgui opencv_imgproc
    opencv_contrib
    )
  pods_install_executables(realtimeMF_realsense)
  pods_install_headers(
    ./include/rtmf/rtmf.hpp
    ./include/rtmf/realtimeMF_realsense.hpp
    DESTINATION rtmf)
endif()

if(LCM OR DRC)
  message("-- compiling for LCM or DRC")
  cuda_add_executable(realtimeMF_lcm ./src/realtimeMF_lcm.cpp
    ./src/jpeg-utils-ijg.c
    )
  target_link_libraries (realtimeMF_lcm
    jsCore manifold cudaPcl mmf
  multisense_image_utils
    boost_system boost_thread boost_program_options
    pcl_common pcl_io pcl_visualization pcl_filters pcl_features
    pcl_search
    opencv_core opencv_highgui opencv_imgproc
    opencv_contrib
    opencv_calib3d
    jpeg z png
    )

  pods_use_pkg_config_packages(realtimeMF_lcm
    lcm
    )

  pods_install_executables(realtimeMF_lcm)
  pods_install_headers(
    ./include/rtmf/realtimeMF_lcm.hpp
    ./include/rtmf/rtmf.hpp
    ./include/rtmf/realtimeMF_openni.hpp
    ./include/rtmf/jpeg-utils-ijg.h
  DESTINATION rtmf)
endif()
