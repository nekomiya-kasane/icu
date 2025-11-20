# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Tool Utility Library (libicutu)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" TOOLUTIL_SOURCES)
list(FILTER TOOLUTIL_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM TOOLUTIL_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Public headers
file(GLOB TOOLUTIL_PUBLIC_HEADERS 
    "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
)

# Library name
set(TOOLUTIL_LIBRARY_NAME icutu)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${TOOLUTIL_LIBRARY_NAME} SHARED ${TOOLUTIL_SOURCES})
    
    set_target_properties(${TOOLUTIL_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${TOOLUTIL_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
    )
    
    target_compile_definitions(${TOOLUTIL_LIBRARY_NAME} PRIVATE
        U_TOOLUTIL_IMPLEMENTATION
    )
    
    target_include_directories(${TOOLUTIL_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    )
    
    # Link dependencies
    target_link_libraries(${TOOLUTIL_LIBRARY_NAME} PUBLIC icuuc icui18n)
    
    # Install shared library
    install(TARGETS ${TOOLUTIL_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::tu ALIAS ${TOOLUTIL_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${TOOLUTIL_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${TOOLUTIL_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${TOOLUTIL_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        U_TOOLUTIL_IMPLEMENTATION
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    )
    
    # Link dependencies
    target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icuuc_static icui18n_static)
    
    # Install static library
    install(TARGETS ${STATIC_LIBRARY_NAME}
        EXPORT ICUTargets
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::tu_static ALIAS ${STATIC_LIBRARY_NAME})
endif()
