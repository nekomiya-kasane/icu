# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU I/O Library (libicuio)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" IO_SOURCES)
list(FILTER IO_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM IO_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Public headers
file(GLOB IO_PUBLIC_HEADERS 
    "${CMAKE_CURRENT_SOURCE_DIR}/unicode/*.h"
)

# Library name
set(IO_LIBRARY_NAME icuio)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${IO_LIBRARY_NAME} SHARED ${IO_SOURCES})
    
    set_target_properties(${IO_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${IO_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
        PUBLIC_HEADER "${IO_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${IO_LIBRARY_NAME} PRIVATE
        U_IO_IMPLEMENTATION
    )
    
    target_include_directories(${IO_LIBRARY_NAME}
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
    target_link_libraries(${IO_LIBRARY_NAME} PUBLIC icuuc icui18n)
    
    # Install shared library
    install(TARGETS ${IO_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::io ALIAS ${IO_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${IO_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${IO_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${IO_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
        PUBLIC_HEADER "${IO_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        U_IO_IMPLEMENTATION
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
    add_library(ICU::io_static ALIAS ${STATIC_LIBRARY_NAME})
endif()
