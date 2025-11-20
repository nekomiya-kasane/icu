# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gencmn tool - Generate common data file

set_source_files_properties(gencmn.c PROPERTIES LANGUAGE C)

add_executable(gencmn gencmn.c)

target_include_directories(gencmn PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gencmn PRIVATE icutu)
else()
    target_link_libraries(gencmn PRIVATE icutu_static)
endif()

install(TARGETS gencmn
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
