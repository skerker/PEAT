cmake_minimum_required (VERSION 2.6)
project (PEAT)
set(CMAKE_VERBOSE_MAKEFILE ON)

# Initialize CXXFLAGS.
set(CMAKE_CXX_FLAGS "-Ofast -std=c++11")
#For static in linux
#set(CMAKE_CXX_FLAGS "-static -Ofast -std=c++11 -Wl,--whole-archive -lpthread -Wl,--no-whole-archive")
#For cgdb
#set(CMAKE_CXX_FLAGS "-O0 -std=c++11 -w -g -DDEBUG_log=1")
#if your boost is installed in somewhere CMake cannot find, please specified as below:
#SET(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} "$HOME/Downloads/boost_1_51_0/")
#SET(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "$HOME/Downloads/boost_1_51_0/lib")
# Compiler-specific C++11 activation.
if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
    execute_process(
        COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
    if (NOT (GCC_VERSION VERSION_GREATER 4.7 OR GCC_VERSION VERSION_EQUAL 4.7))
        message(FATAL_ERROR "${PROJECT_NAME} requires g++ 4.7 or greater.")
    endif ()
elseif ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
else ()
    message(FATAL_ERROR "Your C++ compiler does not support C++11.")
endif ()


if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    set(LINUX TRUE)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(MACOSX TRUE)
endif()

include (${CMAKE_ROOT}/Modules/FindBoost.cmake)

#boost
set(Boost_USE_STATIC_LIBS        OFF)
set(Boost_USE_MULTITHREADED      ON)
set(Boost_USE_STATIC_RUNTIME    OFF)
find_package( Boost COMPONENTS date_time filesystem system serialization system filesystem regex thread iostreams program_options)
find_package( Threads )
find_package( ZLIB )
#find_package( CURL )

if(Boost_FOUND AND Threads_FOUND AND ZLIB_FOUND)
	include_directories(${Boost_INCLUDE_DIRS} ${ZLIB_INCLUDE_DIRS} ) 
# add the executable
	add_executable(bin/PEAT src/main.cpp)
#Static linux
#	add_executable(bin/PEAT_linux_new src/main.cpp)
	add_definitions(-DmyALPHABET)
	if (LINUX)
		message ("Your operating system is Linux...")
		target_link_libraries( bin/PEAT ${Boost_LIBRARIES} ${ZLIB_LIBRARIES} ${CURL_LIBRARIES} ${CMAKE_SOURCE_DIR}/src/ThreadPool/libthread.a ${CMAKE_SOURCE_DIR}/src/iohandler/ihandler/IhandlerFactory.a )
#		target_link_libraries( bin/PEAT_linux ${Boost_LIBRARIES} ${ZLIB_LIBRARIES} ${CURL_LIBRARIES} ${CMAKE_SOURCE_DIR}/src/ThreadPool/libthread.a ${CMAKE_SOURCE_DIR}/src/iohandler/ihandler/IhandlerFactory.a )
	elseif (MACOSX)
		message ("Your operating system is MacOSX...")
		target_link_libraries( bin/PEAT ${Boost_LIBRARIES} ${ZLIB_LIBRARIES} ${CURL_LIBRARIES} ${CMAKE_SOURCE_DIR}/src/ThreadPool/libthread_mac.a ${CMAKE_SOURCE_DIR}/src/iohandler/ihandler/IhandlerFactory_mac.a )
	endif()
endif()
