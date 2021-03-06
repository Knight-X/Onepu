cmake_minimum_required(VERSION 2.6)
project(onepu)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/build/cmake")

#Nuparu location (alternatively, /usr/local if not using Nuparu)
set(NUPARU "../Nuparu")

#Set up include and lib paths
include_directories(${NUPARU}/include)
include_directories(${NUPARU}/src)
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ${NUPARU}/lib/osx)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ${NUPARU}/lib/linux /usr/lib64)
elseif(WIN32)
	set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ${NUPARU}/lib/win)
endif()

set(GLFW_INCLUDE_DIR ${NUPARU}/include)
set(GLFW_LIBRARY_DIR ${CMAKE_LIBRARY_PATH})
set(GLEW_INCLUDE_DIR ${NUPARU}/include)
set(GLEW_LIBRARY_DIR ${CMAKE_LIBRARY_PATH})

#Use GCC instead of clang for now, since clang doesn't have OpenMP support yet
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set (CMAKE_C_COMPILER "gcc")
	set (CMAKE_CXX_COMPILER "g++")
	#Force XCode to gcc
	if(CMAKE_GENERATOR STREQUAL Xcode)
		set(CMAKE_XCODE_ATTRIBUTE_GCC_VERSION "com.apple.compilers.llvmgcc42")
	endif()
endif()

#find up and set up core dependency libs
find_library(GLFW_LIBRARY "glfw3" HINTS ${GLFW_LIBRARY_DIR})
find_package(OPENGL)
find_package(GLEW)
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux" OR ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	find_package(GLUT)
endif()

set(CORELIBS ${GLFW_LIBRARY} ${GLUT_LIBRARY} ${OPENGL_LIBRARY} ${GLEW_LIBRARY} ${IOKIT})

#Set up OpenMP stuff
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	#Workaround for OSX OpenMP detection problems (<=10.8's clang has no OpenMP support)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fopenmp")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fopenmp")
else()
	find_package(OpenMP)
	if(OPENMP_FOUND)
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
	endif()
endif()

# Mac OS X specific hacks/fixes
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
   	#Link IOKit
   	set(IOKIT "-framework IOKit")
	set(CORELIBS ${CORELIBS} ${IOKIT})
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

# Linux specific hacks/fixes
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lX11 -lXxf86vm -lXrandr -lpthread -lXi")
endif()

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux" OR ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -m64 -msse2")
elseif(WIN32)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2")
endif()

set(SOURCE_FILES "src/main.cpp" 
				 "src/viewer/rigviewer.cpp")

add_executable(onepu ${SOURCE_FILES})

target_link_libraries(onepu ${CORELIBS})
