###############################################################################
cmake_minimum_required(VERSION 3.0)
project(Botanical_Tree_Simulation)

###############################################################################

find_package(OpenGL REQUIRED)

# c++11, -g option is used to export debug symbols for gdb
if(${CMAKE_CXX_COMPILER_ID} MATCHES GNU OR
    ${CMAKE_CXX_COMPILER_ID} MATCHES Clang)
  # Using C++11 on OSX requires using libc++ instead of libstd++.
  # libc++ is an implementation of the C++ standard library for OSX.
  if(APPLE)
    if(XCODE)
      set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")
      set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
    else()
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++")
    endif()
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g")
  endif()
endif()

# define library postfix so that different builds will produce
# distinguished libraries
set(CMAKE_RELEASE_POSTFIX "_r" CACHE string "Release postfix")
set(CMAKE_DEBUG_POSTFIX "_d" CACHE string "Debug postfix")
set(CMAKE_RELWITHDEBINFO_POSTFIX "_rd" CACHE string
  "Release with debug info postfix")
set(CMAKE_MINSIZEREL_POSTFIX "_mr" CACHE string
  "Minimum size release postfix")
mark_as_advanced(CMAKE_RELEASE_POSTFIX)
mark_as_advanced(CMAKE_DEBUG_POSTFIX)
mark_as_advanced(CMAKE_RELWITHDEBINFO_POSTFIX)
mark_as_advanced(CMAKE_MINSIZEREL_POSTFIX)

# ctest
include(CTest)
enable_testing()

# for rdm (emacs)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# for grouping into folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
  message(FATAL_ERROR "Please select another Build Directory ! (and give it a
  clever name, like bin_Visual2012_64bits/)")
endif()
if(CMAKE_SOURCE_DIR MATCHES " ")
  message( "Your Source Directory contains spaces. If you experience problems
    when compiling, this can be the cause." )
endif()
if(CMAKE_BINARY_DIR MATCHES " ")
  message("Your Build Directory contains spaces. If you experience problems
  when compiling, this can be the cause.")
endif()

# Compile external dependencies
add_subdirectory(external)

# On Visual 2005 and above, this module can set the debug working directory
cmake_policy(SET CMP0026 OLD)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/external/rpavlik-cmake-modules-fe2273")
include(CreateLaunchers)
include(MSVCMultipleProcessCompile) # /MP

###############################################################################

include_directories(
  external/glfw-3.2.1/include/GLFW/
  external/glm-0.9.7.1/
  external/glew-1.13.0/include/
  external/Simple-OpenGL-Image-Library/include/
  external/tinyxml2/include/
  external/tinyobjloader/include/
  .
  )

set(ALL_LIBS
  ${OPENGL_LIBRARY}
  glfw
  GLEW_1130
  SOIL
  TINYXML2
  )

add_definitions(
  -DTW_STATIC
  -DTW_NO_LIB_PRAGMA
  -DTW_NO_DIRECT3D
  -DGLEW_STATIC
  -D_CRT_SECURE_NO_WARNINGS
  )

###############################################################################
# Project 
add_executable(exempted_project
  exemplar/main.cpp
  common/camera.cpp
  common/camera.h
  common/model.cpp
  common/model.h
  common/shader.cpp
  common/shader.h
  common/texture.cpp
  common/texture.h
  common/util.cpp
  common/util.h
  exemplar/StandardShading.fragmentshader
  exemplar/StandardShading.vertexshader
  exemplar/depth.fragmentshader
  exemplar/depth.vertexshader
  )
target_link_libraries(exempted_project
  ${ALL_LIBS}
  )
# Xcode and Visual working directories
set_target_properties(exempted_project
  PROPERTIES
  XCODE_ATTRIBUTE_CONFIGURATION_BUILD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/exemplar/"
  PROJECT_LABEL "Exempted project on graphics"
  FOLDER "Exercise"
  )
create_target_launcher(exempted_project WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/exemplar/")
create_default_target_launcher(exempted_project WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/exemplar/")

###############################################################################

SOURCE_GROUP(sources REGULAR_EXPRESSION ".*/common/.*cpp$" )
SOURCE_GROUP(headers REGULAR_EXPRESSION ".*/common/.*h$" )
SOURCE_GROUP(shaders REGULAR_EXPRESSION ".*/exemplar/.*shader$" )
SOURCE_GROUP(obj REGULAR_EXPRESSION ".*/exemplar/.*obj$" )

###############################################################################
# copy
if (NOT ${CMAKE_GENERATOR} MATCHES "Xcode" )

  add_custom_command(
    TARGET exempted_project POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/exempted_project${CMAKE_EXECUTABLE_SUFFIX}" "${CMAKE_CURRENT_SOURCE_DIR}/exemplar/"
    )

elseif (${CMAKE_GENERATOR} MATCHES "Xcode" )

endif (NOT ${CMAKE_GENERATOR} MATCHES "Xcode" )
