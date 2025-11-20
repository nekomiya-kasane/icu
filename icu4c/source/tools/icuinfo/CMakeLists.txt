# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# icuinfo tool - Display ICU information

# icuinfo executable
add_executable(icuinfo icuinfo.cpp)

set_target_properties(icuinfo PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(icuinfo PRIVATE
    ${CMAKE_SOURCE_DIR}/icu4c/source/common
    ${CMAKE_SOURCE_DIR}/icu4c/source/i18n
    ${CMAKE_SOURCE_DIR}/icu4c/source/tools/toolutil
    ${CMAKE_SOURCE_DIR}/icu4c/source/tools/ctestfw
)

target_link_libraries(icuinfo PRIVATE
    icuuc
    icui18n
    icutu
    icutest
)

install(TARGETS icuinfo
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)

# testplug - ICU plugin test library
if(ICU_ENABLE_PLUGINS)
    add_library(testplug MODULE testplug.c)
    
    set_target_properties(testplug PROPERTIES
        C_STANDARD 11
        C_STANDARD_REQUIRED ON
        PREFIX ""
    )
    
    target_include_directories(testplug PRIVATE
        ${CMAKE_SOURCE_DIR}/icu4c/source/common
        ${CMAKE_SOURCE_DIR}/icu4c/source/tools/toolutil
    )
    
    target_link_libraries(testplug PRIVATE
        icuuc
        icutu
    )
    
    install(TARGETS testplug
        LIBRARY DESTINATION ${ICU_INSTALL_LIBDIR}
    )
endif()
