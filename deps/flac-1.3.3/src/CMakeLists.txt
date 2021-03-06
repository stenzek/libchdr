cmake_minimum_required(VERSION 3.0)

option(ENABLE_64_BIT_WORDS "Set FLAC__BYTES_PER_WORD to 8 (4 is the default)" OFF)
option(WITH_XMMS "Build XMMS plugin" OFF)
option(WITH_UTILS "Build utils" OFF)
option(WITH_FLAC "Build flac command-line tool" OFF)
option(WITH_METAFLAC "Build metaflac command-line tool" OFF)

check_include_file("iconv.h" HAVE_ICONV)

add_subdirectory("libFLAC")

if(BUILD_CXXLIBS)
    add_subdirectory("libFLAC++")
endif()

if(WITH_METAFLAC OR WITH_FLAC OR WITH_UTILS OR WITH_XMMS OR BUILD_TESTING)
    add_subdirectory("share")
endif()

if(WITH_METAFLAC)
add_subdirectory("metaflac")
endif()

if(WITH_FLAC)
    add_subdirectory("flac")
endif()

if(WITH_UTILS)
    add_subdirectory("utils")
endif()

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
