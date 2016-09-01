#   Copyright 2010-2016 Comcast Cable Communications Management, LLC
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

cmake_minimum_required(VERSION 2.8.7)

project(parodus)

include(ExternalProject)
include(CTest)

set(INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/_install)
set(INCLUDE_DIR ${INSTALL_DIR}/include)
set(LIBRARY_DIR ${INSTALL_DIR}/lib${LIB_SUFFIX})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE")

find_package (Threads)

# base64 external dependency
ExternalProject_Add(trower-base64
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/trower-base64
    GIT_REPOSITORY https://github.com/Comcast/trower-base64.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
)
add_library(libtrower-base64 STATIC IMPORTED)
add_dependencies(libtrower-base64 trower-base64)


# nopoll external dependency
ExternalProject_Add(nopoll
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/nopoll
    GIT_REPOSITORY https://github.com/ASPLes/nopoll.git
    GIT_TAG "master"
    CONFIGURE_COMMAND COMMAND <SOURCE_DIR>/autogen.sh
    CONFIGURE_COMMAND COMMAND <SOURCE_DIR>/configure --prefix=${PREFIX}
                                                     --includedir=${INCLUDE_DIR}
                                                     --libdir=${LIBRARY_DIR}
    BUILD_IN_SOURCE 1
)
add_library(libnopoll STATIC SHARED IMPORTED)
add_dependencies(libnopoll nopoll)


# msgpack-c external dependency
ExternalProject_Add(msgpack
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/msgpack
    GIT_REPOSITORY https://github.com/msgpack/msgpack-c.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
)
add_library(libmsgpack STATIC SHARED IMPORTED)
add_dependencies(libmsgpack msgpack)

# wrp-c external dependency
ExternalProject_Add(wrp-c
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/_prefix/wrp-c
    GIT_REPOSITORY https://github.com/Comcast/wrp-c.git
    GIT_TAG "master"
    CMAKE_ARGS += -DCMAKE_INSTALL_PREFIX=${INSTALL_DIR}
                  -DMSGPACK_ENABLE_CXX=OFF
                  -DMSGPACK_BUILD_EXAMPLES=OFF
                  -DBUILD_TESTING=OFF
)
add_library(libwrp-c STATIC SHARED IMPORTED)
add_dependencies(libwrp-c wrp-c)

link_directories ( ${LIBRARY_DIR} )

add_definitions(-std=c99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wall")

add_subdirectory(src)
if (BUILD_TESTING)
    add_subdirectory(tests)
endif (BUILD_TESTING)