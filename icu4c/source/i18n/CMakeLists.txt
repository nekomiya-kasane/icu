# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Internationalization Library (libicui18n)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" I18N_SOURCES)
list(FILTER I18N_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM I18N_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Public headers
file(GLOB I18N_PUBLIC_HEADERS 
    "${CMAKE_CURRENT_SOURCE_DIR}/unicode/*.h"
)

# Library name
set(I18N_LIBRARY_NAME icui18n)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${I18N_LIBRARY_NAME} SHARED ${I18N_SOURCES})
    
    set_target_properties(${I18N_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${I18N_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
        PUBLIC_HEADER "${I18N_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${I18N_LIBRARY_NAME} PRIVATE
        U_I18N_IMPLEMENTATION
    )
    
    target_include_directories(${I18N_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
    )
    
    # Link dependencies
    target_link_libraries(${I18N_LIBRARY_NAME} PUBLIC icuuc)
    
    if(WIN32)
        target_link_libraries(${I18N_LIBRARY_NAME} PRIVATE advapi32)
    elseif(UNIX AND NOT APPLE)
        target_link_libraries(${I18N_LIBRARY_NAME} PRIVATE pthread m)
    elseif(APPLE)
        target_link_libraries(${I18N_LIBRARY_NAME} PRIVATE pthread m)
    endif()
    
    # Install shared library
    install(TARGETS ${I18N_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::i18n ALIAS ${I18N_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${I18N_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${I18N_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${I18N_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
        PUBLIC_HEADER "${I18N_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        U_I18N_IMPLEMENTATION
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/../common
    )
    
    # Link dependencies
    target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icuuc_static)
    
    if(WIN32)
        target_link_libraries(${STATIC_LIBRARY_NAME} PRIVATE advapi32)
    elseif(UNIX AND NOT APPLE)
        target_link_libraries(${STATIC_LIBRARY_NAME} PRIVATE pthread m)
    elseif(APPLE)
        target_link_libraries(${STATIC_LIBRARY_NAME} PRIVATE pthread m)
    endif()
    
    # Install static library
    install(TARGETS ${STATIC_LIBRARY_NAME}
        EXPORT ICUTargets
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::i18n_static ALIAS ${STATIC_LIBRARY_NAME})
endif()
