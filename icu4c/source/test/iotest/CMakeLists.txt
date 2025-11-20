# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# iotest - ICU I/O tests

# Collect all source files
file(GLOB IOTEST_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
)

# Create executable
add_executable(iotest ${IOTEST_SOURCES})

# Set C++ standard
set_target_properties(iotest PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Link libraries
target_link_libraries(iotest
    PRIVATE
        icuuc
        icui18n
        icuio
        icutest
)

# Include directories
target_include_directories(iotest
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/i18n
        ${CMAKE_SOURCE_DIR}/icu4c/source/io
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/ctestfw
)

# Add test
add_test(NAME iotest COMMAND iotest)

# Set test environment (need ICU DLLs in PATH on Windows)
if(WIN32)
    set_tests_properties(iotest PROPERTIES
        ENVIRONMENT "PATH=$<TARGET_FILE_DIR:icuuc>;$<TARGET_FILE_DIR:icui18n>;$<TARGET_FILE_DIR:icuio>;$<TARGET_FILE_DIR:icudt>;$ENV{PATH}"
    )
endif()

# Install
install(TARGETS iotest
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
