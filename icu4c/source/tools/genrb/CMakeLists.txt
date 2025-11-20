# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# genrb tool - Generate resource bundle data

file(GLOB GENRB_SOURCES "*.cpp" "*.c")

# Remove derb.cpp - it's a separate tool
list(FILTER GENRB_SOURCES EXCLUDE REGEX "derb\\.cpp$")

# Set language property for C files
foreach(src ${GENRB_SOURCES})
    if(src MATCHES "\\.c$")
        set_source_files_properties(${src} PROPERTIES LANGUAGE C)
    endif()
endforeach()

add_executable(genrb ${GENRB_SOURCES})

target_include_directories(genrb PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    ${CMAKE_CURRENT_SOURCE_DIR}/../../io
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(genrb PRIVATE icuuc icui18n icuio icutu)
else()
    target_link_libraries(genrb PRIVATE icuuc_static icui18n_static icuio_static icutu_static)
endif()

# derb tool - Resource bundle decompiler (separate executable)
file(GLOB DERB_SOURCES "derb.cpp")
add_executable(derb ${DERB_SOURCES})

target_include_directories(derb PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    ${CMAKE_CURRENT_SOURCE_DIR}/../../io
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(derb PRIVATE icuuc icui18n icuio icutu)
else()
    target_link_libraries(derb PRIVATE icuuc_static icui18n_static icuio_static icutu_static)
endif()

install(TARGETS derb
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)

install(TARGETS genrb
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
