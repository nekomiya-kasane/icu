# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Tools

# Build toolutil library first (required by other tools)
add_subdirectory(toolutil)

# Build ctestfw library (test framework)
add_subdirectory(ctestfw)

# Build individual tools
add_subdirectory(genbrk)
add_subdirectory(genccode)
add_subdirectory(gencfu)
add_subdirectory(gencmn)
add_subdirectory(gencnval)
add_subdirectory(gendict)
add_subdirectory(gennorm2)
add_subdirectory(genrb)
add_subdirectory(gensprep)
add_subdirectory(gentest)
add_subdirectory(icuinfo)
add_subdirectory(icupkg)
add_subdirectory(icuswap)
add_subdirectory(makeconv)
add_subdirectory(pkgdata)

# Data export tool
add_subdirectory(icuexportdata)

# Optional: escapesrc tool
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/escapesrc/CMakeLists.txt")
    add_subdirectory(escapesrc)
endif()
