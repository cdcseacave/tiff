# ----------------------------------------------------------------------------
#  CMake file for libtiff. See root CMakeLists.txt
#
# ----------------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8)

# Find dependencies:
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMake/Modules)

if(NOT TIFF_LIBRARY)
	set(TIFF_LIBRARY "tiff")
endif()
project(${TIFF_LIBRARY})

# fix CMake IntDir variable
if(MSVC AND "${MSVC_VERSION}" STRGREATER "1500")
    #set(COTIRE_INTDIR "$(Platform)/cotire")
    set(CMAKE_CFG_INTDIR "$(Platform)/$(Configuration)")
endif()

# Define helper functions and macros.
INCLUDE(CMake/Utils.cmake)
INCLUDE(CMake/Cotire.cmake)

# Init sesion with macros defined in Utils.cmake
GetOperatingSystemArchitectureBitness(SYSTEM)
ComposePackageLibSuffix()
ConfigCompilerAndLinker()

include(CheckFunctionExists)
include(CheckIncludeFile)

check_include_file(assert.h HAVE_ASSERT_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(io.h HAVE_IO_H)
check_function_exists(jbg_newlen HAVE_JBG_NEWLEN)
check_function_exists(mmap HAVE_MMAP)
check_include_file(search.h HAVE_SEARCH_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)

if(WIN32 AND NOT HAVE_WINRT)
	set(USE_WIN32_FILEIO 1)
endif()

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/tif_config.h.cmakein"
               "${CMAKE_CURRENT_BINARY_DIR}/tif_config.h" @ONLY)

include_directories("${CMAKE_CURRENT_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}")

# needed packages
find_package(ZLIB REQUIRED)
include_directories(${ZLIB_INCLUDE_DIR})

# list sources files
set(LIBRARY_FILES_C
    tif_aux.c
    tif_close.c
    tif_codec.c
    tif_color.c
    tif_compress.c
    tif_dir.c
    tif_dirinfo.c
    tif_dirread.c
    tif_dirwrite.c
    tif_dumpmode.c
    tif_error.c
    tif_extension.c
    tif_fax3.c
    tif_fax3sm.c
    tif_flush.c
    tif_getimage.c
    tif_jbig.c
    tif_jpeg_12.c
    tif_jpeg.c
    tif_luv.c
    tif_lzma.c
    tif_lzw.c
    tif_next.c
    tif_ojpeg.c
    tif_open.c
    tif_packbits.c
    tif_pixarlog.c
    tif_predict.c
    tif_print.c
    tif_read.c
    tif_strip.c
    tif_swab.c
    tif_thunder.c
    tif_tile.c
    tif_version.c
    tif_warning.c
    tif_write.c
    tif_zip.c
    tif_stream.cxx
    )
set(LIBRARY_FILES_H
    # t4.h
    # uvcode.h
    tif_dir.h
    tif_fax3.h
    tiff.h
    tiffio.h
    tiffiop.h
    tiffvers.h
    tif_predict.h
    tiffio.hxx
    "${CMAKE_CURRENT_BINARY_DIR}/tif_config.h"
    )

if(WIN32 AND NOT HAVE_WINRT)
	list(APPEND LIBRARY_FILES_C tif_win32.c)
else()
	list(APPEND LIBRARY_FILES_C tif_unix.c)
endif()

if(UNIX AND (CMAKE_COMPILER_IS_GNUCXX OR CV_ICC))
	set(BUILD_SHARED_LIBS ON)
endif()

cxx_library_with_type(${TIFF_LIBRARY} "" "" "${cxx_default}" ${LIBRARY_FILES_C} ${LIBRARY_FILES_H})
target_link_libraries(${TIFF_LIBRARY} ${ZLIB_LIBRARIES})

set_target_properties(${TIFF_LIBRARY}
    PROPERTIES
	LINKER_LANGUAGE CPP
    OUTPUT_NAME "${TIFF_LIBRARY}"
    )

INSTALL(FILES ${LIBRARY_FILES_H} DESTINATION "include")
INSTALL(TARGETS ${TIFF_LIBRARY} DESTINATION "lib")
