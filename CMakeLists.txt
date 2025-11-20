# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

cmake_minimum_required(VERSION 3.28)

# Policy settings for better compatibility
if(POLICY CMP0091)
    cmake_policy(SET CMP0091 NEW)  # MSVC runtime library flags
endif()

# Read ICU version from uvernum.h
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/icu4c/source/common/unicode/uvernum.h" UVERNUM_H_CONTENT)
string(REGEX MATCH "#define U_ICU_VERSION_MAJOR_NUM ([0-9]+)" _ ${UVERNUM_H_CONTENT})
set(ICU_VERSION_MAJOR ${CMAKE_MATCH_1})
string(REGEX MATCH "#define U_ICU_VERSION_MINOR_NUM ([0-9]+)" _ ${UVERNUM_H_CONTENT})
set(ICU_VERSION_MINOR ${CMAKE_MATCH_1})
string(REGEX MATCH "#define U_ICU_VERSION_PATCHLEVEL_NUM ([0-9]+)" _ ${UVERNUM_H_CONTENT})
set(ICU_VERSION_PATCH ${CMAKE_MATCH_1})

project(ICU 
    VERSION ${ICU_VERSION_MAJOR}.${ICU_VERSION_MINOR}.${ICU_VERSION_PATCH}
    LANGUAGES NONE
    DESCRIPTION "International Components for Unicode"
    HOMEPAGE_URL "https://icu.unicode.org/"
)

# Enable C and C++ languages after project() to allow better compiler detection
enable_language(C)
enable_language(CXX)

# Require C++23
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Require C11
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Build options
option(ICU_BUILD_SHARED_LIBS "Build shared libraries" ON)
option(ICU_BUILD_STATIC_LIBS "Build static libraries" OFF)
option(ICU_BUILD_TOOLS "Build ICU tools" ON)
option(ICU_BUILD_EXTRAS "Build extra tools (uconv, etc.)" ON)
option(ICU_BUILD_TESTS "Build ICU tests" OFF)
option(ICU_BUILD_SAMPLES "Build ICU samples" OFF)
option(ICU_BUILD_LAYOUTEX "Build layoutex library (requires HarfBuzz)" OFF)
option(ICU_ENABLE_PLUGINS "Enable ICU plugins" ON)
option(ICU_ENABLE_DYLOAD "Enable dynamic loading" ON)
option(ICU_ENABLE_TRACING "Enable tracing" OFF)
option(ICU_DATA_PACKAGING "Data packaging mode (files/archive/library/static)" "archive")

# Set default build type
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Platform detection
if(WIN32)
    set(ICU_PLATFORM_WINDOWS TRUE)
elseif(APPLE)
    set(ICU_PLATFORM_APPLE TRUE)
elseif(UNIX)
    set(ICU_PLATFORM_UNIX TRUE)
endif()

# todo: this is bad
add_compile_definitions(U_HAVE_CHAR16_T=0)

# Disable dependency scanning for MSVC with Ninja to avoid /showIncludes
if(MSVC AND CMAKE_GENERATOR STREQUAL "Ninja")
    set(CMAKE_C_DEPENDS_USE_COMPILER FALSE)
    set(CMAKE_CXX_DEPENDS_USE_COMPILER FALSE)
    set(CMAKE_DEPENDS_IN_PROJECT_ONLY ON)
    # Remove /showIncludes flag that Ninja adds
    string(REPLACE "/showIncludes" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
    string(REPLACE "/showIncludes" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    string(REPLACE "/showIncludes" "" CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
    string(REPLACE "/showIncludes" "" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
endif()

# Compiler flags
if(MSVC)
    add_compile_options(/W3 /utf-8)
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_WARNINGS)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(/Od /Zi)
    else()
        add_compile_options(/O2)
    endif()
else()
    add_compile_options(-Wall -Wextra -pedantic)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-g -O0)
    else()
        add_compile_options(-O3)
    endif()
endif()

# ICU configuration
set(ICU_DATA_DIR "${CMAKE_INSTALL_PREFIX}/share/icu/${ICU_VERSION_MAJOR}.${ICU_VERSION_MINOR}")
set(ICU_LIBRARY_SUFFIX "")

# Version suffix for library names
set(ICU_VERSION_SUFFIX "${ICU_VERSION_MAJOR}")

# Global include directories
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/icu4c/source/common
    ${CMAKE_CURRENT_SOURCE_DIR}/icu4c/source/i18n
    ${CMAKE_CURRENT_BINARY_DIR}/icu4c/source/common
)

# Installation directories
include(GNUInstallDirs)
set(ICU_INSTALL_BINDIR ${CMAKE_INSTALL_BINDIR})
set(ICU_INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR})
set(ICU_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR})
set(ICU_INSTALL_DATADIR ${CMAKE_INSTALL_DATADIR}/icu/${ICU_VERSION_MAJOR}.${ICU_VERSION_MINOR})

# Export compile commands for IDE integration
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Add subdirectories
add_subdirectory(icu4c/source)

# Package configuration
include(CMakePackageConfigHelpers)

# Create ICUConfig.cmake
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/ICUConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/ICUConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ICU
)

# Create ICUConfigVersion.cmake
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/ICUConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Install CMake config files
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/ICUConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/ICUConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ICU
)

# Print configuration summary
message(STATUS "")
message(STATUS "ICU ${PROJECT_VERSION} Configuration Summary:")
message(STATUS "  CMake version:        ${CMAKE_VERSION}")
message(STATUS "  C++ Standard:         C++${CMAKE_CXX_STANDARD}")
message(STATUS "  C Standard:           C${CMAKE_C_STANDARD}")
message(STATUS "  Build type:           ${CMAKE_BUILD_TYPE}")
message(STATUS "  Build shared libs:    ${ICU_BUILD_SHARED_LIBS}")
message(STATUS "  Build static libs:    ${ICU_BUILD_STATIC_LIBS}")
message(STATUS "  Build tools:          ${ICU_BUILD_TOOLS}")
message(STATUS "  Build tests:          ${ICU_BUILD_TESTS}")
message(STATUS "  Build samples:        ${ICU_BUILD_SAMPLES}")
message(STATUS "  Enable plugins:       ${ICU_ENABLE_PLUGINS}")
message(STATUS "  Enable dyload:        ${ICU_ENABLE_DYLOAD}")
message(STATUS "  Data packaging:       ${ICU_DATA_PACKAGING}")
message(STATUS "  Install prefix:       ${CMAKE_INSTALL_PREFIX}")
message(STATUS "")
