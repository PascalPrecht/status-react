
# Copyright (C) 2016, Canonical Ltd.
# All rights reserved.

# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree. An additional grant
# of patent rights can be found in the PATENTS file in the same directory.

cmake_minimum_required(VERSION 3.12.1)

set(APP_NAME Status)
set(JS_APP_NAME StatusIm)
set(REACT_BUILD_STATIC_LIB ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUNICODE -std=c++11")
if(STATUS_NO_LOGGING)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DSTATUS_NO_LOGGING")
endif()
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DUNICODE -std=c11")
set(CMAKE_INSTALL_PREFIX bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_PREFIX}" CACHE PATH "Where to place compiled executables.")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules/")

project(${APP_NAME} C CXX)
set(${APP_NAME}_BINARY_DIR ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

set(USED_QT_MODULES Core Concurrent Qml Quick WebSockets Svg)
include(QtConfiguration)

set(ICON_PNG_RESOURCE_PATH "${CMAKE_SOURCE_DIR}/../deployment/status-icon.png")
if (WIN32)
  set(ICO_RESOURCE_PATH "${CMAKE_SOURCE_DIR}/../deployment/windows/status.ico")
  set(APPLICATION_MAIN_RC_PATH "${CMAKE_SOURCE_DIR}/../deployment/windows/Status.rc")

  configure_file(
    ${CMAKE_SOURCE_DIR}/../deployment/windows/Status.rc.in
    ${APPLICATION_MAIN_RC_PATH}
  )
endif()

message(STATUS "EXTERNAL_MODULES_DIR: ${EXTERNAL_MODULES_DIR}")

string(REGEX MATCH "BUILD_FOR_BUNDLE" BUILD_FOR_BUNDLE "${CMAKE_CXX_FLAGS}")

# It is important to distinguish between directory hierarchies
# in release and dev builds. Release contains .env
# in the root dir of the build, while dev build's current dir
# is status-react/desktop/bin, hence the need
# to jump two levels up.
# This is required by react-native-config library which references ENVFILE
# from its own CMakeLists.txt file, so it will read the value set here
if(${BUILD_FOR_BUNDLE} MATCHES "BUILD_FOR_BUNDLE")
  set(ENVFILE "/.env")
else()
  set(ENVFILE "/../../.env")
endif()

foreach(external_module ${EXTERNAL_MODULES_DIR})
  message(STATUS "external_module: ${external_module}")
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../${external_module} ${CMAKE_CURRENT_BINARY_DIR}/${external_module})
endforeach(external_module)

add_subdirectory(reportApp)

# Import any new Qt modules that have been added by external dependencies
import_qt_modules()

# APPLICATION_MAIN_CPP_PATH contains absolute path to generated template copy of main.cpp for application executable
get_filename_component(APPLICATION_MAIN_CPP_PATH main.cpp ABSOLUTE)

set(REACTQT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../node_modules/react-native/React)
include_directories(${REACTQT_DIR}/Layout)
add_subdirectory(${REACTQT_DIR}Qt/runtime/src ${CMAKE_CURRENT_BINARY_DIR}/lib)
add_subdirectory(${REACTQT_DIR}Qt/application/src ${CMAKE_CURRENT_BINARY_DIR}/bin)

if (Qt5_POSITION_INDEPENDENT_CODE)
  SET(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif(Qt5_POSITION_INDEPENDENT_CODE)

target_sources(${APP_NAME} 
  PRIVATE
    "${CMAKE_CURRENT_LIST_DIR}/appconfig.h"
    "${CMAKE_CURRENT_LIST_DIR}/appconfig.cpp")

#set(SOURCE_ROOT ${CMAKE_SOURCE_DIR})
include(CompleteBundle)

if (WIN32)
  set(RUN_SCRIPT_FILE_NAME "run-app.bat")
else()
  set(RUN_SCRIPT_FILE_NAME "run-app.sh")
endif()

configure_file(
  ${RUN_SCRIPT_FILE_NAME}.in
  ${CMAKE_CURRENT_BINARY_DIR}/${RUN_SCRIPT_FILE_NAME}
  @ONLY
)
