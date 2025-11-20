# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# pkgdata tool - Package data for ICU

file(GLOB PKGDATA_SOURCES "*.cpp" "*.c")

# Set language property for C files
foreach(src ${PKGDATA_SOURCES})
    if(src MATCHES "\\.c$")
        set_source_files_properties(${src} PROPERTIES LANGUAGE C)
    endif()
endforeach()

add_executable(pkgdata ${PKGDATA_SOURCES})

target_include_directories(pkgdata PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(pkgdata PRIVATE icutu)
else()
    target_link_libraries(pkgdata PRIVATE icutu_static)
endif()

install(TARGETS pkgdata
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
