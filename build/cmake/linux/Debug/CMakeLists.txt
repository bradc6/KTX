cmake_minimum_required(VERSION 3.10.2 FATAL_ERROR)
cmake_policy(VERSION 3.10.2)
project(KTX)

set(CMAKE_CONFIGURATION_TYPES Debug Release CACHE INTERNAL "")
#If you don't choose a build type we choose debug for you
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set(configuration "Debug")
enable_language(ASM)
set(builddir "${CMAKE_CURRENT_BINARY_DIR}")
set(obj "${builddir}/obj")

set(CMAKE_C_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_NINJA_FORCE_RESPONSE_FILE 1)

#Base filepaths that everything else will alias off of
set(LIBKTX_LIB_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../lib)
set(LIBKTX_PUBLIC_HEADERS_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../include)
set(LIBKTX_PRIVATE_HEADERS_DIR ${CMAKE_CURRENT_LIST_DIR}/../../../../other_include)

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

target_compile_definitions(libktx_es3 PRIVATE KTX_OPENGL_ES3=1 $<IF:$<CONFIG:Debug>,DEBUG=1,_DEBUG=1>)

set_target_properties(libktx_es3 PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_es3 PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.es3")

############################################
#LIBKTX built for OpenGL ES 1
############################################
add_library(libktx_es1 STATIC ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})
#Public include directories are "inherited" to those who link to the target
target_include_directories(libktx_es1 PUBLIC ${LIBKTX_PUBLIC_HEADERS_DIR})
target_include_directories(libktx_es1 PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})

target_compile_definitions(libktx_es1 PRIVATE KTX_OPENGL_ES1=1 $<IF:$<CONFIG:Debug>,DEBUG=1,_DEBUG=1>)

set_target_properties(libktx_es1 PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_es1 PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.es1")

############################################
#LIBKTX built for OpenGL Desktop
############################################
add_library(libktx_gl STATIC ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})
#Public include directories are "inherited" to those who link to the target
target_include_directories(libktx_gl PUBLIC ${LIBKTX_PUBLIC_HEADERS_DIR})
target_include_directories(libktx_gl PRIVATE ${LIBKTX_PRIVATE_HEADERS_DIR})

target_compile_definitions(libktx_gl PRIVATE KTX_OPENGL=1 $<IF:$<CONFIG:Debug>,DEBUG=1,_DEBUG=1>)

set_target_properties(libktx_gl PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${LIBKTX_BINARY_LIBRARY_OUT_DIR}/$<CONFIG>/lib)
set_target_properties(libktx_gl PROPERTIES ARCHIVE_OUTPUT_NAME "ktx.gl")

############################################
#LIBKTX Tools
############################################

############################################
#LIBKTX Documentation
############################################

set(LIBKTX_DOCUMENTATION_SOURCE "")
#include the LibKTX source
list(APPEND LIBKTX_DOCUMENTATION_SOURCE ${LIBKTX_PUBLIC_HEADERS} ${LIBKTX_PRIVATE_HEADERS} ${LIBKTX_PRIVATE_SOURCE})



#libktx.gyp:libktx.doc#target
set(TARGET "libktx.doc")
set(TOOLSET "target")
list(APPEND libktx_doc__buildDoc__input
  "${CMAKE_CURRENT_LIST_DIR}/../../../../ktxDoxy"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../LICENSE.md"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../include/ktx.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/checkheader.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/errstr.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/etcdec.cxx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/etcunpack.cxx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gl_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles1_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles2_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles3_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/hashtable.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxfilestream.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxfilestream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxint.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxmemstream.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxmemstream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxstream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/loader.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/swap.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/uthash.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/writer.c")
list(APPEND libktx_doc__buildDoc__output
  "${CMAKE_CURRENT_LIST_DIR}/../../../../build/doc/html"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../build/doc/latex"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../build/doc/man")
add_custom_command(OUTPUT ${libktx_doc__buildDoc__output}
  COMMAND ${CMAKE_COMMAND} -E make_directory build/doc
  COMMAND doxygen ktxDoxy
  DEPENDS ${libktx_doc__buildDoc__input}
  WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
  COMMENT Generating documentation with Doxygen
  VERBATIM
)
add_custom_target(libktx_doc__buildDoc
  DEPENDS ${libktx_doc__buildDoc__output}
  SOURCES ${libktx_doc__buildDoc__input}
)
list(APPEND libktx_doc__touchTimestamp__input
  "${CMAKE_CURRENT_LIST_DIR}/../../../../ktxDoxy"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../LICENSE.md"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../include/ktx.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/checkheader.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/errstr.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/etcdec.cxx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/etcunpack.cxx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gl_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles1_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles2_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/gles3_funcptrs.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/hashtable.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxfilestream.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxfilestream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxint.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxmemstream.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxmemstream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/ktxstream.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/loader.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/swap.c"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/uthash.h"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../lib/writer.c")
set(libktx_doc__touchTimestamp__output "${CMAKE_CURRENT_LIST_DIR}/../../../../build/doc/.gentimestamp")
add_custom_command(OUTPUT ${libktx_doc__touchTimestamp__output}
  COMMAND ${CMAKE_COMMAND} -E make_directory build/doc
  COMMAND touch build/doc/.gentimestamp
  DEPENDS ${libktx_doc__touchTimestamp__input}
  WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
  COMMENT setting generation timestamp
  VERBATIM
)
add_custom_target(libktx_doc__touchTimestamp
  DEPENDS ${libktx_doc__touchTimestamp__output}
  SOURCES ${libktx_doc__touchTimestamp__input}
)
add_custom_target(libktx_doc SOURCES)
set_target_properties(libktx_doc PROPERTIES EXCLUDE_FROM_ALL "TRUE")
set_target_properties(libktx_doc__buildDoc PROPERTIES EXCLUDE_FROM_ALL "TRUE")
set_target_properties(libktx_doc__touchTimestamp PROPERTIES EXCLUDE_FROM_ALL "TRUE")
add_dependencies(libktx_doc
  libktx_doc__buildDoc
  libktx_doc__touchTimestamp
)
unset(TOOLSET)
unset(TARGET)


#libktx.gyp:libgl#target
 set(TARGET "libgl")
 set(TOOLSET "target")
 add_custom_target(libgl SOURCES)
 set_target_properties(libgl PROPERTIES EXCLUDE_FROM_ALL "FALSE")
 unset(TOOLSET)
 unset(TARGET)

#ktxtools.gyp:toktx#target
set(TARGET "toktx")
set(TOOLSET "target")
list(APPEND toktx__cxx_srcs
  "../../../../tools/toktx/image.cpp"
  "../../../../tools/toktx/toktx.cpp")
list(APPEND toktx__other_srcs
  "../../../../tools/toktx/image.h"
  "../../../../tools/toktx/stdafx.h"
  "../../../../tools/toktx/targetver.h")
add_executable(toktx ${toktx__cxx_srcs} ${toktx__other_srcs})
set_target_properties(toktx PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_source_files_properties(${toktx__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
set_target_properties(toktx PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${builddir}")
set_target_properties(toktx PROPERTIES PREFIX "")
set_target_properties(toktx PROPERTIES RUNTIME_OUTPUT_NAME "toktx")
set_target_properties(toktx PROPERTIES SUFFIX "")
set_source_files_properties(${builddir}/toktx PROPERTIES GENERATED "TRUE")
list(APPEND toktx__include_dirs
  "${CMAKE_CURRENT_LIST_DIR}/../../../../include"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../other_include")
set_property(TARGET toktx APPEND PROPERTY INCLUDE_DIRECTORIES ${toktx__include_dirs})
set_target_properties(toktx PROPERTIES COMPILE_DEFINITIONS "KTX_OPENGL=1;DEBUG;_DEBUG;")
set_target_properties(toktx PROPERTIES COMPILE_FLAGS "-O0 ")
set_target_properties(toktx PROPERTIES XCODE_ATTRIBUTE_COPY_PHASE_STRIP "NO")
set_target_properties(toktx PROPERTIES XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS "YES")
set_target_properties(toktx PROPERTIES XCODE_ATTRIBUTE_PRODUCT_BUNDLE_IDENTIFIER "org.khronos.${PRODUCT_NAME}")
set_target_properties(toktx PROPERTIES XCODE_ATTRIBUTE_STRIP_INSTALLED_PRODUCT "NO")
set_target_properties(toktx PROPERTIES XCODE_ATTRIBUTE_GCC_OPTIMIZATION_LEVEL "0")
target_link_libraries(toktx
  libktx_gl
)
unset(TOOLSET)
unset(TARGET)

#ktxtools.gyp:toktx-tests#target
set(TARGET "toktx-tests")
set(TOOLSET "target")
set(toktx-tests__toktx-tests__input "${CMAKE_CURRENT_LIST_DIR}/../../../../tests/toktx-tests")
set(toktx-tests__toktx-tests__output "${CMAKE_CURRENT_LIST_DIR}/../../../../tools/toktx/testsrun")
add_custom_command(OUTPUT ${toktx-tests__toktx-tests__output}
  COMMAND ${CMAKE_COMMAND} -E make_directory tools/toktx
  COMMAND tests/toktx-tests "${builddir}/toktx"
  DEPENDS ${toktx-tests__toktx-tests__input}
  WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
  COMMENT Running toktx tests
  VERBATIM
)
add_custom_target(toktx-tests__toktx-tests
  DEPENDS ${toktx-tests__toktx-tests__output}
  SOURCES ${toktx-tests__toktx-tests__input}
)
add_custom_target(toktx-tests SOURCES)
set_target_properties(toktx-tests PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_target_properties(toktx-tests__toktx-tests PROPERTIES EXCLUDE_FROM_ALL "FALSE")
add_dependencies(toktx-tests__toktx-tests
  toktx
)
add_dependencies(toktx-tests
  toktx
  toktx-tests__toktx-tests
)
unset(TOOLSET)
unset(TARGET)

#ktxtests.gyp:libsdl#target
set(TARGET "libsdl")
set(TOOLSET "target")
list(APPEND libsdl__copies__input
  "${CMAKE_CURRENT_LIST_DIR}/../../../../other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0.4.0")
list(APPEND libsdl__copies__output
  "${builddir}/libSDL2-2.0.so.0"
  "${builddir}/libSDL2-2.0.so.0.4.0")
add_custom_command(
OUTPUT ${libsdl__copies__output}
COMMAND ${CMAKE_COMMAND} -E copy other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0 ${builddir}/libSDL2-2.0.so.0
COMMAND ${CMAKE_COMMAND} -E copy other_lib/linux/${configuration}-x64/libSDL2-2.0.so.0.4.0 ${builddir}/libSDL2-2.0.so.0.4.0
DEPENDS ${libsdl__copies__input}
WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
COMMENT Copying for libsdl
VERBATIM
)
add_custom_target(libsdl__copies
  DEPENDS ${libsdl__copies__output}
  SOURCES ${libsdl__copies__input}
)
add_custom_target(libsdl SOURCES)
set_target_properties(libsdl PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_target_properties(libsdl__copies PROPERTIES EXCLUDE_FROM_ALL "FALSE")
add_dependencies(libsdl
  libsdl__copies
)
unset(TOOLSET)
unset(TARGET)

#ktxtests.gyp:appfwSDL#target
set(TARGET "appfwSDL")
set(TOOLSET "target")
list(APPEND appfwSDL__cxx_srcs
  "../../../../tests/loadtests/appfwSDL/main.cpp"
  "../../../../tests/loadtests/appfwSDL/AppBaseSDL.cpp"
  "../../../../tests/loadtests/appfwSDL/GLAppSDL.cpp")
list(APPEND appfwSDL__other_srcs
  "../../../../tests/loadtests/appfwSDL/AppBaseSDL.h"
  "../../../../tests/loadtests/appfwSDL/GLAppSDL.h")
add_library(appfwSDL STATIC ${appfwSDL__cxx_srcs} ${appfwSDL__other_srcs})
set_target_properties(appfwSDL PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_source_files_properties(${appfwSDL__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
set_target_properties(appfwSDL PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${obj}.${TOOLSET}/")
set_target_properties(appfwSDL PROPERTIES PREFIX "lib")
set_target_properties(appfwSDL PROPERTIES ARCHIVE_OUTPUT_NAME "appfwSDL")
set_target_properties(appfwSDL PROPERTIES SUFFIX ".a")
set_source_files_properties(${obj}.${TOOLSET}/libappfwSDL.a PROPERTIES GENERATED "TRUE")
list(APPEND appfwSDL__include_dirs
  "${CMAKE_CURRENT_LIST_DIR}/../../../../tests/loadtests/common"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../other_include")
set_property(TARGET appfwSDL APPEND PROPERTY INCLUDE_DIRECTORIES ${appfwSDL__include_dirs})
set_target_properties(appfwSDL PROPERTIES COMPILE_DEFINITIONS "DEBUG;_DEBUG;")
set_target_properties(appfwSDL PROPERTIES COMPILE_FLAGS "-O0 ")
set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_COPY_PHASE_STRIP "NO")
set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS "YES")
set_target_properties(appfwSDL PROPERTIES XCODE_ATTRIBUTE_GCC_OPTIMIZATION_LEVEL "0")
add_dependencies(appfwSDL
  libsdl
)
unset(TOOLSET)
unset(TARGET)

#ktxtests.gyp:gl3loadtests#target
set(TARGET "gl3loadtests")
set(TOOLSET "target")
list(APPEND gl3loadtests__copies__input
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_R11_EAC.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_RG11_EAC.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_SIGNED_R11_EAC.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/conftestimage_SIGNED_RG11_EAC.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/down-reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc1.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgba1.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgba8.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-rgb.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGBa1.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGBa8.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/etc2-sRGB.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/hi_mark.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/hi_mark_sq.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/luminance_sized_reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/luminance_unsized_reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/no-npot.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-amg-reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgba-reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-mipmap-reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/rgb-reference.ktx"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../testimages/up-reference.ktx")
list(APPEND gl3loadtests__copies__output
  "${builddir}/testimages/conftestimage_R11_EAC.ktx"
  "${builddir}/testimages/conftestimage_RG11_EAC.ktx"
  "${builddir}/testimages/conftestimage_SIGNED_R11_EAC.ktx"
  "${builddir}/testimages/conftestimage_SIGNED_RG11_EAC.ktx"
  "${builddir}/testimages/down-reference.ktx"
  "${builddir}/testimages/etc1.ktx"
  "${builddir}/testimages/etc2-rgba1.ktx"
  "${builddir}/testimages/etc2-rgba8.ktx"
  "${builddir}/testimages/etc2-rgb.ktx"
  "${builddir}/testimages/etc2-sRGBa1.ktx"
  "${builddir}/testimages/etc2-sRGBa8.ktx"
  "${builddir}/testimages/etc2-sRGB.ktx"
  "${builddir}/testimages/hi_mark.ktx"
  "${builddir}/testimages/hi_mark_sq.ktx"
  "${builddir}/testimages/luminance_sized_reference.ktx"
  "${builddir}/testimages/luminance_unsized_reference.ktx"
  "${builddir}/testimages/no-npot.ktx"
  "${builddir}/testimages/rgb-amg-reference.ktx"
  "${builddir}/testimages/rgba-reference.ktx"
  "${builddir}/testimages/rgb-mipmap-reference.ktx"
  "${builddir}/testimages/rgb-reference.ktx"
  "${builddir}/testimages/up-reference.ktx")
add_custom_command(
OUTPUT ${gl3loadtests__copies__output}
COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_R11_EAC.ktx ${builddir}/testimages/conftestimage_R11_EAC.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_RG11_EAC.ktx ${builddir}/testimages/conftestimage_RG11_EAC.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_SIGNED_R11_EAC.ktx ${builddir}/testimages/conftestimage_SIGNED_R11_EAC.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/conftestimage_SIGNED_RG11_EAC.ktx ${builddir}/testimages/conftestimage_SIGNED_RG11_EAC.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/down-reference.ktx ${builddir}/testimages/down-reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc1.ktx ${builddir}/testimages/etc1.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgba1.ktx ${builddir}/testimages/etc2-rgba1.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgba8.ktx ${builddir}/testimages/etc2-rgba8.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-rgb.ktx ${builddir}/testimages/etc2-rgb.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGBa1.ktx ${builddir}/testimages/etc2-sRGBa1.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGBa8.ktx ${builddir}/testimages/etc2-sRGBa8.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/etc2-sRGB.ktx ${builddir}/testimages/etc2-sRGB.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/hi_mark.ktx ${builddir}/testimages/hi_mark.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/hi_mark_sq.ktx ${builddir}/testimages/hi_mark_sq.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/luminance_sized_reference.ktx ${builddir}/testimages/luminance_sized_reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/luminance_unsized_reference.ktx ${builddir}/testimages/luminance_unsized_reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/no-npot.ktx ${builddir}/testimages/no-npot.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-amg-reference.ktx ${builddir}/testimages/rgb-amg-reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/rgba-reference.ktx ${builddir}/testimages/rgba-reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-mipmap-reference.ktx ${builddir}/testimages/rgb-mipmap-reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/rgb-reference.ktx ${builddir}/testimages/rgb-reference.ktx
COMMAND ${CMAKE_COMMAND} -E copy testimages/up-reference.ktx ${builddir}/testimages/up-reference.ktx
DEPENDS ${gl3loadtests__copies__input}
WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/../../../../
COMMENT Copying for gl3loadtests
VERBATIM
)
add_custom_target(gl3loadtests__copies
  DEPENDS ${gl3loadtests__copies__output}
  SOURCES ${gl3loadtests__copies__input}
)
list(APPEND gl3loadtests__c_srcs
  "../../../../tests/loadtests/common/at.c"
  "../../../../tests/loadtests/shader-based/sample_01_draw_texture.c"
  "../../../../tests/loadtests/shader-based/sample_02_cube_textured.c"
  "../../../../tests/loadtests/shader-based/shaderfuncs.c"
  "../../../../tests/loadtests/shader-based/shaders.c")
list(APPEND gl3loadtests__cxx_srcs
  "../../../../tests/loadtests/common/LoadTests.cpp"
  "../../../../tests/loadtests/shader-based/LoadTestsGL3.cpp")
list(APPEND gl3loadtests__other_srcs
  "../../../../tests/loadtests/common/at.h"
  "../../../../tests/loadtests/common/LoadTests.h"
  "../../../../tests/loadtests/data/cube.h"
  "../../../../tests/loadtests/data/frame.h")
link_directories( ../../../../other_lib/linux/${configuration}-x64
)
add_executable(gl3loadtests ${gl3loadtests__c_srcs} ${gl3loadtests__cxx_srcs} ${gl3loadtests__other_srcs})
set_target_properties(gl3loadtests PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_target_properties(gl3loadtests__copies PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_source_files_properties(${gl3loadtests__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
set_target_properties(gl3loadtests PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${builddir}")
set_target_properties(gl3loadtests PROPERTIES PREFIX "")
set_target_properties(gl3loadtests PROPERTIES RUNTIME_OUTPUT_NAME "gl3loadtests")
set_target_properties(gl3loadtests PROPERTIES SUFFIX "")
set_source_files_properties(${builddir}/gl3loadtests PROPERTIES GENERATED "TRUE")
list(APPEND gl3loadtests__include_dirs
  "${CMAKE_CURRENT_LIST_DIR}/../../../../tests/loadtests/common"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../other_include"
  "${CMAKE_CURRENT_LIST_DIR}/../../../../include")
set_property(TARGET gl3loadtests APPEND PROPERTY INCLUDE_DIRECTORIES ${gl3loadtests__include_dirs})
set_target_properties(gl3loadtests PROPERTIES COMPILE_DEFINITIONS "GL_CONTEXT_PROFILE=SDL_GL_CONTEXT_PROFILE_CORE;GL_CONTEXT_MAJOR_VERSION=3;GL_CONTEXT_MINOR_VERSION=3;KTX_OPENGL=1;DEBUG;_DEBUG;")
set_target_properties(gl3loadtests PROPERTIES COMPILE_FLAGS "-O0 ")
set_target_properties(gl3loadtests PROPERTIES LINK_FLAGS "-Wl,-rpath,. ")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_COPY_PHASE_STRIP "NO")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_GCC_GENERATE_DEBUGGING_SYMBOLS "YES")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_STRIP_INSTALLED_PRODUCT "NO")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_GCC_OPTIMIZATION_LEVEL "0")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_PRODUCT_BUNDLE_IDENTIFIER "org.khronos.${PRODUCT_NAME}")
set_target_properties(gl3loadtests PROPERTIES XCODE_ATTRIBUTE_INFOPLIST_FILE "tests/loadtests/resources_mac/Info.plist")
add_dependencies(gl3loadtests__copies
  appfwSDL
  libktx_gl
  libgl
  libsdl
)
add_dependencies(gl3loadtests
  libgl
  libsdl
  gl3loadtests__copies
)
target_link_libraries(gl3loadtests
  appfwSDL
  libktx_gl
  "-lSDL2-2.0"
  "-lSDL2main"
  "-ldl"
  "-lpthread"
  "-lGL"
)
unset(TOOLSET)
unset(TARGET)
