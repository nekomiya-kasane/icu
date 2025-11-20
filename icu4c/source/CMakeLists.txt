# Copyright (C) 2016 and later: Unicode, Inc. and others.
# License & terms of use: http://www.unicode.org/copyright.html

# ICU4C Source Directory

# Add subdirectories in dependency order
add_subdirectory(stubdata)
if(ICU_BUILD_DATA)
    add_subdirectory(data)
endif()
add_subdirectory(common)
add_subdirectory(i18n)
add_subdirectory(io)

# layoutex requires HarfBuzz and icu-le-hb
if(ICU_BUILD_LAYOUTEX)
    add_subdirectory(layoutex)
endif()

# Add tools if enabled
if(ICU_BUILD_TOOLS)
    add_subdirectory(tools)
endif()

# Add extra tools (uconv, etc.)
if(ICU_BUILD_EXTRAS)
    add_subdirectory(extra)
endif()

# Add tests if enabled
if(ICU_BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()

# Add samples if enabled
if(ICU_BUILD_SAMPLES)
    add_subdirectory(samples)
endif()
