# LuaDist:
# Original modified and reused for LuaDist project by Peter Kapec
# Please note that the package source code is licensed under its own license.
# Notes:
#	Following script is the original build script distributed with PCRE source code and
#	was modified to suit LuaDist (search for "#LuaDist: ..." comments )

# LuaDist:
project ( pcre C CXX )
cmake_minimum_required ( VERSION 2.8 )
include ( cmake/dist.cmake )

###########################################################################################

# CMakeLists.txt
#
#
# This file allows building PCRE with the CMake configuration and build
# tool. Download CMake in source or binary form from http://www.cmake.org/
#
# Original listfile by Christian Ehrlicher <Ch.Ehrlicher@gmx.de>
# Refined and expanded by Daniel Richard G. <skunk@iSKUNK.ORG>
# 2007-09-14 mod by Sheri so 7.4 supported configuration options can be entered
# 2007-09-19 Adjusted by PH to retain previous default settings
# 2007-12-26 (a) On UNIX, use names libpcre instead of just pcre
#            (b) Ensure pcretest and pcregrep link with the local library,
#                not a previously-installed one.
#            (c) Add PCRE_SUPPORT_LIBREADLINE, PCRE_SUPPORT_LIBZ, and
#                PCRE_SUPPORT_LIBBZ2.
# 2008-01-20 Brought up to date to include several new features by Christian
#            Ehrlicher.
# 2008-01-22 Sheri added options for backward compatibility of library names when
#            building with minGW:
#            if "ON", NON_STANDARD_LIB_PREFIX causes shared libraries to
#            be built without "lib" as prefix. (The libraries will be named pcre.dll,
#            pcreposix.dll and pcrecpp.dll).
#            if "ON", NON_STANDARD_LIB_SUFFIX causes shared libraries to
#            be built with suffix of "-0.dll". (The libraries will be named
#            libpcre-0.dll, libpcreposix-0.dll and libpcrecpp-0.dll - same names
#            built by default with Configure and Make.
# 2008-01-23 PH removed the automatic build of pcredemo.

set ( CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake )
# for FindReadline.cmake

# external packages
find_package ( BZip2 )
find_package ( ZLIB )
find_package ( Readline )

# Configuration checks

include ( CheckIncludeFile )
include ( CheckIncludeFileCXX )
include ( CheckFunctionExists )
include ( CheckTypeSize )

check_include_file ( dirent.h HAVE_DIRENT_H )
check_include_file ( sys/stat.h HAVE_SYS_STAT_H )
check_include_file ( sys/types.h HAVE_SYS_TYPES_H )
check_include_file ( unistd.h HAVE_UNISTD_H )
check_include_file ( windows.h HAVE_WINDOWS_H )

check_include_file_cxx ( type_traits.h HAVE_TYPE_TRAITS_H )
check_include_file_cxx ( bits/type_traits.h HAVE_BITS_TYPE_TRAITS_H )

check_function_exists ( bcopy HAVE_BCOPY )
check_function_exists ( memmove HAVE_MEMMOVE )
check_function_exists ( strerror HAVE_STRERROR )
check_function_exists ( strtoll HAVE_STRTOLL )
check_function_exists ( strtoq HAVE_STRTOQ )
check_function_exists ( _strtoi64 HAVE__STRTOI64 )

check_type_size ( "long long" LONG_LONG )
check_type_size ( "unsigned long long" UNSIGNED_LONG_LONG )

# User-configurable options
#
# (Note: CMakeSetup displays these in alphabetical order, regardless of
# the order we use here)

# LuaDist: BUILD_SHARED_LIBS ON (was OFF)
set ( BUILD_SHARED_LIBS ON CACHE BOOL "Build shared libraries instead of static ones." )

option ( PCRE_BUILD_PCRECPP "Build the PCRE C++ library (pcrecpp)." ON )

set ( PCRE_EBCDIC OFF CACHE BOOL "Use EBCDIC coding instead of ASCII. (This is rarely used outside of mainframe systems)" )

set ( PCRE_LINK_SIZE "2" CACHE STRING "Internal link size (2, 3 or 4 allowed). See LINK_SIZE in config.h.in for details." )

set ( PCRE_MATCH_LIMIT "10000000" CACHE STRING "Default limit on internal looping. See MATCH_LIMIT in config.h.in for details." )

set ( PCRE_MATCH_LIMIT_RECURSION "MATCH_LIMIT" CACHE STRING "Default limit on internal recursion. See MATCH_LIMIT_RECURSION in config.h.in for details." )

# LuaDist: PCRE_NEWLINE ON (was LF)
set ( PCRE_NEWLINE "ANYCRLF" CACHE STRING "What to recognize as a newline (one of CR, LF, CRLF, ANY, ANYCRLF)." )

set ( PCRE_NO_RECURSE OFF CACHE BOOL "If ON, then don't use stack recursion when matching. See NO_RECURSE in config.h.in for details." )

set ( PCRE_POSIX_MALLOC_THRESHOLD "10" CACHE STRING "Threshold for malloc() usage. See POSIX_MALLOC_THRESHOLD in config.h.in for details." )

# LuaDist: PCRE_SUPPORT_UNICODE_PROPERTIES ON (was OFF)
set ( PCRE_SUPPORT_UNICODE_PROPERTIES ON CACHE BOOL "Enable support for Unicode properties. (If set, UTF-8 support will be enabled as well)" )

# LuaDist: PCRE_SUPPORT_UTF8 ON (was OFF)
set ( PCRE_SUPPORT_UTF8 ON CACHE BOOL "Enable support for the Unicode UTF-8 encoding." )

# LuaDist: PCRE_SUPPORT_BSR_ANYCRLF ON (was OFF)
set ( PCRE_SUPPORT_BSR_ANYCRLF ON CACHE BOOL "ON=Backslash-R matches only LF CR and CRLF, OFF=Backslash-R matches all Unicode Linebreaks" )

if ( MINGW )
  option ( NON_STANDARD_LIB_PREFIX "ON=Shared libraries built in mingw will be named pcre.dll, etc., instead of libpcre.dll, etc." 
    OFF )

  option ( NON_STANDARD_LIB_SUFFIX "ON=Shared libraries built in mingw will be named libpcre-0.dll, etc., instead of libpcre.dll, etc." 
    OFF )
endif ( MINGW )

# bzip2 lib
if ( BZIP2_FOUND )
  option ( PCRE_SUPPORT_LIBBZ2 "Enable support for linking pcregrep with libbz2." 
    ON )
endif ( BZIP2_FOUND )
if ( PCRE_SUPPORT_LIBBZ2 )
  include_directories ( ${BZIP2_INCLUDE_DIR} )
endif ( PCRE_SUPPORT_LIBBZ2 )

# zlib
if ( ZLIB_FOUND )
  option ( PCRE_SUPPORT_LIBZ "Enable support for linking pcregrep with libz." ON )
endif ( ZLIB_FOUND )
if ( PCRE_SUPPORT_LIBZ )
  include_directories ( ${ZLIB_INCLUDE_DIR} )
endif ( PCRE_SUPPORT_LIBZ )

# readline lib
if ( READLINE_FOUND )
  option ( PCRE_SUPPORT_LIBREADLINE "Enable support for linking pcretest with libreadline." 
    ON )
endif ( READLINE_FOUND )
if ( PCRE_SUPPORT_LIBREADLINE )
  include_directories ( ${READLINE_INCLUDE_DIR} )
endif ( PCRE_SUPPORT_LIBREADLINE )

# Prepare build configuration

set ( pcre_have_type_traits 0 )
set ( pcre_have_bits_type_traits 0 )

if ( HAVE_TYPE_TRAITS_H )
  set ( pcre_have_type_traits 1 )
endif ( HAVE_TYPE_TRAITS_H )

if ( HAVE_BITS_TYPE_TRAITS_H )
  set ( pcre_have_bits_type_traits 1 )
endif ( HAVE_BITS_TYPE_TRAITS_H )

set ( pcre_have_long_long 0 )
set ( pcre_have_ulong_long 0 )

if ( HAVE_LONG_LONG )
  set ( pcre_have_long_long 1 )
endif ( HAVE_LONG_LONG )

if ( HAVE_UNSIGNED_LONG_LONG )
  set ( pcre_have_ulong_long 1 )
endif ( HAVE_UNSIGNED_LONG_LONG )

if ( NOT BUILD_SHARED_LIBS )
  set ( PCRE_STATIC 1 )
endif ( NOT BUILD_SHARED_LIBS )

if ( PCRE_SUPPORT_BSR_ANYCRLF )
  set ( BSR_ANYCRLF 1 )
endif ( PCRE_SUPPORT_BSR_ANYCRLF )

if ( PCRE_SUPPORT_UTF8 OR PCRE_SUPPORT_UNICODE_PROPERTIES )
  set ( SUPPORT_UTF8 1 )
endif ( PCRE_SUPPORT_UTF8 OR PCRE_SUPPORT_UNICODE_PROPERTIES )

if ( PCRE_SUPPORT_UNICODE_PROPERTIES )
  set ( SUPPORT_UCP 1 )
endif ( PCRE_SUPPORT_UNICODE_PROPERTIES )

if ( PCRE_SUPPORT_LIBREADLINE )
  set ( SUPPORT_LIBREADLINE 1 )
  set ( PCRETEST_LIBS ${READLINE_LIBRARY} )
endif ( PCRE_SUPPORT_LIBREADLINE )

if ( PCRE_SUPPORT_LIBZ )
  set ( SUPPORT_LIBZ 1 )
  set ( PCREGREP_LIBS ${PCREGREP_LIBS} ${ZLIB_LIBRARIES} )
endif ( PCRE_SUPPORT_LIBZ )

if ( PCRE_SUPPORT_LIBBZ2 )
  set ( SUPPORT_LIBBZ2 1 )
  set ( PCREGREP_LIBS ${PCREGREP_LIBS} ${BZIP2_LIBRARIES} )
endif ( PCRE_SUPPORT_LIBBZ2 )

set ( NEWLINE "" )

if ( PCRE_NEWLINE STREQUAL "LF" )
  set ( NEWLINE "10" )
endif ( PCRE_NEWLINE STREQUAL "LF" )
if ( PCRE_NEWLINE STREQUAL "CR" )
  set ( NEWLINE "13" )
endif ( PCRE_NEWLINE STREQUAL "CR" )
if ( PCRE_NEWLINE STREQUAL "CRLF" )
  set ( NEWLINE "3338" )
endif ( PCRE_NEWLINE STREQUAL "CRLF" )
if ( PCRE_NEWLINE STREQUAL "ANY" )
  set ( NEWLINE "-1" )
endif ( PCRE_NEWLINE STREQUAL "ANY" )
if ( PCRE_NEWLINE STREQUAL "ANYCRLF" )
  set ( NEWLINE "-2" )
endif ( PCRE_NEWLINE STREQUAL "ANYCRLF" )

if ( NEWLINE STREQUAL "" )
  message ( FATAL_ERROR "The PCRE_NEWLINE variable must be set to one of the following values: \"LF\", \"CR\", \"CRLF\", \"ANY\", \"ANYCRLF\"." )
endif ( NEWLINE STREQUAL "" )

if ( PCRE_EBCDIC )
  set ( EBCDIC 1 )
endif ( PCRE_EBCDIC )

if ( PCRE_NO_RECURSE )
  set ( NO_RECURSE 1 )
endif ( PCRE_NO_RECURSE )

# Output files

configure_file ( config-cmake.h.in ${CMAKE_BINARY_DIR}/config.h @ONLY )

configure_file ( pcre.h.generic ${CMAKE_BINARY_DIR}/pcre.h COPYONLY )

# What about pcre-config and libpcre.pc?

if ( PCRE_BUILD_PCRECPP )
  configure_file ( pcre_stringpiece.h.in ${CMAKE_BINARY_DIR}/pcre_stringpiece.h @ONLY )

  configure_file ( pcrecpparg.h.in ${CMAKE_BINARY_DIR}/pcrecpparg.h @ONLY )
endif ( PCRE_BUILD_PCRECPP )

# Character table generation

option ( PCRE_REBUILD_CHARTABLES "Rebuild char tables" OFF )
if ( PCRE_REBUILD_CHARTABLES )
  add_executable ( dftables dftables.c )

  get_target_property ( DFTABLES_EXE dftables LOCATION )

  add_custom_command ( COMMENT "Generating character tables (pcre_chartables.c) for current locale" 
    DEPENDS dftables COMMAND ${DFTABLES_EXE} ARGS ${CMAKE_BINARY_DIR}/pcre_chartables.c 
    OUTPUT ${CMAKE_BINARY_DIR}/pcre_chartables.c )
else ( PCRE_REBUILD_CHARTABLES )
  configure_file ( ${CMAKE_SOURCE_DIR}/pcre_chartables.c.dist ${CMAKE_BINARY_DIR}/pcre_chartables.c 
    COPYONLY )
endif ( PCRE_REBUILD_CHARTABLES )

# Source code

set ( PCRE_HEADERS ${CMAKE_BINARY_DIR}/pcre.h )

set ( PCRE_SOURCES ${CMAKE_BINARY_DIR}/pcre_chartables.c pcre_compile.c pcre_config.c 
  pcre_dfa_exec.c pcre_exec.c pcre_fullinfo.c pcre_get.c pcre_globals.c pcre_info.c 
  pcre_newline.c pcre_maketables.c pcre_ord2utf8.c pcre_refcount.c pcre_study.c pcre_tables.c 
  pcre_try_flipped.c pcre_ucp_searchfuncs.c pcre_valid_utf8.c pcre_version.c pcre_xclass.c )

set ( PCREPOSIX_HEADERS pcreposix.h )

set ( PCREPOSIX_SOURCES pcreposix.c )

set ( PCRECPP_HEADERS pcrecpp.h pcre_scanner.h ${CMAKE_BINARY_DIR}/pcrecpparg.h ${CMAKE_BINARY_DIR}/pcre_stringpiece.h )

set ( PCRECPP_SOURCES pcrecpp.cc pcre_scanner.cc pcre_stringpiece.cc )

# Build setup

add_definitions ( -DHAVE_CONFIG_H )

if ( MSVC )
  add_definitions ( -D_CRT_SECURE_NO_DEPRECATE )
endif ( MSVC )

set ( CMAKE_INCLUDE_CURRENT_DIR 1 )
# needed to make sure to not link debug libs
# against release libs and vice versa

#LuaDist: Create SHARED libraries
# pcre
add_library ( pcre ${PCRE_HEADERS} ${PCRE_SOURCES} ${CMAKE_BINARY_DIR}/config.h ${DEF_FILE_PCRE} )

#LuaDist: Rename pcre library
set_target_properties ( pcre PROPERTIES OUTPUT_NAME pcre CLEAN_DIRECT_OUTPUT 1 )

#LuaDist: Create libraries

add_library ( pcreposix ${PCREPOSIX_HEADERS} ${PCREPOSIX_SOURCES} ${DEF_FILE_PCRPOSIX} )
target_link_libraries ( pcreposix pcre )

#LuaDist: Rename pcreposix library
set_target_properties ( pcreposix PROPERTIES OUTPUT_NAME pcreposix CLEAN_DIRECT_OUTPUT 
  1 )

if ( MINGW AND NOT PCRE_STATIC )
  if ( NON_STANDARD_LIB_PREFIX )
    set_target_properties ( pcre pcreposix PROPERTIES PREFIX "" )
  endif ( NON_STANDARD_LIB_PREFIX )

  if ( NON_STANDARD_LIB_SUFFIX )
    set_target_properties ( pcre pcreposix PROPERTIES SUFFIX "-0.dll" )
  endif ( NON_STANDARD_LIB_SUFFIX )
endif ( MINGW AND NOT PCRE_STATIC )

# pcrecpp
if ( PCRE_BUILD_PCRECPP )
  #LuaDist: Create libraries
  add_library ( pcrecpp ${PCRECPP_HEADERS} ${PCRECPP_SOURCES} ${DEF_FILE_PCRECPP} )
  target_link_libraries ( pcrecpp pcre )

  #LuaDist: Rename pcreposix library
  set_target_properties ( pcrecpp PROPERTIES OUTPUT_NAME pcrecpp CLEAN_DIRECT_OUTPUT 
    1 )

  if ( MINGW AND NOT PCRE_STATIC )
    if ( NON_STANDARD_LIB_PREFIX )
      set_target_properties ( pcrecpp PROPERTIES PREFIX "" )
    endif ( NON_STANDARD_LIB_PREFIX )

    if ( NON_STANDARD_LIB_SUFFIX )
      set_target_properties ( pcrecpp PROPERTIES SUFFIX "-0.dll" )
    endif ( NON_STANDARD_LIB_SUFFIX )
  endif ( MINGW AND NOT PCRE_STATIC )
endif ( PCRE_BUILD_PCRECPP )

# Executables

add_executable ( pcretest pcretest.c )
target_link_libraries ( pcretest pcreposix ${PCRETEST_LIBS} )

add_executable ( pcregrep pcregrep.c )
target_link_libraries ( pcregrep pcreposix ${PCREGREP_LIBS} )

# Removed by PH (2008-01-23) because pcredemo shouldn't really be built
# automatically, and it gave trouble in some environments anyway.
# ADD_EXECUTABLE(pcredemo pcredemo.c)
# TARGET_LINK_LIBRARIES(pcredemo pcreposix)
# IF(NOT BUILD_SHARED_LIBS)
#     # make sure to not use declspec(dllimport) in static mode on windows
#         SET_TARGET_PROPERTIES(pcredemo PROPERTIES COMPILE_FLAGS "-DPCRE_STATIC")
# ENDIF(NOT BUILD_SHARED_LIBS)

if ( PCRE_BUILD_PCRECPP )
  add_executable ( pcrecpp_unittest pcrecpp_unittest.cc )
  target_link_libraries ( pcrecpp_unittest pcrecpp )
  if ( MINGW AND NON_STANDARD_LIB_NAMES AND NOT PCRE_STATIC )
    set_target_properties ( pcrecpp PROPERTIES PREFIX "" )
  endif ( MINGW AND NON_STANDARD_LIB_NAMES AND NOT PCRE_STATIC )

  add_executable ( pcre_scanner_unittest pcre_scanner_unittest.cc )
  target_link_libraries ( pcre_scanner_unittest pcrecpp )

  add_executable ( pcre_stringpiece_unittest pcre_stringpiece_unittest.cc )
  target_link_libraries ( pcre_stringpiece_unittest pcrecpp )
endif ( PCRE_BUILD_PCRECPP )

# Testing

enable_testing ( )

get_target_property ( PCREGREP_EXE pcregrep DEBUG_LOCATION )
get_target_property ( PCRETEST_EXE pcretest DEBUG_LOCATION )

# Write out a CTest configuration file that sets some needed environment
# variables for the test scripts.
#
file ( WRITE ${CMAKE_BINARY_DIR}/CTestCustom.ctest "# This is a generated file. SET(ENV{srcdir} ${CMAKE_SOURCE_DIR}) SET(ENV{pcregrep} ${PCREGREP_EXE}) SET(ENV{pcretest} ${PCRETEST_EXE}) " )

if ( UNIX )
  add_test ( pcre_test ${CMAKE_SOURCE_DIR}/RunTest )
  add_test ( pcre_grep_test ${CMAKE_SOURCE_DIR}/RunGrepTest )
endif ( UNIX )
if ( WIN32 )
  add_test ( pcre_test cmd /C ${CMAKE_SOURCE_DIR}/RunTest.bat )
endif ( WIN32 )

get_target_property ( PCRECPP_UNITTEST_EXE pcrecpp_unittest DEBUG_LOCATION )

get_target_property ( PCRE_SCANNER_UNITTEST_EXE pcre_scanner_unittest ADD_CUSTOM_COMMAND_LOCATION )

get_target_property ( PCRE_STRINGPIECE_UNITTEST_EXE pcre_stringpiece_unittest DEBUG_LOCATION )

add_test ( pcrecpp_test ${PCRECPP_UNITTEST_EXE} )
add_test ( pcre_scanner_test ${PCRE_SCANNER_UNITTEST_EXE} )
add_test ( pcre_stringpiece_test ${PCRE_STRINGPIECE_UNITTEST_EXE} )

# Installation

#LuaDist: All install paths are modified for LuaDist

set ( CMAKE_INSTALL_ALWAYS 1 )

#INSTALL(TARGETS pcre pcreposix pcregrep pcretest
#        DESTINATION bin
#        DESTINATION lib
#        ARCHIVE DESTINATION lib)

install_library ( pcre pcreposix )
install_executable ( pcregrep pcretest )
install_header ( ${PCRE_HEADERS} ${PCREPOSIX_HEADERS} )

file ( GLOB html ${CMAKE_SOURCE_DIR}/doc/html/*.html )
file ( GLOB man1 ${CMAKE_SOURCE_DIR}/doc/*.1 )
file ( GLOB man3 ${CMAKE_SOURCE_DIR}/doc/*.3 )

if ( PCRE_BUILD_PCRECPP )
  #INSTALL(TARGETS pcrecpp
  #        DESTINATION bin
  #        DESTINATION lib
  #        ARCHIVE DESTINATION lib)

  install_library ( pcrecpp )

  #INSTALL(FILES ${PCRECPP_HEADERS} DESTINATION include)
  install_header ( ${PCRECPP_HEADERS} )

else ( PCRE_BUILD_PCRECPP )
  # Remove pcrecpp.3
  foreach ( man ${man3} )
  get_filename_component ( man_tmp ${man} NAME )
  if ( NOT man_tmp STREQUAL "pcrecpp.3" )
    set ( man3_new ${man3} ${man} )
  endif ( NOT man_tmp STREQUAL "pcrecpp.3" )
  endforeach ( man ${man3} )
  set ( man3 ${man3_new} )
endif ( PCRE_BUILD_PCRECPP )

#INSTALL(FILES ${man1} DESTINATION man/man1)
#INSTALL(FILES ${man3} DESTINATION man/man3)
#INSTALL(FILES ${html} DESTINATION share/doc/pcre/html)
install_doc ( ${html} )

# help, only for nice output
if ( BUILD_SHARED_LIBS )
  set ( BUILD_STATIC_LIBS OFF )
else ( BUILD_SHARED_LIBS )
  set ( BUILD_STATIC_LIBS ON )
endif ( BUILD_SHARED_LIBS )

message ( STATUS "" )
message ( STATUS "" )
message ( STATUS "PCRE configuration summary:" )
message ( STATUS "" )
message ( STATUS " Install prefix .................. : " ${CMAKE_INSTALL_PREFIX} )
message ( STATUS " C compiler ...................... : " ${CMAKE_C_COMPILER} )
message ( STATUS " C++ compiler .................... : " ${CMAKE_CXX_COMPILER} )
message ( STATUS " C compiler flags ................ : " ${CMAKE_C_FLAGS} )
#FIXME
message ( STATUS " C++ compiler flags .............. : " ${CMAKE_CXX_FLAGS} )
#FIXME
message ( STATUS "" )
message ( STATUS " Build C++ library ............... : " ${PCRE_BUILD_PCRECPP} )
message ( STATUS " Enable UTF-8 support ............ : " ${PCRE_SUPPORT_UNICODE_PROPERTIES} )
message ( STATUS " Unicode properties .............. : " ${PCRE_SUPPORT_UNICODE_PROPERTIES} )
message ( STATUS " Newline char/sequence ........... : " ${PCRE_NEWLINE} )
message ( STATUS " \\R matches only ANYCRLF ......... : " ${PCRE_SUPPORT_BSR_ANYCRLF} )
message ( STATUS " EBCDIC coding ................... : " ${PCRE_EBCDIC} )
message ( STATUS " Rebuild char tables ............. : " ${PCRE_REBUILD_CHARTABLES} )
message ( STATUS " No stack recursion .............. : " ${PCRE_NO_RECURSE} )
message ( STATUS " POSIX mem threshold ............. : " ${PCRE_POSIX_MALLOC_THRESHOLD} )
message ( STATUS " Internal link size .............. : " ${PCRE_LINK_SIZE} )
message ( STATUS " Match limit ..................... : " ${PCRE_MATCH_LIMIT} )
message ( STATUS " Match limit recursion ........... : " ${PCRE_MATCH_LIMIT_RECURSION} )
message ( STATUS " Build shared libs ............... : " ${BUILD_SHARED_LIBS} )
message ( STATUS " Build static libs ............... : " ${BUILD_STATIC_LIBS} )
if ( ZLIB_FOUND )
  message ( STATUS " Link pcregrep with libz ......... : " ${PCRE_SUPPORT_LIBZ} )
else ( ZLIB_FOUND )
  message ( STATUS " Link pcregrep with libz ......... : None" )
endif ( ZLIB_FOUND )
if ( BZIP2_FOUND )
  message ( STATUS " Link pcregrep with libbz2 ....... : " ${PCRE_SUPPORT_LIBBZ2} )
else ( BZIP2_FOUND )
  message ( STATUS " Link pcregrep with libbz2 ....... : None" )
endif ( BZIP2_FOUND )
if ( NOT PCRE_SUPPORT_LIBREADLINE )
  message ( STATUS " Link pcretest with libreadline .. : None" )
else ( NOT PCRE_SUPPORT_LIBREADLINE )
  message ( STATUS " Link pcretest with libreadline .. : " ${PCRE_SUPPORT_LIBREADLINE} )
endif ( NOT PCRE_SUPPORT_LIBREADLINE )
if ( MINGW AND NOT PCRE_STATIC )
  message ( STATUS " Non-standard dll names (prefix) . : " ${NON_STANDARD_LIB_PREFIX} )
  message ( STATUS " Non-standard dll names (suffix) . : " ${NON_STANDARD_LIB_SUFFIX} )
endif ( MINGW AND NOT PCRE_STATIC )
message ( STATUS "" )

# end CMakeLists.txt
###########################################################################################
