# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gennorm2 tool - Generate normalization data

file(GLOB GENNORM2_SOURCES "*.cpp")

add_executable(gennorm2 ${GENNORM2_SOURCES})

target_include_directories(gennorm2 PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gennorm2 PRIVATE icuuc icui18n icutu)
else()
    target_link_libraries(gennorm2 PRIVATE icuuc_static icui18n_static icutu_static)
endif()

install(TARGETS gennorm2
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
