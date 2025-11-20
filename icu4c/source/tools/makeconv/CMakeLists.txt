# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# makeconv tool - Compile converter tables

file(GLOB MAKECONV_SOURCES "*.c" "*.cpp")
list(FILTER MAKECONV_SOURCES EXCLUDE REGEX ".*test.*")

# Set language property for C files
foreach(src ${MAKECONV_SOURCES})
    if(src MATCHES "\\.c$")
        set_source_files_properties(${src} PROPERTIES LANGUAGE C)
    endif()
endforeach()

add_executable(makeconv ${MAKECONV_SOURCES})

target_include_directories(makeconv PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(makeconv PRIVATE icutu)
else()
    target_link_libraries(makeconv PRIVATE icutu_static)
endif()

install(TARGETS makeconv
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
