cmake_minimum_required(VERSION 3.10.2 FATAL_ERROR)
cmake_policy(VERSION 3.10.2)
project(KTX)

set(CMAKE_CONFIGURATION_TYPES Debug Release CACHE INTERNAL "")
#If you don't choose a build type we choose debug for you
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build, options are: Debug Release" FORCE)
endif(NOT CMAKE_BUILD_TYPE)

#Find packages on the system
find_package(Doxygen)

#Base filepaths that everything else will alias off of
set(LIBKTX_ROOT_DIR  ${CMAKE_CURRENT_LIST_DIR}/../../../..)
set(LIBKTX_LIB_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../lib)
set(LIBKTX_PUBLIC_HEADERS_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../include)
set(LIBKTX_PRIVATE_HEADERS_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../other_include)

set(LIBKTX_TOOLS_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../tools)

set(LIBKTX_BINARY_OUT_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(LIBKTX_BINARY_LIBRARY_OUT_DIR ${CMAKE_CURRENT_BINARY_DIR})

#The sources for the KTX texture format reference implementation
set(LIBKTX_PUBLIC_HEADERS
  ${LIBKTX_PUBLIC_HEADERS_DIR}/ktx.h
  )

set(LIBKTX_PRIVATE_HEADERS
  ${LIBKTX_LIB_SOURCE_DIR}/gl_funcptrs.h
  ${LIBKTX_LIB_SOURCE_DIR}/gles1_funcptrs.h
  ${LIBKTX_LIB_SOURCE_DIR}/gles2_funcptrs.h
  ${LIBKTX_LIB_SOURCE_DIR}/gles3_funcptrs.h
  ${LIBKTX_LIB_SOURCE_DIR}/ktxfilestream.h
  ${LIBKTX_LIB_SOURCE_DIR}/ktxint.h
  ${LIBKTX_LIB_SOURCE_DIR}/ktxmemstream.h
  ${LIBKTX_LIB_SOURCE_DIR}/ktxstream.h
  ${LIBKTX_LIB_SOURCE_DIR}/uthash.h
  )

set(LIBKTX_PRIVATE_SOURCE
  ${LIBKTX_LIB_SOURCE_DIR}/checkheader.c
  ${LIBKTX_LIB_SOURCE_DIR}/errstr.c
  ${LIBKTX_LIB_SOURCE_DIR}/hashtable.c
  ${LIBKTX_LIB_SOURCE_DIR}/ktxfilestream.c
  ${LIBKTX_LIB_SOURCE_DIR}/ktxmemstream.c
  ${LIBKTX_LIB_SOURCE_DIR}/loader.c
  ${LIBKTX_LIB_SOURCE_DIR}/swap.c
  ${LIBKTX_LIB_SOURCE_DIR}/writer.c
  ${LIBKTX_LIB_SOURCE_DIR}/etcdec.cxx
  ${LIBKTX_LIB_SOURCE_DIR}/etcunpack.cxx
  )

############################################
#LIBKTX built for OpenGL ES 3
############################################
add_library(libktx_es3 STATIC ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})
#Public include directories are "inherited" to those who link to the target
target_include_directories(libktx_es3 PUBLIC ${LIBKTX_PUBLIC_HEADERS_DIR})
target_include_directories(libktx_es3 PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})

target_compile_definitions(libktx_es3 PRIVATE KTX_OPENGL_ES3=1
                                              $<$<CONFIG:Debug>:DEBUG=1>
                                              $<$<CONFIG:Debug>:_DEBUG=1>)

set_target_properties(libktx_es3 PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_es3 PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.es3")

############################################
#LIBKTX built for OpenGL ES 1
############################################
add_library(libktx_es1 STATIC ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})
#Public include directories are "inherited" to those who link to the target
target_include_directories(libktx_es1 PUBLIC ${LIBKTX_PUBLIC_HEADERS_DIR})
target_include_directories(libktx_es1 PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})

target_compile_definitions(libktx_es1 PRIVATE KTX_OPENGL_ES1=1
                                              $<$<CONFIG:Debug>:DEBUG=1>
                                              $<$<CONFIG:Debug>:_DEBUG=1>)

set_target_properties(libktx_es1 PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_es1 PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.es1")

############################################
#LIBKTX built for OpenGL Desktop
############################################
add_library(libktx_gl STATIC ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})
#Public include directories are "inherited" to those who link to the target
target_include_directories(libktx_gl PUBLIC ${LIBKTX_PUBLIC_HEADERS_DIR})
target_include_directories(libktx_gl PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})

target_compile_definitions(libktx_gl PRIVATE KTX_OPENGL=1 
                                    $<$<CONFIG:Debug>:DEBUG=1>
                                    $<$<CONFIG:Debug>:_DEBUG=1>)

set_target_properties(libktx_gl PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_gl PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.gl")

############################################
#LIBKTX Tools
############################################

#TOKTX
set(KTX_TOOL_TOKTX_SOURCE_DIR ${LIBKTX_TOOLS_SOURCE_DIR}/toktx)
set(KTX_TOOL_TOKTX_SOURCE
   ${KTX_TOOL_TOKTX_SOURCE_DIR}/image.h
   ${KTX_TOOL_TOKTX_SOURCE_DIR}/stdafx.h
   ${KTX_TOOL_TOKTX_SOURCE_DIR}/targetver.h
   ${KTX_TOOL_TOKTX_SOURCE_DIR}/image.cpp
   ${KTX_TOOL_TOKTX_SOURCE_DIR}/toktx.cpp
  )

add_executable(toktx ${KTX_TOOL_TOKTX_SOURCE})

target_link_libraries(toktx PUBLIC libktx_gl)
target_include_directories(toktx PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})
target_compile_definitions(toktx PRIVATE $<$<CONFIG:Debug>:DEBUG=1>
                                         $<$<CONFIG:Debug>:_DEBUG=1>
                                         $<$<CONFIG:Debug>:KTX_OPENGL=1>)

set_target_properties(toktx PROPERTIES EXCLUDE_FROM_ALL TRUE)
set_target_properties(toktx PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${LIBKTX_BINARY_OUT_DIR}/$<CONFIG>/bin)


############################################
#LIBKTX Documentation
############################################

set(LIBKTX_DOCUMENTATION_SOURCE
  ${LIBKTX_ROOT_DIR}/LICENSE.md
  )

#include the LibKTX source
list(APPEND LIBKTX_DOCUMENTATION_SOURCE ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})

#If we found doxygen then offer the documentation target
if(DOXYGEN_FOUND)
  #Set Doxygen options
  set(DOXYGEN_GENERATE_HTML YES)
  set(DOXYGEN_GENERATE_MAN YES)
  set(DOXYGEN_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/doc)

  doxygen_add_docs(libktx_docs 
                  ${LIBKTX_DOCUMENTATION_SOURCE}
                  WORKING_DIRECTORY ${LIBKTX_ROOT_DIR}
                  COMMENT "Generating KTX Documentation")
endif()


# #libktx.gyp:libgl#target
#  set(TARGET "libgl")
#  set(TOOLSET "target")
#  add_custom_target(libgl SOURCES)
#  set_target_properties(libgl PROPERTIES EXCLUDE_FROM_ALL "FALSE")
#  unset(TOOLSET)
#  unset(TARGET)



# #ktxtests.gyp:libsdl#target
# set(TARGET "libsdl")
# set(TOOLSET "target")
# list(APPEND libsdl__copies__input
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0.4.0")
# list(APPEND libsdl__copies__output
#   "${builddir}/libSDL2-2.0.so.0"
#   "${builddir}/libSDL2-2.0.so.0.4.0")
# add_custom_command(
# OUTPUT ${libsdl__copies__output}
# COMMAND ${CMAKE_COMMAND} -E copy other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0 ${builddir}/libSDL2-2.0.so.0
# COMMAND ${CMAKE_COMMAND} -E copy other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0.4.0 ${builddir}/libSDL2-2.0.so.0.4.0
# DEPENDS ${libsdl__copies__input}
# WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
# COMMENT Copying for libsdl
# VERBATIM
# )
# add_custom_target(libsdl__copies
#   DEPENDS ${libsdl__copies__output}
#   SOURCES ${libsdl__copies__input}
# )
# add_custom_target(libsdl SOURCES)
# set_target_properties(libsdl PROPERTIES EXCLUDE_FROM_ALL "FALSE")
# set_target_properties(libsdl__copies PROPERTIES EXCLUDE_FROM_ALL "FALSE")
# add_dependencies(libsdl
#   libsdl__copies
# )
# unset(TOOLSET)
# unset(TARGET)

# #ktxtests.gyp:appfwSDL#target
# set(TARGET "appfwSDL")
# set(TOOLSET "target")
# list(APPEND appfwSDL__cxx_srcs
#   "../../../../tests/loadtests/appfwSDL/main.cpp"
#   "../../../../tests/loadtests/appfwSDL/AppBaseSDL.cpp"
#   "../../../../tests/loadtests/appfwSDL/GLAppSDL.cpp")
# list(APPEND appfwSDL__other_srcs
#   "../../../../tests/loadtests/appfwSDL/AppBaseSDL.h"
#   "../../../../tests/loadtests/appfwSDL/GLAppSDL.h")
# add_library(appfwSDL STATIC ${appfwSDL__cxx_srcs} ${appfwSDL__other_srcs})
# set_target_properties(appfwSDL PROPERTIES EXCLUDE_FROM_ALL "FALSE")
# set_source_files_properties(${appfwSDL__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
# set_target_properties(appfwSDL PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${obj}.${TOOLSET}/")
# set_target_properties(appfwSDL PROPERTIES PREFIX "lib")
# set_target_properties(appfwSDL PROPERTIES ARCHIVE_OUTPUT_NAME "appfwSDL")
# set_target_properties(appfwSDL PROPERTIES SUFFIX ".a")
# set_source_files_properties(${obj}.${TOOLSET}/libappfwSDL.a PROPERTIES GENERATED "TRUE")
# list(APPEND appfwSDL__include_dirs
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../tests/loadtests/common"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../other_include")
# set_property(TARGET appfwSDL APPEND PROPERTY INCLUDE_DIRECTORIES ${appfwSDL__include_dirs})
# set_target_properties(appfwSDL PROPERTIES COMPILE_DEFINITIONS "DEBUG;_DEBUG;")
# set_target_properties(appfwSDL PROPERTIES COMPILE_FLAGS "-O0 ")
# set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_COPY_PHASE_STRIP "NO")
# set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS "YES")
# set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_GCC_OPTIMIZATION_LEVEL "0")
# add_dependencies(appfwSDL
#   libsdl
# )
# unset(TOOLSET)
# unset(TARGET)

# #ktxtests.gyp:gl3loadtests#target
# set(TARGET "gl3loadtests")
# set(TOOLSET "target")
# list(APPEND gl3loadtests__copies__input
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_R11_EAC.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_RG11_EAC.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_SIGNED_R11_EAC.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_SIGNED_RG11_EAC.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/down-reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc1.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgba1.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgba8.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgb.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGBa1.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGBa8.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGB.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/hi_mark.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/hi_mark_sq.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/luminance_sized_reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/luminance_unsized_reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/no-npot.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-amg-reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgba-reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-mipmap-reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-reference.ktx"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/up-reference.ktx")
# list(APPEND gl3loadtests__copies__output
#   "${builddir}/testimages/conftestimage_R11_EAC.ktx"
#   "${builddir}/testimages/conftestimage_RG11_EAC.ktx"
#   "${builddir}/testimages/conftestimage_SIGNED_R11_EAC.ktx"
#   "${builddir}/testimages/conftestimage_SIGNED_RG11_EAC.ktx"
#   "${builddir}/testimages/down-reference.ktx"
#   "${builddir}/testimages/etc1.ktx"
#   "${builddir}/testimages/etc2-rgba1.ktx"
#   "${builddir}/testimages/etc2-rgba8.ktx"
#   "${builddir}/testimages/etc2-rgb.ktx"
#   "${builddir}/testimages/etc2-sRGBa1.ktx"
#   "${builddir}/testimages/etc2-sRGBa8.ktx"
#   "${builddir}/testimages/etc2-sRGB.ktx"
#   "${builddir}/testimages/hi_mark.ktx"
#   "${builddir}/testimages/hi_mark_sq.ktx"
#   "${builddir}/testimages/luminance_sized_reference.ktx"
#   "${builddir}/testimages/luminance_unsized_reference.ktx"
#   "${builddir}/testimages/no-npot.ktx"
#   "${builddir}/testimages/rgb-amg-reference.ktx"
#   "${builddir}/testimages/rgba-reference.ktx"
#   "${builddir}/testimages/rgb-mipmap-reference.ktx"
#   "${builddir}/testimages/rgb-reference.ktx"
#   "${builddir}/testimages/up-reference.ktx")
# add_custom_command(
# OUTPUT ${gl3loadtests__copies__output}
# COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_R11_EAC.ktx ${builddir}/testimages/conftestimage_R11_EAC.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_RG11_EAC.ktx ${builddir}/testimages/conftestimage_RG11_EAC.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_SIGNED_R11_EAC.ktx ${builddir}/testimages/conftestimage_SIGNED_R11_EAC.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_SIGNED_RG11_EAC.ktx ${builddir}/testimages/conftestimage_SIGNED_RG11_EAC.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/down-reference.ktx ${builddir}/testimages/down-reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc1.ktx ${builddir}/testimages/etc1.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgba1.ktx ${builddir}/testimages/etc2-rgba1.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgba8.ktx ${builddir}/testimages/etc2-rgba8.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgb.ktx ${builddir}/testimages/etc2-rgb.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGBa1.ktx ${builddir}/testimages/etc2-sRGBa1.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGBa8.ktx ${builddir}/testimages/etc2-sRGBa8.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGB.ktx ${builddir}/testimages/etc2-sRGB.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/hi_mark.ktx ${builddir}/testimages/hi_mark.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/hi_mark_sq.ktx ${builddir}/testimages/hi_mark_sq.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/luminance_sized_reference.ktx ${builddir}/testimages/luminance_sized_reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/luminance_unsized_reference.ktx ${builddir}/testimages/luminance_unsized_reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/no-npot.ktx ${builddir}/testimages/no-npot.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-amg-reference.ktx ${builddir}/testimages/rgb-amg-reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/rgba-reference.ktx ${builddir}/testimages/rgba-reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-mipmap-reference.ktx ${builddir}/testimages/rgb-mipmap-reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-reference.ktx ${builddir}/testimages/rgb-reference.ktx
# COMMAND ${CMAKE_COMMAND} -E copy testimages/up-reference.ktx ${builddir}/testimages/up-reference.ktx
# DEPENDS ${gl3loadtests__copies__input}
# WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
# COMMENT Copying for gl3loadtests
# VERBATIM
# )
# add_custom_target(gl3loadtests__copies
#   DEPENDS ${gl3loadtests__copies__output}
#   SOURCES ${gl3loadtests__copies__input}
# )
# list(APPEND gl3loadtests__c_srcs
#   "../../../../tests/loadtests/common/at.c"
#   "../../../../tests/loadtests/shader-based/sample_01_draw_texture.c"
#   "../../../../tests/loadtests/shader-based/sample_02_cube_textured.c"
#   "../../../../tests/loadtests/shader-based/shaderfuncs.c"
#   "../../../../tests/loadtests/shader-based/shaders.c")
# list(APPEND gl3loadtests__cxx_srcs
#   "../../../../tests/loadtests/common/LoadTests.cpp"
#   "../../../../tests/loadtests/shader-based/LoadTestsGL3.cpp")
# list(APPEND gl3loadtests__other_srcs
#   "../../../../tests/loadtests/common/at.h"
#   "../../../../tests/loadtests/common/LoadTests.h"
#   "../../../../tests/loadtests/data/cube.h"
#   "../../../../tests/loadtests/data/frame.h")
# link_directories( ../../../../other_lib/linux/${configuration}-x64
# )
# add_executable(gl3loadtests ${gl3loadtests__c_srcs} ${gl3loadtests__cxx_srcs} ${gl3loadtests__other_srcs})
# set_target_properties(gl3loadtests PROPERTIES EXCLUDE_FROM_ALL "FALSE")
# set_target_properties(gl3loadtests__copies PROPERTIES EXCLUDE_FROM_ALL "FALSE")
# set_source_files_properties(${gl3loadtests__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
# set_target_properties(gl3loadtests PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${builddir}")
# set_target_properties(gl3loadtests PROPERTIES PREFIX "")
# set_target_properties(gl3loadtests PROPERTIES RUNTIME_OUTPUT_NAME "gl3loadtests")
# set_target_properties(gl3loadtests PROPERTIES SUFFIX "")
# set_source_files_properties(${builddir}/gl3loadtests PROPERTIES GENERATED "TRUE")
# list(APPEND gl3loadtests__include_dirs
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../tests/loadtests/common"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../other_include"
#   "${CMAKE_CURRENT_LIST_DIR}/../../../../include")
# set_property(TARGET gl3loadtests APPEND PROPERTY INCLUDE_DIRECTORIES ${gl3loadtests__include_dirs})
# set_target_properties(gl3loadtests PROPERTIES COMPILE_DEFINITIONS "GL_CONTEXT_PROFILE=SDL_GL_CONTEXT_PROFILE_CORE;GL_CONTEXT_MAJOR_VERSION=3;GL_CONTEXT_MINOR_VERSION=3;KTX_OPENGL=1;DEBUG;_DEBUG;")
# set_target_properties(gl3loadtests PROPERTIES COMPILE_FLAGS "-O0 ")
# set_target_properties(gl3loadtests PROPERTIES LINK_FLAGS "-Wl,-rpath,. ")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_COPY_PHASE_STRIP "NO")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS "YES")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_STRIP_INSTALLED_PRODUCT "NO")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_GCC_OPTIMIZATION_LEVEL "0")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_PRODUCT_BUNDLE_IDENTIFIER "org.khronos.${PRODUCT_NAME}")
# set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_INFOPLIST_FILE "tests/loadtests/resources_mac/Info.plist")
# add_dependencies(gl3loadtests__copies
#   appfwSDL
#   libktx_gl
#   libgl
#   libsdl
# )
# add_dependencies(gl3loadtests
#   libgl
#   libsdl
#   gl3loadtests__copies
# )
# target_link_libraries(gl3loadtests
#   appfwSDL
#   libktx_gl
#   "-lSDL2-2.0"
#   "-lSDL2main"
#   "-ldl"
#   "-lpthread"
#   "-lGL"
# )
# unset(TOOLSET)
# unset(TARGET)
