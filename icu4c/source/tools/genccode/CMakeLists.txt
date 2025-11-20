# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# genccode tool - Generate C code from data files

set_source_files_properties(genccode.c PROPERTIES LANGUAGE C)

add_executable(genccode genccode.c)

target_include_directories(genccode PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(genccode PRIVATE icutu)
else()
    target_link_libraries(genccode PRIVATE icutu_static)
endif()

install(TARGETS genccode
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
