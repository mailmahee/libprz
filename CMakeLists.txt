CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)
set(PROJECT_NAME_STR prz)
project(${PROJECT_NAME_STR} C CXX)

#-------------------
# The version number
#-------------------
set (libprz_VERSION_MAJOR 0)
set (libprz_VERSION_MINOR 5)
set (libprz_VERSION_STRING ${libprz_VERSION_MAJOR}.${libprz_VERSION_MINOR})

#-------------------
# setup deps
#-------------------
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(Boost_USE_STATIC_LIBS ON)
  set(Boost_USE_STATIC_RUNTIME ON)
else()
  set(Boost_USE_STATIC_LIBS OFF)
  set(Boost_USE_STATIC_RUNTIME OFF)
endif()

# Boost
set(Boost_USE_MULTITHREADED ON)
find_package(Boost 1.47.0 COMPONENTS unit_test_framework REQUIRED)
set(INCLUDES ${INCLUDES} ${Boost_INCLUDE_DIRS} )
set(LIBS ${LIBS} ${Boost_LIBRARIES} )

# Threading
find_package(Threads)
set(LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT} )

# LevelDB
find_path(LevelDB_INCLUDE_DIRS NAMES leveldb/db.h HINTS /usr/include /usr/local/include)
find_library(LevelDB_LIBRARIES NAMES libleveldb.a libleveldb.lib libleveldb.dylib HINTS /usr/lib /usr/local/lib)

if(LevelDB_INCLUDE_DIRS AND LevelDB_LIBRARIES)
  set(LevelDB_FOUND TRUE)
endif(LevelDB_INCLUDE_DIRS AND LevelDB_LIBRARIES)

if(LevelDB_FOUND)
  message(STATUS "Found LevelDB: ${LevelDB_LIBRARY}")
else(LevelDB_FOUND)
  message(FATAL_ERROR "Could not find leveldb library.")
endif(LevelDB_FOUND)

set(INCLUDES ${INCLUDES} ${LevelDB_INCLUDE_DIRS} )
set(LIBS ${LIBS} ${LevelDB_LIBRARIES} )

#-------------------
# libprz static and shared
#-------------------
set(PROJECT_LIB_NAME ${PROJECT_NAME_STR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "build/bin/")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build/bin)
set(PROJECT_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)

include_directories(${INCLUDES} ${PROJECT_INCLUDE_DIR}/libprz ${PROJECT_SOURCE_DIR}/src)
file(GLOB SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cpp)

add_library(${PROJECT_LIB_NAME} SHARED ${SRC_FILES})
add_library(${PROJECT_LIB_NAME}-static STATIC ${SRC_FILES})

target_link_libraries(${PROJECT_LIB_NAME} ${LIBS})
SET_TARGET_PROPERTIES(${PROJECT_LIB_NAME}-static PROPERTIES OUTPUT_NAME ${PROJECT_LIB_NAME})
SET_TARGET_PROPERTIES(${PROJECT_LIB_NAME} PROPERTIES VERSION ${libprz_VERSION_MAJOR}.${libprz_VERSION_MINOR} SOVERSION ${libprz_VERSION_MAJOR})

set_property(
  TARGET ${PROJECT_LIB_NAME}
  APPEND PROPERTY COMPILE_FLAGS "-Wall -Werror -ansi -Wno-deprecated -pthread -g")

set_property(
  TARGET ${PROJECT_LIB_NAME}-static
  APPEND PROPERTY COMPILE_FLAGS "-Wall -Werror -ansi -Wno-deprecated -pthread -g")

#-------------------
# install target
#-------------------
FILE(GLOB INSTALL_HEADERS "${PROJECT_INCLUDE_DIR}/libprz/*.hpp")
INSTALL(FILES ${INSTALL_HEADERS} DESTINATION include/libprz)

INSTALL(TARGETS ${PROJECT_LIB_NAME}-static ${PROJECT_LIB_NAME}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)

#-------------------
# uninstall target
#-------------------
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

#-------------------
# unit tests
#-------------------
enable_testing()
add_subdirectory(test/unit)

#-------------------
# the demo program
#-------------------
# add_subdirectory(src/demo EXCLUDE_FROM_ALL)
