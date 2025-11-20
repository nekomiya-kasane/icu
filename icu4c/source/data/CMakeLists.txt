# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Data Build
# This implements full data building from source, similar to makedata.mak

message(STATUS "")
message(STATUS "=== ICU Data Build Configuration ===")

# Options
option(ICU_DATA_BUILD_FROM_SOURCE "Build ICU data from source files" ON)
option(ICU_DATA_FILTER_FILE "Path to data filter file" "")

# Data configuration
set(U_ICUDATA_NAME "icudt${ICU_VERSION_MAJOR}")
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)|(i386)|(i686)")
    set(U_ICUDATA_ENDIAN_SUFFIX "l")  # Little endian
else()
    set(U_ICUDATA_ENDIAN_SUFFIX "b")  # Big endian
endif()
set(ICUPKG "${U_ICUDATA_NAME}${U_ICUDATA_ENDIAN_SUFFIX}")

# Directories
set(ICUOUT "${CMAKE_CURRENT_BINARY_DIR}/out")
set(ICUBLD "${ICUOUT}/build")
set(ICUBLD_PKG "${ICUBLD}/${ICUPKG}")
set(ICUTMP "${ICUOUT}/tmp")
set(ICUSRCDATA "${CMAKE_CURRENT_SOURCE_DIR}")

# Create output directories
file(MAKE_DIRECTORY "${ICUOUT}")
file(MAKE_DIRECTORY "${ICUBLD}")
file(MAKE_DIRECTORY "${ICUBLD_PKG}")
file(MAKE_DIRECTORY "${ICUTMP}")

# Check for prebuilt data archive
set(ICUDATA_SOURCE_ARCHIVE "")
file(GLOB ICUDATA_ARCHIVES "${ICUSRCDATA}/in/${ICUPKG}.dat")
if(ICUDATA_ARCHIVES)
    list(GET ICUDATA_ARCHIVES 0 ICUDATA_SOURCE_ARCHIVE)
    message(STATUS "Found prebuilt data archive: ${ICUDATA_SOURCE_ARCHIVE}")
    set(ICU_DATA_BUILD_FROM_SOURCE OFF)
else()
    # Check for big endian version
    file(GLOB ICUDATA_ARCHIVES_BE "${ICUSRCDATA}/in/${U_ICUDATA_NAME}b.dat")
    if(ICUDATA_ARCHIVES_BE)
        list(GET ICUDATA_ARCHIVES_BE 0 ICUDATA_ARCHIVE_BE)
        message(STATUS "Found big-endian data archive: ${ICUDATA_ARCHIVE_BE}")
        message(STATUS "Will convert to ${U_ICUDATA_ENDIAN_SUFFIX} endian")
        set(ICUDATA_SOURCE_ARCHIVE "${ICUTMP}/${ICUPKG}.dat")
        set(ICUDATA_NEEDS_CONVERSION TRUE)
    endif()
endif()

if(ICU_DATA_BUILD_FROM_SOURCE AND NOT ICUDATA_SOURCE_ARCHIVE)
    message(STATUS "Building ICU data from source files...")
    
    # Find Python 3
    find_package(Python3 COMPONENTS Interpreter)
    if(NOT Python3_FOUND)
        message(FATAL_ERROR "Python 3 is required to build ICU data from source. "
                           "Either install Python 3 or provide a prebuilt data file in data/in/")
    endif()
    
    message(STATUS "Python 3: ${Python3_EXECUTABLE}")
    
    # Data building requires tools to be built first
    # These dependencies are handled by the tools subdirectory
    
    # Build command arguments
    # Note: tool_dir should point to the build directory where tools are compiled
    # For single-config generators (Ninja, Makefile), tools are directly in tool subdirs
    # For multi-config generators (Visual Studio), they are in <tool>/<CONFIG>/ subdirs
    set(DATA_BUILD_ARGS
        --mode windows-exec
        --src_dir "${ICUSRCDATA}"
        --tool_dir "${CMAKE_BINARY_DIR}/icu4c/source/tools"
        --out_dir "${ICUBLD_PKG}"
        --tmp_dir "${ICUTMP}"
    )
    
    # Set tool_cfg based on generator type
    # For single-config generators (Ninja, Makefile): use "." to indicate tools are in tool dir
    # For multi-config generators (Visual Studio): use config name like "Debug" or "Release"
    if(CMAKE_CONFIGURATION_TYPES)
        list(APPEND DATA_BUILD_ARGS --tool_cfg "$<CONFIG>")
    else()
        list(APPEND DATA_BUILD_ARGS --tool_cfg ".")
    endif()
    
    # Add filter file if specified
    if(ICU_DATA_FILTER_FILE)
        list(APPEND DATA_BUILD_ARGS --filter_file "${ICU_DATA_FILTER_FILE}")
    endif()
    
    # Add verbose flag for Debug builds
    list(APPEND DATA_BUILD_ARGS "$<$<CONFIG:Debug>:--verbose>")
    
    # Create a custom target for building data
    # Note: Tools need ICU DLLs in PATH to run
    if(WIN32)
        # On Windows, add directories containing ICU DLLs to PATH
        file(TO_NATIVE_PATH "${CMAKE_BINARY_DIR}/icu4c/source/common" ICU_COMMON_DIR)
        file(TO_NATIVE_PATH "${CMAKE_BINARY_DIR}/icu4c/source/i18n" ICU_I18N_DIR)
        file(TO_NATIVE_PATH "${CMAKE_BINARY_DIR}/icu4c/source/io" ICU_IO_DIR)
        file(TO_NATIVE_PATH "${CMAKE_BINARY_DIR}/icu4c/source/stubdata" ICU_DATA_DIR)
        set(ICU_PATH "${ICU_COMMON_DIR};${ICU_I18N_DIR};${ICU_IO_DIR};${ICU_DATA_DIR};$ENV{PATH}")
    else()
        set(ICU_PATH "$ENV{PATH}")
    endif()
    
    add_custom_command(
        OUTPUT "${ICUBLD_PKG}.timestamp"
        COMMAND ${CMAKE_COMMAND} -E env 
                "PYTHONPATH=${CMAKE_CURRENT_SOURCE_DIR}/../python"
                "PATH=${ICU_PATH}"
                ${Python3_EXECUTABLE} -B -m icutools.databuilder
                ${DATA_BUILD_ARGS}
        COMMAND ${CMAKE_COMMAND} -E touch "${ICUBLD_PKG}.timestamp"
        DEPENDS genrb makeconv gencnval icupkg pkgdata icuuc icui18n icuio icudt
        WORKING_DIRECTORY "${ICUSRCDATA}"
        COMMENT "Building ICU data from source..."
        VERBATIM
    )
    
    add_custom_target(icu_data_build ALL
        DEPENDS "${ICUBLD_PKG}.timestamp"
    )
    
    set(ICU_DATA_BUILT_FROM_SOURCE TRUE PARENT_SCOPE)
    
elseif(ICUDATA_SOURCE_ARCHIVE)
    if(ICUDATA_NEEDS_CONVERSION)
        # Convert endianness using icupkg
        add_custom_command(
            OUTPUT "${ICUDATA_SOURCE_ARCHIVE}"
            COMMAND icupkg -t${U_ICUDATA_ENDIAN_SUFFIX} 
                    "${ICUDATA_ARCHIVE_BE}" 
                    "${ICUDATA_SOURCE_ARCHIVE}"
            DEPENDS icupkg "${ICUDATA_ARCHIVE_BE}"
            COMMENT "Converting data archive to ${U_ICUDATA_ENDIAN_SUFFIX} endian..."
        )
        add_custom_target(icu_data_convert ALL
            DEPENDS "${ICUDATA_SOURCE_ARCHIVE}"
        )
    endif()
    
    message(STATUS "Using prebuilt data archive")
    set(ICU_DATA_BUILT_FROM_SOURCE FALSE PARENT_SCOPE)
    
    # Install the data archive
    install(FILES "${ICUDATA_SOURCE_ARCHIVE}"
        DESTINATION ${ICU_INSTALL_DATADIR}
        RENAME "${ICUPKG}.dat"
    )
else()
    message(WARNING "No data source found. Building with stub data only.")
    message(WARNING "")
    message(WARNING "To build with full ICU data:")
    message(WARNING "  Option 1: Download prebuilt data")
    message(WARNING "    - Get icudt${ICU_VERSION_MAJOR}l.dat from https://github.com/unicode-org/icu/releases")
    message(WARNING "    - Place in icu4c/source/data/in/")
    message(WARNING "  Option 2: Build from source")
    message(WARNING "    - Install Python 3")
    message(WARNING "    - Set ICU_DATA_BUILD_FROM_SOURCE=ON")
    message(WARNING "")
    set(ICU_DATA_BUILT_FROM_SOURCE FALSE PARENT_SCOPE)
endif()

message(STATUS "Data package name: ${ICUPKG}")
message(STATUS "Data output directory: ${ICUBLD_PKG}")
message(STATUS "=== End ICU Data Configuration ===")
message(STATUS "")