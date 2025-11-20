# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# icuswap - ICU data endianness converter

# Source files
set(ICUSWAP_SOURCES
    icuswap.cpp
)

# Create executable
add_executable(icuswap ${ICUSWAP_SOURCES})

# Set C++ standard
set_target_properties(icuswap PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Link libraries
target_link_libraries(icuswap
    PRIVATE
        icuuc
        icutu
)

# Include directories
target_include_directories(icuswap
    PRIVATE
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/toolutil
)

# Install
install(TARGETS icuswap
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
