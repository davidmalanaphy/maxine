cmake_minimum_required(VERSION 2.8.0)
project(maxine CXX)

include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

list(APPEND CMAKE_MODULE_PATH
        "${CMAKE_SOURCE_DIR}/cmake")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

message(STATUS "Submodule: " ${SUBMODULE_FLAGS})
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
	set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "default install path" FORCE )
endif()


if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Debug' as none was specified")
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "default install path" FORCE )
endif()

find_program(CCACHE_FOUND ccache)

if(CCACHE_FOUND)
    set(CMAKE_CXX_COMPILER_LAUNCHER ccache)
endif()

# submodule deps
if(NOT SUBMODULE_ROOT_DIR)
    set(SUBMODULE_ROOT_DIR ${PROJECT_SOURCE_DIR})
endif()
if(NOT SUBMODULE_PREFIX_DIR)
    set(SUBMODULE_PREFIX_DIR ${CMAKE_BINARY_DIR})
endif()

find_library(EVENT_LIB
  NAMES event
  HINTS "${CMAKE_INSTALL_PREFIX}/lib"
)
find_library(EVENT_PTHREAD_LIB
  NAMES event_pthreads
  HINTS "${CMAKE_INSTALL_PREFIX}/lib"
)
find_path(EVENT_INCLUDE_DIR event2/event.h
        PATHS "${CMAKE_INSTALL_PREFIX}/include")

if (EVENT_LIB AND EVENT_PTHREAD_LIB)
  message(STATUS "event-found: " ${EVENT_LIB})
  message(STATUS "event-pthread-found: " ${EVENT_PTHREAD_LIB})
  message(STATUS "event-headers-found: " ${EVENT_INCLUDE_DIR})
  include_directories(${EVENT_INCLUDE_DIR})
else()
  set(LIBEVENT_CMAKE ${SUBMODULE_ROOT_DIR}/ext/libevent CACHE PATH "Location of libevent-cmake" FORCE)
  message(STATUS "libevent-location: " ${LIBEVENT_CMAKE})
  ExternalProject_Add(
    LIBEVENT_PROJECT
    PREFIX ${SUBMODULE_PREFIX_DIR}
    SOURCE_DIR ${LIBEVENT_CMAKE}
    CMAKE_ARGS -DEVENT__DISABLE_TESTS=ON -DEVENT__DISABLE_REGRESS=ON -DEVENT__DISABLE_OPENSSL=ON -DEVENT__DISABLE_SAMPLES=ON -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
    ExternalProject_Get_Property(LIBEVENT_PROJECT binary_dir)
    set(CPACK_INSTALL_CMAKE_PROJECTS ${CPACK_INSTALL_CMAKE_PROJECTS} "${binary_dir};libevent;ALL;/")
endif()

# thirdparty libraries

include_directories(SYSTEM ${CMAKE_INSTALL_PREFIX}/thirdparty/include)
link_directories(${CMAKE_INSTALL_PREFIX}/thirdparty/lib)

#properties
set(PROPERTIES_CMAKE ${SUBMODULE_ROOT_DIR}/ext/properties CACHE PATH "Location of properties-cmake" FORCE)
message(STATUS "properties-location: " ${PROPERTIES_CMAKE})
ExternalProject_Add(
    PROPERTIES_PROJECT
    PREFIX ${SUBMODULE_PREFIX_DIR}
    SOURCE_DIR ${PROPERTIES_CMAKE}
    CMAKE_ARGS -DSUBMODULE_PREFIX_DIR=${SUBMODULE_PREFIX_DIR} -DSUBMODULE_ROOT_DIR=${SUBMODULE_ROOT_DIR} -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} ${SUBMODULE_FLAGS} ${PYTHON_FLAG}
    )
ExternalProject_Get_Property(PROPERTIES_PROJECT binary_dir)
set(PROPERTIES_PROJECT_DIR ${binary_dir}/lib/cmake/properties CACHE PATH "location of properties cmake files" FORCE)
set(CPACK_INSTALL_CMAKE_PROJECTS ${CPACK_INSTALL_CMAKE_PROJECTS} "${binary_dir};properties;ALL;/")

#logger
set(LOGGER_CMAKE ${SUBMODULE_ROOT_DIR}/ext/logger CACHE PATH "Location of logger-cmake" FORCE)
message(STATUS "logger-location: " ${LOGGER_CMAKE})
ExternalProject_Add(
  LOGGER_PROJECT
  PREFIX ${SUBMODULE_PREFIX_DIR}
  SOURCE_DIR ${LOGGER_CMAKE}
  CMAKE_ARGS -DSUBMODULE_PREFIX_DIR=${SUBMODULE_PREFIX_DIR} -DSUBMODULE_ROOT_DIR=${SUBMODULE_ROOT_DIR} -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} ${SUBMODULE_FLAGS} ${PYTHON_FLAG}
  )
ExternalProject_Get_Property(LOGGER_PROJECT binary_dir)
set(LOGGER_PROJECT_DIR ${binary_dir}/lib/cmake/logger CACHE PATH "location of logger cmake files" FORCE)
set(CPACK_INSTALL_CMAKE_PROJECTS ${CPACK_INSTALL_CMAKE_PROJECTS} "${binary_dir};logger;ALL;/")

#codec
set(CODEC_CMAKE ${SUBMODULE_ROOT_DIR}/ext/codec CACHE PATH "Location of codec-cmake" FORCE)
message(STATUS "codec-location: " ${CODEC_CMAKE})
ExternalProject_Add(
  CODEC_PROJECT
  PREFIX ${SUBMODULE_PREFIX_DIR}
  SOURCE_DIR ${CODEC_CMAKE}
  CMAKE_ARGS -DSUBMODULE_PREFIX_DIR=${SUBMODULE_PREFIX_DIR} -DSUBMODULE_ROOT_DIR=${SUBMODULE_ROOT_DIR} -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} ${SUBMODULE_FLAGS} -DEXAMPLES=ON ${PYTHON_FLAG}
  )
ExternalProject_Get_Property(CODEC_PROJECT binary_dir)
set(CODEC_PROJECT_DIR ${binary_dir}/lib/cmake/codec CACHE PATH "location of codec cmake files" FORCE)
set(CPACK_INSTALL_CMAKE_PROJECTS ${CPACK_INSTALL_CMAKE_PROJECTS} "${binary_dir};codec;ALL;/")

add_dependencies(LOGGER_PROJECT PROPERTIES_PROJECT) 
add_dependencies(CODEC_PROJECT PROPERTIES_PROJECT LOGGER_PROJECT)

