cmake_minimum_required(VERSION 2.8.7 FATAL_ERROR)
project(gl_platform_test)

#------------------------------------------------------------------------------
# Required CPM Setup - See: http://github.com/CIBC-Internal/cpm
#------------------------------------------------------------------------------
set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
find_package(Git)
if(NOT GIT_FOUND)
  message(FATAL_ERROR "CPM requires Git.")
endif()
if ((NOT DEFINED CPM_MODULE_CACHE_DIR) AND (NOT "$ENV{CPM_CACHE_DIR}" STREQUAL ""))
  set(CPM_MODULE_CACHE_DIR "$ENV{CPM_CACHE_DIR}")
endif()
if ((NOT EXISTS ${CPM_DIR}/CPM.cmake) AND (DEFINED CPM_MODULE_CACHE_DIR))
  if (EXISTS "${CPM_MODULE_CACHE_DIR}/github_cibcinternal_cpm")
    message(STATUS "Found cached version of CPM.")
    file(COPY "${CPM_MODULE_CACHE_DIR}/github_cibcinternal_cpm/" DESTINATION ${CPM_DIR})
  endif()
endif()
if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
  message(STATUS "Cloning repo (https://github.com/CIBC-Internal/cpm)")
  execute_process(
    COMMAND "${GIT_EXECUTABLE}" clone https://github.com/CIBC-Internal/cpm ${CPM_DIR}
    RESULT_VARIABLE error_code
    OUTPUT_QUIET ERROR_QUIET)
  if(error_code)
    message(FATAL_ERROR "CPM failed to get the hash for HEAD")
  endif()
endif()
include(${CPM_DIR}/CPM.cmake)

# ++ MODULE: GL Platform
CPM_AddModule("gl_platform"
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/..")

# ++ MODULE: GL Context creation
CPM_AddModule("gl_context"
  GIT_REPOSITORY "https://github.com/CIBC-Internal/cpm-gl-batch-context.git"
  GIT_TAG "origin/master")

CPM_Finish()

#-----------------------------------------------------------------------
# Configure OpenGL
#-----------------------------------------------------------------------
find_package(OpenGL REQUIRED)

#-----------------------------------------------------------------------
# Setup strict warnings and werror
#-----------------------------------------------------------------------

if(APPLE)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror -Wconversion -Wshadow")
endif()

#-----------------------------------------------------------------------
# Setup source
#-----------------------------------------------------------------------
# Setup spire test sources. Even though this is a batch renderer, this is
# a good test of concept.

file(GLOB Sources
  "*.cpp"
  "*.hpp"
  )

#-----------------------------------------------------------------------
# Setup executable
#-----------------------------------------------------------------------

# Make sure we don't link against OpenGL libraries if using OSMesa
if (USE_OS_MESA)
  set(OPENGL_LIBRARIES)
endif()

add_executable(gl_platform_test ${Sources})
target_link_libraries(gl_platform_test
  ${CPM_LIBRARIES}
  ${OPENGL_LIBRARIES})

