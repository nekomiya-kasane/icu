# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# gencfu tool - Generate confusable data

add_executable(gencfu gencfu.cpp)

target_include_directories(gencfu PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../../common
    ${CMAKE_CURRENT_SOURCE_DIR}/../../i18n
    ${CMAKE_CURRENT_SOURCE_DIR}/../toolutil
)

if(ICU_BUILD_SHARED_LIBS)
    target_link_libraries(gencfu PRIVATE icuuc icui18n icutu)
else()
    target_link_libraries(gencfu PRIVATE icuuc_static icui18n_static icutu_static)
endif()

install(TARGETS gencfu
    RUNTIME DESTINATION ${ICU_INSTALL_BINDIR}
)
