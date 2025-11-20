# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# icupkg tool - Extract or package ICU data

add_executable(icupkg icupkg.cpp)

target_include_directories(icupkg PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(icupkg PRIVATE icutu)
else()
    target_link_libraries(icupkg PRIVATE icutu_static)
endif()

install(TARGETS icupkg
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
