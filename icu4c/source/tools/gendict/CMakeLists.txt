# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gendict tool - Generate dictionary data

add_executable(gendict gendict.cpp)

target_include_directories(gendict PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gendict PRIVATE icuuc icui18n icutu)
else()
    target_link_libraries(gendict PRIVATE icuuc_static icui18n_static icutu_static)
endif()

install(TARGETS gendict
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
