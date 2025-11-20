# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# intltest - ICU C++ API tests

# Collect all source files
file(GLOB INTLTEST_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
)

# Create executable
add_executable(intltest ${INTLTEST_SOURCES})

# Set C++ standard
set_target_properties(intltest PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Link libraries
target_link_libraries(intltest
    PRIVATE
        icuuc
        icui18n
        icuio
        icutest
)

# Include directories
target_include_directories(intltest
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/i18n
        ${CMAKE_SOURCE_DIR}/icu4c/source/io
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/ctestfw
)

# Add test
add_test(NAME intltest COMMAND intltest)

# Set test environment (need ICU DLLs in PATH on Windows)
if(WIN32)
    set_tests_properties(intltest PROPERTIES
        ENVIRONMENT "PATH=$<TARGET_FILE_DIR:icuuc>;$<TARGET_FILE_DIR:icui18n>;$<TARGET_FILE_DIR:icuio>;$<TARGET_FILE_DIR:icudt>;$ENV{PATH}"
    )
endif()

# Install
install(TARGETS intltest
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
