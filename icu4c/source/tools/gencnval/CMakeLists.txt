# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gencnval tool - Generate converter alias data

set_source_files_properties(gencnval.c PROPERTIES LANGUAGE C)

add_executable(gencnval gencnval.c)

target_include_directories(gencnval PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gencnval PRIVATE icutu)
else()
    target_link_libraries(gencnval PRIVATE icutu_static)
endif()

install(TARGETS gencnval
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
