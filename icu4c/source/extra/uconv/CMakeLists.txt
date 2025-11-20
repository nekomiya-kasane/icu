# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# uconv - Universal character set converter

# Source files
set(UCONV_SOURCES
    uconv.cpp
    uwmsg.c
)

# Create executable
add_executable(uconv ${UCONV_SOURCES})

# Set C++ standard
set_target_properties(uconv PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Link libraries
target_link_libraries(uconv
    PRIVATE
        icuuc
        icui18n
        icuio
        icutu
)

# Include directories
target_include_directories(uconv
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/i18n
        ${CMAKE_SOURCE_DIR}/icu4c/source/io
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/toolutil
)

# Install
install(TARGETS uconv
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
