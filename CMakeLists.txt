cmake_minimum_required(VERSION 3.1)

project(chdr C)

set(CHDR_VERSION_MAJOR 0)
set(CHDR_VERSION_MINOR 0)

option(BUILD_SHARED_LIBS "Build libchdr also as a shared library" ON)
option(WITH_SYSTEM_FLAC "Use system provided FLAC library" OFF)
option(WITH_SYSTEM_ZLIB "Use system provided zlib library" OFF)

if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -O3 -flto")
endif()

include(FindPkgConfig)

if (NOT DEFINED CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX "/usr/local")
endif()
if (NOT DEFINED CMAKE_INSTALL_LIBDIR)
  set(CMAKE_INSTALL_LIBDIR "lib")
endif()
if (NOT DEFINED CMAKE_INSTALL_INCLUDEDIR)
  set(CMAKE_INSTALL_INCLUDEDIR "include")
endif()

#--------------------------------------------------
# dependencies
#--------------------------------------------------

# crypto
set(CRYPTO_SOURCES
  deps/crypto/md5.c
  deps/crypto/sha1.c)

if (WITH_SYSTEM_FLAC)
  pkg_check_modules(FLAC REQUIRED flac)
  list(APPEND PLATFORM_INCLUDES ${FLAC_INCLUDE_DIRS})
  list(APPEND PLATFORM_LIBS ${FLAC_LIBRARIES})
else()
  set(WITH_OGG OFF CACHE BOOL "ogg support (default: test for libogg)")
  set(BUILD_SHARED_LIBS_SAVED "${BUILD_SHARED_LIBS}")
  set(BUILD_SHARED_LIBS OFF)
  add_subdirectory(deps/flac-1.3.3)
  set(BUILD_SHARED_LIBS "${BUILD_SHARED_LIBS_SAVED}")
  list(APPEND CHDR_LIBS FLAC)
endif()

add_library(crypto-static STATIC ${CRYPTO_SOURCES})
list(APPEND CHDR_INCLUDES deps/crypto)
list(APPEND CHDR_LIBS crypto-static)

# lzma
set(LZMA_SOURCES
  deps/lzma-16.04/C/Alloc.c
  deps/lzma-16.04/C/Bra86.c
  deps/lzma-16.04/C/Bra.c
  deps/lzma-16.04/C/BraIA64.c
  deps/lzma-16.04/C/CpuArch.c
  deps/lzma-16.04/C/LzFind.c
  deps/lzma-16.04/C/Lzma86Dec.c
  deps/lzma-16.04/C/Lzma86Enc.c
  deps/lzma-16.04/C/LzmaDec.c
  deps/lzma-16.04/C/LzmaEnc.c
)

add_library(lzma-static STATIC ${LZMA_SOURCES})
target_compile_options(lzma-static PRIVATE -D_7ZIP_ST)
list(APPEND CHDR_INCLUDES deps/lzma-16.04/C)
list(APPEND CHDR_LIBS lzma-static)

# zlib
if (WITH_SYSTEM_ZLIB)
  pkg_check_modules(ZLIB REQUIRED zlib)
  list(APPEND PLATFORM_INCLUDES ${ZLIB_INCLUDE_DIRS})
  list(APPEND PLATFORM_LIBS ${ZLIB_LIBRARIES})
else()
  add_subdirectory(deps/zlib-1.2.11 EXCLUDE_FROM_ALL)
  list(APPEND CHDR_INCLUDES deps/zlib-1.2.11 ${CMAKE_CURRENT_BINARY_DIR}/deps/zlib-1.2.11)
  list(APPEND CHDR_LIBS zlibstatic)
endif()

#--------------------------------------------------
# chdr
#--------------------------------------------------

set(CHDR_SOURCES
  src/libchdr_bitstream.c
  src/libchdr_cdrom.c
  src/libchdr_chd.c
  src/libchdr_flac.c
  src/libchdr_huffman.c
)

list(APPEND CHDR_INCLUDES include ${CMAKE_CURRENT_BINARY_DIR}/include)

add_library(chdr-static STATIC ${CHDR_SOURCES})
target_include_directories(chdr-static PRIVATE ${CHDR_INCLUDES} ${PLATFORM_INCLUDES})
target_compile_definitions(chdr-static PRIVATE ${CHDR_DEFS})
target_link_libraries(chdr-static ${CHDR_LIBS} ${PLATFORM_LIBS})

if (BUILD_SHARED_LIBS)
  set(LIBS ${PLATFORM_LIBS})
  list(TRANSFORM LIBS PREPEND "-l")
  list(JOIN LIBS " " LIBS)

  add_library(chdr SHARED ${CHDR_SOURCES})
  target_include_directories(chdr PRIVATE ${CHDR_INCLUDES} ${PLATFORM_INCLUDES})
  target_compile_definitions(chdr PRIVATE ${CHDR_DEFS})
  target_link_libraries(chdr ${CHDR_LIBS} ${PLATFORM_LIBS})
  target_link_options(chdr PRIVATE -Wl,--version-script ${CMAKE_SOURCE_DIR}/src/link.T -Wl,--no-undefined)

  set_target_properties(chdr PROPERTIES PUBLIC_HEADER "include/libchdr/bitstream.h;include/libchdr/cdrom.h;include/libchdr/chd.h;include/libchdr/coretypes.h;include/libchdr/flac.h;include/libchdr/huffman.h")
  set_target_properties(chdr PROPERTIES VERSION "${CHDR_VERSION_MAJOR}.${CHDR_VERSION_MINOR}")

  install(TARGETS chdr
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/libchdr"
  )

  configure_file(pkg-config.pc.in ${CMAKE_BINARY_DIR}/libchdr.pc @ONLY)
  install(FILES ${CMAKE_BINARY_DIR}/libchdr.pc DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endif()

