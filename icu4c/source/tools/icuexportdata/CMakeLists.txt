# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# icuexportdata - ICU data export tool

# Source files
set(ICUEXPORTDATA_SOURCES
    icuexportdata.cpp
)

# Create executable
add_executable(icuexportdata ${ICUEXPORTDATA_SOURCES})

# Set C++ standard
set_target_properties(icuexportdata PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

# Link libraries
target_link_libraries(icuexportdata
    PRIVATE
        icuuc
        icui18n
        icutu
)

# Include directories
target_include_directories(icuexportdata
    PRIVATE
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/i18n
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/toolutil
)

# Install
install(TARGETS icuexportdata
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
