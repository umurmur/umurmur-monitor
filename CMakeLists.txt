cmake_minimum_required(VERSION 2.8)

project(umurmur-monitor C)

include(CheckLibraryExists)
include(CheckFunctionExists)
check_library_exists(rt clock_gettime "time.h" HAVE_CLOCK_GETTIME_RT)

if(NOT HAVE_CLOCK_GETTIME_RT)
  check_function_exists(clock_gettime HAVE_CLOCK_GETTIME)
  if(NOT HAVE_CLOCK_GETTIME)
    check_library_exists(c clock_get_time "mach/time.h" HAVE_CLOCK_GET_TIME)
    if(NOT HAVE_CLOCK_GET_TIME)
      message(FATAL_ERROR "Didn't find clock_gettime() or clock_get_time()!")
    endif()
  endif()
else()
  set(LIBRT rt)
endif()

set(SOURCE "src/main.c")

if(UMURMUR_ROOT_PATH)
  include_directories("${UMURMUR_ROOT_PATH}/src")
else()
  if(EXISTS "${CMAKE_CURRENT_LIST_DIR}/../umurmur/CMakeLists.txt")
    include_directories("../umurmur/src")
  else()
    message(FATAL_ERROR "Could not determine the umurmur source path. Please specify UMURMUR_ROOT_PATH")
  endif()
endif()

if(UMURMUR_BINARY_DIR)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${UMURMUR_BINARY_DIR})
endif()

add_executable(umurmur-monitor ${SOURCE})
target_link_libraries(umurmur-monitor ${LIBRT})

install(TARGETS umurmur-monitor RUNTIME DESTINATION "bin")
