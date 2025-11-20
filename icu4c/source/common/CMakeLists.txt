# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Common Library (libicuuc)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" COMMON_SOURCES)
list(FILTER COMMON_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM COMMON_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Public headers
file(GLOB COMMON_PUBLIC_HEADERS 
    "${CMAKE_CURRENT_SOURCE_DIR}/unicode/*.h"
)

# Library name
set(COMMON_LIBRARY_NAME icuuc)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    add_library(${COMMON_LIBRARY_NAME} SHARED ${COMMON_SOURCES})
    
    set_target_properties(${COMMON_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${COMMON_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
        PUBLIC_HEADER "${COMMON_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${COMMON_LIBRARY_NAME} PRIVATE
        U_COMMON_IMPLEMENTATION
    )
    
    target_include_directories(${COMMON_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
    )
    
    # Link to stubdata first (must come before system libraries)
    # On Windows, the data library is named icudt, not icudata
    if(WIN32)
        target_link_libraries(${COMMON_LIBRARY_NAME} PUBLIC icudt)
    else()
        target_link_libraries(${COMMON_LIBRARY_NAME} PUBLIC icudata)
    endif()
    
    # Link dependencies
    if(WIN32)
        target_link_libraries(${COMMON_LIBRARY_NAME} PRIVATE advapi32)
    elseif(UNIX AND NOT APPLE)
        target_link_libraries(${COMMON_LIBRARY_NAME} PRIVATE pthread dl m)
    elseif(APPLE)
        target_link_libraries(${COMMON_LIBRARY_NAME} PRIVATE pthread m)
    endif()
    
    # Install shared library
    install(TARGETS ${COMMON_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${ICU_INSTALL_INCLUDEDIR}/unicode
    )
    
    # Create alias for consistent naming
    add_library(ICU::uc ALIAS ${COMMON_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${COMMON_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${COMMON_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${COMMON_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
        PUBLIC_HEADER "${COMMON_PUBLIC_HEADERS}"
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        U_COMMON_IMPLEMENTATION
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/unicode>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
    )
    
    # Link to stubdata first (must come before system libraries)
    # On Windows, the data library is named icudt, not icudata
    if(WIN32)
        target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icudt_static)
    else()
        target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icudata_static)
    endif()
    
    # Link dependencies
    if(WIN32)
        target_link_libraries(${STATIC_LIBRARY_NAME} PRIVATE advapi32)
    elseif(UNIX AND NOT APPLE)
        target_link_libraries(${STATIC_LIBRARY_NAME} PRIVATE pthread dl m)
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
    add_library(ICU::uc_static ALIAS ${STATIC_LIBRARY_NAME})
endif()

# Export targets
install(EXPORT ICUTargets
    FILE ICUTargets.cmake
    NAMESPACE ICU::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ICU
)
