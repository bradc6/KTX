#Rewrite of the LibKTX cmake build configuration
project(KTX)
cmake_minimum_required(VERSION 2.8.6)

#We will assume you always want to build this project
#with debugging symbols
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif(NOT CMAKE_BUILD_TYPE)

#Sets the directory for cmake to find Cmake configration packages,
#these packages allow for cmake to find libraries that are not included
#in the standard installation of Cmake. The standard packages can be
#found @ /usr/share/cmake-2.8/Modules
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules
                      ${CMAKE_MODULE_PATH})

#Set the current version of the Graphics Examples
set(KTX_MAJOR_VERSION 0)
set(KTX_MINOR_VERSION 0)
set(KTX_PATCH_VERSION 1)

#These variables will act as our list of include folders and linker flags
#for compilation
set(KTXLinkerFlags)
set(KTXIncludeDirectories)
set(KTXLinkerDirectories)

#The depth of the configuration
set(PROJECT_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../)
set(LIBRARY_DIR ${PROJECT_ROOT_DIR}/lib)

#Add our own built in include directory
list(APPEND KTXIncludeDirectories ${PROJECT_ROOT_DIR}/include)

#Find the GLEW library, if we could not find it
#revert to old behavior.
find_package(GLEW)
if(${GLEW_FOUND})
	list(APPEND KTXIncludeDirectories ${GLEW_INCLUDE_DIRS})
	list(APPEND KTXLinkerFlags ${GLEW_LIBRARIES})
	include_directories(${GLEW_INCLUDE_DIRS})
	add_definitions(-DKTX_OPENGL=1)
	add_definitions(-DKTX_USE_GETPROC=1)
else()
	find_path(GLEW_PATH GL/glew.h PATHS $ENV{HI_BSLIB_HOME}/include)
endif()

include_directories(${KTXIncludeDirectories})
link_directories(${KTXLinkerDirectories})


set(LibKTX-Source
	${LIBRARY_DIR}/checkheader.c
	${LIBRARY_DIR}/errstr.c
	${LIBRARY_DIR}/etcdec.cxx
	${LIBRARY_DIR}/etcunpack.cxx
	${LIBRARY_DIR}/hashtable.c
	${LIBRARY_DIR}/ktxint.h
	${LIBRARY_DIR}/loader.c
	${LIBRARY_DIR}/swap.c
	${LIBRARY_DIR}/writer.c
	)

add_library(ktx STATIC ${LibKTX-Source})
target_link_libraries(ktx ${KTXLinkerFlags})
