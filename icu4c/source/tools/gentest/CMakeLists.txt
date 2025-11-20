# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gentest tool - Generate test data

set(GENTEST_SOURCES gentest.c genres32.c)
set_source_files_properties(${GENTEST_SOURCES} PROPERTIES LANGUAGE C)

add_executable(gentest ${GENTEST_SOURCES})

target_include_directories(gentest PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gentest PRIVATE icutu)
else()
    target_link_libraries(gentest PRIVATE icutu_static)
endif()

install(TARGETS gentest
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
