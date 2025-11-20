# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gensprep tool - Generate string prep data

file(GLOB GENSPREP_SOURCES "*.c")

# Set language property for C files
set_source_files_properties(${GENSPREP_SOURCES} PROPERTIES LANGUAGE C)

add_executable(gensprep ${GENSPREP_SOURCES})

target_include_directories(gensprep PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gensprep PRIVATE icutu)
else()
    target_link_libraries(gensprep PRIVATE icutu_static)
endif()

install(TARGETS gensprep
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
