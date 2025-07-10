#################################################################################
##
## Thor C++ Library
## Copyright (c) 2011-2022 Jan Haller
##
## This software is provided 'as-is', without any express or implied
## warranty. In no event will the authors be held liable for any damages
## arising from the use of this software.
##
## Permission is granted to anyone to use this software for any purpose,
## including commercial applications, and to alter it and redistribute it
## freely, subject to the following restrictions:
##
## 1. The origin of this software must not be misrepresented; you must not
##    claim that you wrote the original software. If you use this software
##    in a product, an acknowledgment in the product documentation would be
##    appreciated but is not required.
##
## 2. Altered source versions must be plainly marked as such, and must not be
##    misrepresented as being the original software.
##
## 3. This notice may not be removed or altered from any source distribution.
##
#################################################################################

# Directory Thor


cmake_minimum_required(VERSION 3.16)
project(thor LANGUAGES CXX)

include(FetchContent)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ----------------------------
# Options
# ----------------------------
option(THOR_SHARED_LIBS "Build shared libraries" OFF)
option(THOR_BUILD_EXAMPLES "Build example projects" OFF)
option(THOR_BUILD_DOC "Generate documentation with Doxygen" OFF)
option(THOR_STATIC_STD_LIBS "Use statically linked standard/runtime libraries (must match SFML)" OFF)

# ----------------------------
# Fetch SFML (2.6.2)
# ----------------------------
FetchContent_Declare(
  sfml
  GIT_REPOSITORY https://github.com/SFML/SFML.git
  GIT_TAG 2.6.2
)
FetchContent_MakeAvailable(sfml)

# ----------------------------
# Aurora (local submodule)
# ----------------------------
add_library(aurora INTERFACE)
target_include_directories(aurora INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/extlibs/aurora/include)

# ----------------------------
# Thor library
# ----------------------------
file(GLOB_RECURSE THOR_SOURCES CONFIGURE_DEPENDS
  src/*.cpp
)

add_library(thor ${THOR_SOURCES})
add_library(thor::thor ALIAS thor)

target_include_directories(thor
  PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_link_libraries(thor
  PUBLIC
    sfml-graphics
    sfml-window
    sfml-system
    sfml-audio
    aurora
)

target_compile_definitions(thor PRIVATE THOR_EXPORTS)

if (THOR_STATIC_STD_LIBS AND MSVC)
  foreach(flag_var
    CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
    CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE)
    string(REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
  endforeach()
elseif(THOR_STATIC_STD_LIBS AND CMAKE_COMPILER_IS_GNUCXX)
  target_compile_options(thor PRIVATE -static-libgcc -static-libstdc++)
endif()

set_target_properties(thor PROPERTIES
  WINDOWS_EXPORT_ALL_SYMBOLS ON
  POSITION_INDEPENDENT_CODE ON
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

if (THOR_SHARED_LIBS)
  set_target_properties(thor PROPERTIES
    BUILD_SHARED_LIBS ON
  )
endif()

# ----------------------------
# Examples
# ----------------------------
if(THOR_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

# ----------------------------
# Documentation
# ----------------------------
if(THOR_BUILD_DOC)
  find_package(Doxygen REQUIRED)
  add_subdirectory(doc)
endif()

# ----------------------------
# Install
# ----------------------------
install(DIRECTORY include/ DESTINATION include)
install(FILES License.txt DESTINATION share/doc/thor)
install(DIRECTORY extlibs/aurora/include/ DESTINATION include)

