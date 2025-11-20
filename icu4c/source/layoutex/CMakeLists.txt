# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Layout Extensions Library (libiculx)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" LAYOUTEX_SOURCES)
list(FILTER LAYOUTEX_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM LAYOUTEX_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Public headers
file(GLOB LAYOUTEX_PUBLIC_HEADERS 
    "${CMAKE_CURRENT_SOURCE_DIR}/unicode/*.h"
)

# Library name
set(LAYOUTEX_LIBRARY_NAME iculx)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${LAYOUTEX_LIBRARY_NAME} SHARED ${LAYOUTEX_SOURCES})
    
    set_target_properties(${LAYOUTEX_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${LAYOUTEX_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
        PUBLIC_HEADER "${LAYOUTEX_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${LAYOUTEX_LIBRARY_NAME} PRIVATE
        U_LAYOUTEX_IMPLEMENTATION
    )
    
    if(WIN32)
        target_compile_definitions(${LAYOUTEX_LIBRARY_NAME} PRIVATE
            "U_IMPORT="
            "U_EXPORT=__declspec(dllexport)"
        )
    endif()
    
    target_include_directories(${LAYOUTEX_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../i18n
    )
    
    # Link dependencies
    target_link_libraries(${LAYOUTEX_LIBRARY_NAME} PUBLIC icuuc icui18n)
    
    # Install shared library
    install(TARGETS ${LAYOUTEX_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::lx ALIAS ${LAYOUTEX_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${LAYOUTEX_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${LAYOUTEX_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${LAYOUTEX_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
        PUBLIC_HEADER "${LAYOUTEX_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        U_LAYOUTEX_IMPLEMENTATION
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../i18n
    )
    
    # Link dependencies
    target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icuuc_static icui18n_static)
    
    # Install static library
    install(TARGETS ${STATIC_LIBRARY_NAME}
        EXPORT ICUTargets
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::lx_static ALIAS ${STATIC_LIBRARY_NAME})
endif()
