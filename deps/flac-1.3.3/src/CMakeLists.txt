cmake_minimum_required(VERSION 3.0)

option(ENABLE_64_BIT_WORDS "Set FLAC__BYTES_PER_WORD to 8 (4 is the default)" OFF)
option(WITH_XMMS "Build XMMS plugin" OFF)

check_include_file("iconv.h" HAVE_ICONV)

add_subdirectory("libFLAC")
if(BUILD_CXXLIBS)
    add_subdirectory("libFLAC++")
endif()
add_subdirectory("share")
#add_subdirectory("flac")
add_subdirectory("metaflac")
add_subdirectory("utils")

if(WITH_XMMS)
    add_subdirectory("plugin_common")
    add_subdirectory("plugin_xmms")
endif()

if(BUILD_TESTING)
    add_subdirectory("test_libs_common")
    add_subdirectory("test_libFLAC")
    if(BUILD_CXXLIBS)
        add_subdirectory("test_libFLAC++")
    endif()
    add_subdirectory("test_grabbag")
    add_subdirectory("test_seeking")
    add_subdirectory("test_streams")
endif()