## Copyright 2016 Etix Labs
##
## Licensed under the Apache License, Version 2.0 (the "License");
## you may not use this file except in compliance with the License.
## You may obtain a copy of the License at
##
##     http://www.apache.org/licenses/LICENSE-2.0
##
## Unless required by applicable law or agreed to in writing, software
## distributed under the License is distributed on an "AS IS" BASIS,
## WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
## See the License for the specific language governing permissions and
## limitations under the License.

# min config
cmake_minimum_required(VERSION 2.8.1)

cmake_policy(SET CMP0042 NEW)

set(PROJECT_NAME camera_emulation_server)

project(${PROJECT_NAME})

# Cmake properties
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_INCLUDE_DIRECTORIES_PROJECT_BEFORE ON)
set(PREDEFINED_TARGETS_FOLDER "CMake")
set(CMAKE_VERBOSE_MAKEFILE true)

# include find package and try to find required packages
include(FindPkgConfig)
pkg_check_modules(GLIB2 glib-2.0)

# Check build type
if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "No CMAKE_BUILD_TYPE specified, default to Debug")
    set(CMAKE_BUILD_TYPE "Debug")
endif()
message(STATUS "Building ${CMAKE_BUILD_TYPE} ERTSP")

# compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14") #enable C++14

# set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -W -Wall -Wextra -Wno-unused-function") # extra warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color") #enable error coloration on gcc

# release specific flags
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2") #enable error coloration on gcc

#debug specific flags
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fprofile-arcs -ftest-coverage")

# find gstreamer 1.x libraries
pkg_search_module(GSTREAMER REQUIRED gstreamer-1.0)
find_library(LIB_GSTREAMER NAMES ${GSTREAMER_LIBRARIES} HINTS ${GSTREAMER_LIBRARY_DIRS})

FIND_LIBRARY(GST_RTSP_SERVER_LIBRARY_DIRS NAMES gstrtspserver-1.0 PATHS "/usr/lib/x86_64-linux-gnu/gstreamer-1.0/")

FIND_PATH(GST_RTSP_SERVER_INCLUDE_DIRS gst/rtsp-server/rtsp-server.h PATHS "/usr/include/gstreamer-1.0")

include_directories (
        "${PROJECT_SOURCE_DIR}/include"
        "${GSTREAMER_INCLUDE_DIRS}"
        "${GST_RTSP_SERVER_INCLUDE_DIRS}"
)

link_directories (
  "${GSTREAMER_LIBRARY_DIRS}"
  "${GST_RTSP_SERVER_LIBRARY_DIRS}"
)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "Clang")
    # search special osx gstreamer libs
    pkg_search_module(GSTREAMER_APP REQUIRED gstreamer-app-1.0)
    find_library(LIB_GSTREAMER NAMES ${GSTREAMER_APP_LIBRARIES} HINTS ${GSTREAMER_APP_LIBRARY_DIRS})

    include_directories (${GSTREAMER_APP_INCLUDE_DIRS})

    link_directories (${GSTREAMER_APP_LIBRARY_DIRS})
endif()

add_executable(camera_emulation_server "src/server.cpp")

target_link_libraries (camera_emulation_server ${GSTREAMER_LIBRARIES} ${GST_RTSP_SERVER_LIBRARY_DIRS})
