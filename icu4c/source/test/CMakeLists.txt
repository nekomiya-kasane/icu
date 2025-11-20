# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU Tests

# Build ctestfw library first (test framework, already built in tools)
# It's already built in tools/ctestfw

# Build intltest (C++ tests)
add_subdirectory(intltest)

# Build cintltst (C tests)
add_subdirectory(cintltst)

# Build iotest (I/O tests)
if(TARGET icuio)
    add_subdirectory(iotest)
endif()
