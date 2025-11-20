# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Data Library (libicudata)
# This builds a stub data library. For full data support, use the data
# directory build or link against a pre-built data library.

set(STUBDATA_SOURCES
    stubdata.cpp
)

# Library name
# On Windows, ICU uses 'icudt' for data library, not 'icudata'
if(WIN32)
    set(DATA_LIBRARY_NAME icudt)
else()
    set(DATA_LIBRARY_NAME icudata)
endif()

# Data directory path
set(ICU_DATA_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../data")

# Option to build with actual data (requires data build tools)
option(ICU_DATA_BUILDTOOL "Build ICU data using build tools" OFF)

# Data packaging mode
set(ICU_DATA_PACKAGING_MODE "archive" CACHE STRING "Data packaging mode: files, archive, library, static")
set_property(CACHE ICU_DATA_PACKAGING_MODE PROPERTY STRINGS "files" "archive" "library" "static")

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${DATA_LIBRARY_NAME} SHARED ${STUBDATA_SOURCES})
    
    set_target_properties(${DATA_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${DATA_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
    )
    
    # On Windows, import library should have version suffix for consistency
    if(WIN32)
        set_target_properties(${DATA_LIBRARY_NAME} PROPERTIES
            ARCHIVE_OUTPUT_NAME ${DATA_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
        )
    endif()
    
    target_compile_definitions(${DATA_LIBRARY_NAME} PRIVATE
        STUBDATA_BUILD
    )
    
    target_include_directories(${DATA_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../common/unicode
    )
    
    # Install shared library
    install(TARGETS ${DATA_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::data ALIAS ${DATA_LIBRARY_NAME})
endif()

# Install ICU data files if they exist
if(EXISTS "${ICU_DATA_SOURCE_DIR}")
    # Install data directory structure
    install(DIRECTORY "${ICU_DATA_SOURCE_DIR}/"
        DESTINATION ${ICU_INSTALL_DATADIR}
        FILES_MATCHING
        PATTERN "*.txt"
        PATTERN "*.res"
        PATTERN "*.nrm"
        PATTERN "*.icu"
        PATTERN "*.brk"
        PATTERN "*.cnv"
        PATTERN "*.dat"
        PATTERN "*.spp"
        PATTERN "Makefile*" EXCLUDE
        PATTERN "*.in" EXCLUDE
        PATTERN "*.vcxproj*" EXCLUDE
        PATTERN "*.xml" EXCLUDE
        PATTERN "*.py" EXCLUDE
    )
    
    message(STATUS "ICU data files will be installed to: ${ICU_INSTALL_DATADIR}")
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${DATA_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${STUBDATA_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${DATA_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        STUBDATA_BUILD
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../common/unicode
    )
    
    # Install static library
    install(TARGETS ${STATIC_LIBRARY_NAME}
        EXPORT ICUTargets
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::data_static ALIAS ${STATIC_LIBRARY_NAME})
endif()

# Configure data library to know where to find data at runtime
if(TARGET ${DATA_LIBRARY_NAME})
    target_compile_definitions(${DATA_LIBRARY_NAME} PRIVATE
        ICU_DATA_DIR="${ICU_INSTALL_DATADIR}"
    )
endif()

if(TARGET ${DATA_LIBRARY_NAME}_static)
    target_compile_definitions(${DATA_LIBRARY_NAME}_static PRIVATE
        ICU_DATA_DIR="${ICU_INSTALL_DATADIR}"
    )
endif()

# Print data configuration summary
message(STATUS "")
message(STATUS "ICU Data Configuration:")
message(STATUS "  Data packaging mode:  ${ICU_DATA_PACKAGING_MODE}")
message(STATUS "  Data source dir:      ${ICU_DATA_SOURCE_DIR}")
message(STATUS "  Data install dir:     ${ICU_INSTALL_DATADIR}")
message(STATUS "  Build data tools:     ${ICU_DATA_BUILDTOOL}")
message(STATUS "")
