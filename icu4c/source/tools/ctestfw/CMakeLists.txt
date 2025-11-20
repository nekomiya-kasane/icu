# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU C Test Framework Library (libicutest)

# Read source files from sources.txt
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/sources.txt" CTESTFW_SOURCES)
list(FILTER CTESTFW_SOURCES EXCLUDE REGEX "^$")
list(TRANSFORM CTESTFW_SOURCES PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Separate C and C++ files
set(CTESTFW_C_SOURCES)
set(CTESTFW_CXX_SOURCES)
foreach(src ${CTESTFW_SOURCES})
    if(src MATCHES "\\.c$")
        list(APPEND CTESTFW_C_SOURCES ${src})
    else()
        list(APPEND CTESTFW_CXX_SOURCES ${src})
    endif()
endforeach()

# Create object library for C files to ensure they are compiled as C
if(CTESTFW_C_SOURCES)
    add_library(ctestfw_c_objects OBJECT ${CTESTFW_C_SOURCES})
    set_target_properties(ctestfw_c_objects PROPERTIES
        LINKER_LANGUAGE C
        POSITION_INDEPENDENT_CODE ON
    )
    set_source_files_properties(${CTESTFW_C_SOURCES} PROPERTIES LANGUAGE C)
    target_include_directories(ctestfw_c_objects PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/../../common
        ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
        ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
    )
    target_compile_definitions(ctestfw_c_objects PRIVATE
        T_CTEST_IMPLEMENTATION
        _CRT_NONSTDC_NO_WARNINGS
        _CRT_SECURE_NO_WARNINGS
    )
endif()

# Library name
set(CTESTFW_LIBRARY_NAME icutest)

# Create shared library if enabled
if(ICU_BUILD_SHARED_LIBS)
    if(CTESTFW_C_SOURCES)
        add_library(${CTESTFW_LIBRARY_NAME} SHARED ${CTESTFW_CXX_SOURCES} $<TARGET_OBJECTS:ctestfw_c_objects>)
    else()
        add_library(${CTESTFW_LIBRARY_NAME} SHARED ${CTESTFW_SOURCES})
    endif()
    
    set_target_properties(${CTESTFW_LIBRARY_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${ICU_VERSION_MAJOR}
        OUTPUT_NAME ${CTESTFW_LIBRARY_NAME}${ICU_VERSION_SUFFIX}
    )
    
    target_compile_definitions(${CTESTFW_LIBRARY_NAME} PRIVATE
        T_CTEST_IMPLEMENTATION
    )
    
    target_include_directories(${CTESTFW_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
            ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
    )
    
    # Link dependencies
    target_link_libraries(${CTESTFW_LIBRARY_NAME} PUBLIC icuuc icui18n icutu)
    
    # Install shared library
    install(TARGETS ${CTESTFW_LIBRARY_NAME}
        EXPORT ICUTargets
        RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::test ALIAS ${CTESTFW_LIBRARY_NAME})
endif()

# Create static library if enabled
if(ICU_BUILD_STATIC_LIBS)
    set(STATIC_LIBRARY_NAME ${CTESTFW_LIBRARY_NAME}_static)
    
    add_library(${STATIC_LIBRARY_NAME} STATIC ${CTESTFW_SOURCES})
    
    set_target_properties(${STATIC_LIBRARY_NAME} PROPERTIES
        OUTPUT_NAME ${CTESTFW_LIBRARY_NAME}${ICU_VERSION_SUFFIX}s
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PUBLIC
        U_STATIC_IMPLEMENTATION
    )
    
    target_compile_definitions(${STATIC_LIBRARY_NAME} PRIVATE
        T_CTEST_IMPLEMENTATION
    )
    
    target_include_directories(${STATIC_LIBRARY_NAME}
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/../../common
            ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
            ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
    )
    
    # Link dependencies
    target_link_libraries(${STATIC_LIBRARY_NAME} PUBLIC icuuc_static icui18n_static icutu_static)
    
    # Install static library
    install(TARGETS ${STATIC_LIBRARY_NAME}
        EXPORT ICUTargets
        ARCHIVE DESTINATION ${ICU_INSTALL_LIBDIR}
    )
    
    # Create alias for consistent naming
    add_library(ICU::test_static ALIAS ${STATIC_LIBRARY_NAME})
endif()
