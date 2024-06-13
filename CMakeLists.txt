#  Copyright (C) 1999-2023 Markus Rickert, Aimidi.
#  This file is part of the GNU LIBICONV Library.
#
#  The GNU LIBICONV Library is free software; you can redistribute it
#  and/or modify it under the terms of the GNU Lesser General Public
#  License as published by the Free Software Foundation; either version 2.1
#  of the License, or (at your option) any later version.
#
#  The GNU LIBICONV Library is distributed in the hope that it will be
#  useful, but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
#  Lesser General Public License for more details.
#
#  You should have received a copy of the GNU Lesser General Public
#  License along with the GNU LIBICONV Library; see the file COPYING.LIB.
#  If not, see <https://www.gnu.org/licenses/>.  */


cmake_minimum_required(VERSION 3.5)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(CheckCCompilerFlag)
include(CheckCSourceCompiles)
include(CheckCSourceRuns)
include(CheckFunctionExists)
include(CheckIncludeFiles)
include(CheckSymbolExists)
include(CheckTypeSize)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(TestBigEndian)

set(LIBCHARSET_VERSION_MAJOR 1)
set(LIBCHARSET_VERSION_MINOR 5)
set(LIBCHARSET_VERSION ${LIBCHARSET_VERSION_MAJOR}.${LIBCHARSET_VERSION_MINOR})

set(LIBCHARSET_SOVERSION_MAJOR 1)
set(LIBCHARSET_SOVERSION_MINOR 0)
set(LIBCHARSET_SOVERSION_PATCH 0)
set(LIBCHARSET_SOVERSION ${LIBCHARSET_SOVERSION_MAJOR}.${LIBCHARSET_SOVERSION_MINOR}.${LIBCHARSET_SOVERSION_PATCH})

set(LIBICONV_VERSION_MAJOR 1)
set(LIBICONV_VERSION_MINOR 17)
set(LIBICONV_VERSION ${LIBICONV_VERSION_MAJOR}.${LIBICONV_VERSION_MINOR})

set(LIBICONV_SOVERSION_MAJOR 2)
set(LIBICONV_SOVERSION_MINOR 6)
set(LIBICONV_SOVERSION_PATCH 1)
set(LIBICONV_SOVERSION ${LIBICONV_SOVERSION_MAJOR}.${LIBICONV_SOVERSION_MINOR}.${LIBICONV_SOVERSION_PATCH})

project(libiconv VERSION ${LIBICONV_VERSION} LANGUAGES C)

if(NOT GPERF_PROJECT_PATH)
	set(GPERF_PROJECT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/gperf)
endif()

if(NOT ICONV_PROJECT_PATH)
	set(ICONV_PROJECT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/libiconv)
endif()

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(LIBICONV_BUILD_DOCUMENTATION "Build documentation" OFF)
option(LIBICONV_ENABLE_EXTRA "Enable a few rarely used encodings" OFF)
option(LIBICONV_ENABLE_NLS "Translate program messages to user's native language" OFF)
option(LIBICONV_ENABLE_RELOCATABLE "Install a package that can be moved in the file system" ON)

if(NOT TARGET gperf::gperf)
	find_package(Gperf CONFIG)
	if(NOT TARGET gperf::gperf)
		add_subdirectory(gperf)
	endif()
endif()

check_symbol_exists(EILSEQ "errno.h" HAVE_EILSEQ)
if(NOT HAVE_EILSEQ)
	set(EILSEQ "ENOENT")
endif()
set(ENABLE_EXTRA ${LIBICONV_ENABLE_EXTRA})
set(ENABLE_NLS ${LIBICONV_ENABLE_NLS})
set(ENABLE_RELOCATABLE ${LIBICONV_ENABLE_RELOCATABLE})
check_function_exists(iconv HAVE_ICONV)
check_c_source_compiles("
	#include <langinfo.h>
	int main() { char* cs = nl_langinfo(CODESET); return !cs; }
" HAVE_LANGINFO_CODESET)
check_function_exists(mbrtowc HAVE_MBRTOWC)
check_function_exists(mbsinit HAVE_MBSINIT)
check_c_compiler_flag("-fvisibility=hidden -Werror" HAVE_VISIBILITY)
check_type_size(wchar_t WCHAR_T)
check_function_exists(wcrtomb HAVE_WCRTOMB)
check_c_source_compiles("
	#include <stdlib.h>
	#include <iconv.h>
	extern
	#ifdef __cplusplus
	\"C\"
	#endif
	#if defined(__STDC__) || defined(__cplusplus)
	size_t iconv(iconv_t cd, char** inbuf, size_t* inbytesleft, char** outbuf, size_t* outbytesleft);
	#else
	size_t iconv();
	#endif
	int main() { return 0; }
" ICONV_CONST_TEST)
if(NOT ICONV_CONST_TEST)
	set(ICONV_CONST "const")
endif()
set(INSTALLPREFIX ${CMAKE_INSTALL_PREFIX})
test_big_endian(WORDS_BIGENDIAN)
if(WORDS_BIGENDIAN)
	set(WORDS_LITTLEENDIAN OFF)
else()
	set(WORDS_LITTLEENDIAN ON)
endif()
check_type_size("size_t" SIZE_T)
if(NOT HAVE_SIZE_T)
	set(size_t "unsigned int")
endif()

set(PACKAGE "libicharset")
set(PACKAGE_BUGREPORT "bug-gnu-libiconv@gnu.org")
set(PACKAGE_NAME "libicharset")
set(PACKAGE_STRING "libicharset ${LIBCHARSET_VERSION}")
set(PACKAGE_TARNAME "libicharset")
set(PACKAGE_URL "https://www.gnu.org/software/libiconv/")
set(PACKAGE_VERSION ${LIBCHARSET_VERSION})

configure_file(config.h.cmake.in libcharset/config.h @ONLY)

set(PACKAGE "libiconv")
set(PACKAGE_BUGREPORT "bug-gnu-libiconv@gnu.org")
set(PACKAGE_NAME "libiconv")
set(PACKAGE_STRING "libiconv ${LIBICONV_VERSION}")
set(PACKAGE_TARNAME "libiconv")
set(PACKAGE_URL "https://www.gnu.org/software/libiconv/")
set(PACKAGE_VERSION ${LIBICONV_VERSION})

configure_file(config.h.cmake.in config.h @ONLY)

if(HAVE_VISIBILTY)
	set(HAVE_VISIBILITY 1)
else()
	set(HAVE_VISIBILITY 0)
endif()

if(HAVE_WCHAR_T)
	set(BROKEN_WCHAR_H 0)
	set(HAVE_WCHAR_T 1)
else()
	set(BROKEN_WCHAR_H 1)
	set(HAVE_WCHAR_T 0)
endif()

if(HAVE_WCRTOMB OR HAVE_MBRTOWC)
	set(USE_MBSTATE_T 1)
else()
	set(USE_MBSTATE_T 0)
endif()

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

function(genaliases SUFFIX DEFINITIONS)
	add_executable(genaliases${SUFFIX} ${ICONV_PROJECT_PATH}/lib/genaliases.c ${ICONV_PROJECT_PATH}/lib/encodings.def ${ICONV_PROJECT_PATH}/lib/encodings_local.def)
	target_compile_definitions(genaliases${SUFFIX} PRIVATE ${DEFINITIONS})
	add_custom_command(
		OUTPUT lib/aliases${SUFFIX}.gperf lib/canonical${SUFFIX}.sh lib/canonical_local${SUFFIX}.sh
		COMMAND genaliases${SUFFIX} ${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.gperf ${CMAKE_CURRENT_BINARY_DIR}/lib/canonical${SUFFIX}.sh ${CMAKE_CURRENT_BINARY_DIR}/lib/canonical_local${SUFFIX}.sh
		DEPENDS genaliases${SUFFIX}
	)
	add_custom_command(
		OUTPUT lib/aliases${SUFFIX}.tmp
		COMMAND gperf::gperf -m 10 --output-file=${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.tmp ${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.gperf
		DEPENDS lib/aliases${SUFFIX}.gperf
	)
	add_custom_command(
		OUTPUT lib/aliases${SUFFIX}.h
		COMMAND ${CMAKE_COMMAND} -DINPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.tmp -DOUTPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.h -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/gperf-static.cmake
		DEPENDS cmake/gperf-static.cmake lib/aliases${SUFFIX}.tmp
	)
	add_custom_command(
		OUTPUT lib/canonical${SUFFIX}.h
		COMMAND ${CMAKE_COMMAND} -DPATTERNS_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/canonical${SUFFIX}.sh -DINPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.h -DOUTPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/canonical${SUFFIX}.h -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/genaliases.cmake
		DEPENDS cmake/genaliases.cmake lib/aliases${SUFFIX}.h lib/canonical${SUFFIX}.sh
	)
	add_custom_command(
		OUTPUT lib/canonical_local${SUFFIX}.h
		COMMAND ${CMAKE_COMMAND} -DPATTERNS_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/canonical_local${SUFFIX}.sh -DINPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/aliases${SUFFIX}.h -DOUTPUT_FILENAME=${CMAKE_CURRENT_BINARY_DIR}/lib/canonical_local${SUFFIX}.h -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/genaliases.cmake
		DEPENDS cmake/genaliases.cmake lib/aliases${SUFFIX}.h lib/canonical_local${SUFFIX}.sh
	)
endfunction()

genaliases("" "")
genaliases("_sysaix" "USE_AIX_ALIASES")
genaliases("_syshpux" "USE_HPUX_ALIASES")
genaliases("_sysosf1" "USE_OSF1_ALIASES")
genaliases("_syssolaris" "USE_SOLARIS_ALIASES")

function(genaliases2 SUFFIX TAG DEFINITIONS DEPENDENCIES)
	add_executable(genaliases${SUFFIX} ${ICONV_PROJECT_PATH}/lib/genaliases2.c ${DEPENDENCIES})
	target_compile_definitions(genaliases${SUFFIX} PRIVATE ${DEFINITIONS})
	add_custom_command(
		OUTPUT lib/aliases${SUFFIX}.h lib/canonical${SUFFIX}.h
		COMMAND genaliases${SUFFIX} ${TAG} ${ICONV_PROJECT_PATH}/lib/aliases${SUFFIX}.h ${ICONV_PROJECT_PATH}/lib/canonical${SUFFIX}.h
		DEPENDS genaliases${SUFFIX}
	)
endfunction()

genaliases2("_aix" "aix" "USE_AIX" ${ICONV_PROJECT_PATH}/lib/encodings_aix.def)
genaliases2("_aix_sysaix" "aix" "USE_AIX;USE_AIX_ALIASES" ${ICONV_PROJECT_PATH}/lib/encodings_aix.def)
genaliases2("_osf1" "osf1" "USE_OSF1" ${ICONV_PROJECT_PATH}/lib/encodings_osf1.def)
genaliases2("_osf1_sysosf1" "osf1" "USE_OSF1;USE_OSF1_ALIASES" ${ICONV_PROJECT_PATH}/lib/encodings_osf1.def)
genaliases2("_dos" "dos" "USE_DOS" ${ICONV_PROJECT_PATH}/lib/encodings_dos.def)
genaliases2("_zos" "zos" "USE_ZOS" ${ICONV_PROJECT_PATH}/lib/encodings_zos.def)
genaliases2("_extra" "extra" "USE_EXTRA" ${ICONV_PROJECT_PATH}/lib/encodings_extra.def)

add_executable(genflags ${ICONV_PROJECT_PATH}/lib/converters.h ${ICONV_PROJECT_PATH}/lib/encodings.def ${ICONV_PROJECT_PATH}/lib/encodings_aix.def ${ICONV_PROJECT_PATH}/lib/encodings_dos.def ${ICONV_PROJECT_PATH}/lib/encodings_zos.def ${ICONV_PROJECT_PATH}/lib/encodings_extra.def ${ICONV_PROJECT_PATH}/lib/encodings_osf1.def ${ICONV_PROJECT_PATH}/lib/genflags.c)
add_custom_command(
	OUTPUT lib/flags.h
	COMMAND genflags > lib/flags.h
	DEPENDS ${ICONV_PROJECT_PATH}/lib/converters.h ${ICONV_PROJECT_PATH}/lib/encodings.def ${ICONV_PROJECT_PATH}/lib/encodings_aix.def ${ICONV_PROJECT_PATH}/lib/encodings_dos.def ${ICONV_PROJECT_PATH}/lib/encodings_zos.def ${ICONV_PROJECT_PATH}/lib/encodings_extra.def ${ICONV_PROJECT_PATH}/lib/encodings_osf1.def ${ICONV_PROJECT_PATH}/lib/genflags.c
)

add_executable(gentranslit ${ICONV_PROJECT_PATH}/lib/gentranslit.c ${ICONV_PROJECT_PATH}/lib/translit.def)
add_custom_command(
	OUTPUT lib/translit.h
	COMMAND gentranslit < ${ICONV_PROJECT_PATH}/lib/translit.def > lib/translit.h
	DEPENDS ${ICONV_PROJECT_PATH}/lib/gentranslit.c ${ICONV_PROJECT_PATH}/lib/translit.def
)

file(READ ${ICONV_PROJECT_PATH}/libcharset/include/export.h LIBCHARSET_EXPORT_H)

file(READ ${ICONV_PROJECT_PATH}/libcharset/include/libcharset.h.in LIBCHARSET_H_IN)
string(REGEX REPLACE "extern ([^\"])" "extern LIBCHARSET_DLL_EXPORTED \\1" LIBCHARSET_H_IN "${LIBCHARSET_H_IN}")
string(REGEX REPLACE "(#define _LIBCHARSET_H[^\n]*\n)" "\\1${LIBCHARSET_EXPORT_H}" LIBCHARSET_H_IN "${LIBCHARSET_H_IN}")
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h.build.in "${LIBCHARSET_H_IN}")

file(READ ${ICONV_PROJECT_PATH}/libcharset/include/localcharset.h.in LOCALCHARSET_H_IN)
string(REGEX REPLACE "extern ([^\"])" "extern LIBCHARSET_DLL_EXPORTED \\1" LOCALCHARSET_H_IN "${LOCALCHARSET_H_IN}")
string(REGEX REPLACE "(#define _LOCALCHARSET_H[^\n]*\n)" "\\1${LIBCHARSET_EXPORT_H}" LOCALCHARSET_H_IN "${LOCALCHARSET_H_IN}")
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h.build.in "${LOCALCHARSET_H_IN}")

configure_file(${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h.build.in libcharset/include/libcharset.h @ONLY)
configure_file(${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h.build.in libcharset/include/localcharset.h @ONLY)

file(READ ${ICONV_PROJECT_PATH}/include/export.h EXPORT_H)

file(READ ${ICONV_PROJECT_PATH}/include/iconv.h.in ICONV_H_IN)
string(REGEX REPLACE "extern ([^\"])" "extern LIBICONV_DLL_EXPORTED \\1" ICONV_H_IN "${ICONV_H_IN}")
string(REGEX REPLACE "(_LIBICONV_VERSION[^\n]*\n)" "\\1${EXPORT_H}" ICONV_H_IN "${ICONV_H_IN}")
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h.build.in "${ICONV_H_IN}")

configure_file(${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h.build.in include/iconv.h @ONLY)

if(BUILD_SHARED_LIBS AND MSVC)
	set(DLL_VARIABLE "__declspec (dllimport)")
endif()

configure_file(${ICONV_PROJECT_PATH}/libcharset/include/libcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h.inst @ONLY)
configure_file(${ICONV_PROJECT_PATH}/libcharset/include/localcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h.inst @ONLY)
configure_file(${ICONV_PROJECT_PATH}/include/iconv.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h.inst @ONLY)

add_library(
	Charset
	${ICONV_PROJECT_PATH}/libcharset/lib/localcharset.c
	${ICONV_PROJECT_PATH}/libcharset/lib/relocatable-stub.c
)

target_compile_definitions(
	Charset
	PRIVATE
	BUILDING_LIBCHARSET
	HAVE_CONFIG_H
	IN_LIBRARY
	INSTALLDIR="${CMAKE_INSTALL_LIBDIR}"
	LIBDIR="${CMAKE_INSTALL_LIBDIR}"
	NO_XMALLOC
	relocate=libcharset_relocate
	set_relocation_prefix=libcharset_set_relocation_prefix
)

if(BUILD_SHARED_LIBS)
	target_compile_definitions(Charset PRIVATE BUILDING_DLL)
endif()

target_include_directories(
	Charset
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/libcharset>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/libcharset/include>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

set_target_properties(
	Charset
	PROPERTIES
	C_VISIBILITY_PRESET hidden
	IMPORT_PREFIX lib
	OUTPUT_NAME charset
	POSITION_INDEPENDENT_CODE ON
	PREFIX lib
	SOVERSION ${LIBCHARSET_SOVERSION_MAJOR}
	VERSION ${LIBCHARSET_SOVERSION}
)

if(MSVC)
	if(BUILD_SHARED_LIBS)
		set_target_properties(
			Charset
			PROPERTIES
			DEBUG_POSTFIX _Debug
		)
	else()
		set_target_properties(
			Charset
			PROPERTIES
			DEBUG_POSTFIX _StaticDebug
			MINSIZEREL_POSTFIX _Static
			RELEASE_POSTFIX _Static
			RELWITHDEBINFO_POSTFIX _Static
		)
	endif()
endif()

add_library(
	Iconv
	lib/aliases.h
	lib/aliases_aix.h
	lib/aliases_aix_sysaix.h
	lib/aliases_dos.h
	lib/aliases_extra.h
	lib/aliases_osf1.h
	lib/aliases_osf1_sysosf1.h
	lib/aliases_sysaix.h
	lib/aliases_syshpux.h
	lib/aliases_sysosf1.h
	lib/aliases_syssolaris.h
	lib/aliases_zos.h
	lib/canonical.h
	lib/canonical_aix.h
	lib/canonical_aix_sysaix.h
	lib/canonical_dos.h
	lib/canonical_extra.h
	lib/canonical_local.h
	lib/canonical_local_sysaix.h
	lib/canonical_local_syshpux.h
	lib/canonical_local_sysosf1.h
	lib/canonical_local_syssolaris.h
	lib/canonical_osf1.h
	lib/canonical_osf1_sysosf1.h
	lib/canonical_sysaix.h
	lib/canonical_syshpux.h
	lib/canonical_sysosf1.h
	lib/canonical_syssolaris.h
	lib/canonical_zos.h
	lib/flags.h
	lib/translit.h
	${ICONV_PROJECT_PATH}/lib/converters.h
	${ICONV_PROJECT_PATH}/lib/iconv.c
	${ICONV_PROJECT_PATH}/lib/relocatable.c
	${ICONV_PROJECT_PATH}/libcharset/lib/localcharset.c
)

target_compile_definitions(
	Iconv
	PRIVATE
	BUILDING_LIBCHARSET
	BUILDING_LIBICONV
	HAVE_CONFIG_H
	IN_LIBRARY
	INSTALLDIR="${CMAKE_INSTALL_LIBDIR}"
	LIBDIR="${CMAKE_INSTALL_LIBDIR}"
	NO_XMALLOC
	relocate=libiconv_relocate
	relocate2=libiconv_relocate2
	set_relocation_prefix=libiconv_set_relocation_prefix
)

if(BUILD_SHARED_LIBS)
	target_compile_definitions(Iconv PRIVATE BUILDING_DLL)
endif()

target_include_directories(
	Iconv
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/lib>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(Iconv Charset)

set_target_properties(
	Iconv
	PROPERTIES
	C_VISIBILITY_PRESET hidden
	IMPORT_PREFIX lib
	OUTPUT_NAME iconv
	POSITION_INDEPENDENT_CODE ON
	PREFIX lib
	SOVERSION ${LIBICONV_SOVERSION_MAJOR}
	VERSION ${LIBICONV_SOVERSION}
)

if(MSVC)
	if(BUILD_SHARED_LIBS)
		set_target_properties(
			Iconv
			PROPERTIES
			DEBUG_POSTFIX _Debug
		)
	else()
		set_target_properties(
			Iconv
			PROPERTIES
			DEBUG_POSTFIX _StaticDebug
			MINSIZEREL_POSTFIX _Static
			RELEASE_POSTFIX _Static
			RELWITHDEBINFO_POSTFIX _Static
		)
	endif()
endif()

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h.inst DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${CMAKE_PROJECT_NAME} RENAME libcharset.h COMPONENT development)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h.inst DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${CMAKE_PROJECT_NAME} RENAME localcharset.h COMPONENT development)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h.inst DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${CMAKE_PROJECT_NAME} RENAME iconv.h COMPONENT development)

if(NOT CMAKE_VERSION VERSION_LESS 3.12)
	install(
		TARGETS Charset Iconv
		EXPORT Iconv
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_COMPONENT development
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
	)
else()
	install(
		TARGETS Charset Iconv
		EXPORT Iconv
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_SKIP
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
	)
	if(BUILD_SHARED_LIBS)
		install(
			TARGETS Charset Iconv
			EXPORT Iconv
			LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development NAMELINK_ONLY
		)
	endif()
endif()

if(MSVC AND BUILD_SHARED_LIBS)
	install(FILES $<TARGET_PDB_FILE:Charset> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
	install(FILES $<TARGET_PDB_FILE:Iconv> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
endif()

if(LIBICONV_BUILD_DOCUMENTATION)
	install(FILES man/iconv.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1 COMPONENT documentation)
	install(FILES man/iconv.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
	install(FILES man/iconv_close.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
	install(FILES man/iconv_open.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
	install(FILES man/iconv_open_into.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
	install(FILES man/iconvctl.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
	find_package(Man2html REQUIRED)
	file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/man)
	add_custom_command(OUTPUT man/iconv.1.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconv.1 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconv.1.html DEPENDS man/iconv.1)
	add_custom_command(OUTPUT man/iconv.3.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconv.3 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconv.3.html DEPENDS man/iconv.3)
	add_custom_command(OUTPUT man/iconv_close.3.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconv_close.3 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_close.3.html DEPENDS man/iconv_close.3)
	add_custom_command(OUTPUT man/iconv_open.3.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconv_open.3 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_open.3.html DEPENDS man/iconv_open.3)
	add_custom_command(OUTPUT man/iconv_open_into.3.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconv_open_into.3 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_open_into.3.html DEPENDS man/iconv_open_into.3)
	add_custom_command(OUTPUT man/iconvctl.3.html COMMAND man2html::man2html < ${CMAKE_CURRENT_SOURCE_DIR}/man/iconvctl.3 > ${CMAKE_CURRENT_BINARY_DIR}/man/iconvctl.3.html DEPENDS man/iconvctl.3)
	add_custom_target(html ALL DEPENDS man/iconv.1.html man/iconv.3.html man/iconv_close.3.html man/iconv_open.3.html man/iconv_open_into.3.html man/iconvctl.3.html)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconv.1.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconv.3.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_close.3.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_open.3.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconv_open_into.3.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/man/iconvctl.3.html DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libiconv COMPONENT documentation)
endif()

export(
	TARGETS Charset Iconv
	NAMESPACE Iconv::
	FILE ${CMAKE_CURRENT_BINARY_DIR}/iconv-export.cmake
)

configure_package_config_file(
	iconv-config.cmake.in iconv-config.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iconv-${LIBICONV_VERSION}
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/iconv-config.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iconv-${LIBICONV_VERSION}
	COMPONENT development
)

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/iconv-config-version.cmake
	VERSION ${LIBICONV_VERSION}
	COMPATIBILITY ExactVersion
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/iconv-config-version.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iconv-${LIBICONV_VERSION}
	COMPONENT development
)

install(
	EXPORT Iconv
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iconv-${LIBICONV_VERSION}
	NAMESPACE Iconv::
	FILE iconv-export.cmake
	COMPONENT development
)

configure_file(${ICONV_PROJECT_PATH}/COPYING.LIB ${CMAKE_CURRENT_BINARY_DIR}/COPYING.LIB.txt COPYONLY)

set(CPACK_COMPONENT_DEBUG_DISPLAY_NAME "Debug")
set(CPACK_COMPONENT_DEVELOPMENT_DEPENDS runtime)
set(CPACK_COMPONENT_DEVELOPMENT_DISPLAY_NAME "Development")
set(CPACK_COMPONENT_DOCUMENTATION_DISPLAY_NAME "Documentation")
set(CPACK_COMPONENT_RUNTIME_DISPLAY_NAME "Runtime")
set(CPACK_DEB_COMPONENT_INSTALL ON)
set(CPACK_DEBIAN_DEVELOPMENT_FILE_NAME "DEB-DEFAULT")
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_DEPENDS "${PACKAGE_TARNAME}")
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_NAME "${PACKAGE_TARNAME}-dev")
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_SECTION "libdevel")
set(CPACK_DEBIAN_DOCUMENTATION_FILE_NAME "DEB-DEFAULT")
set(CPACK_DEBIAN_DOCUMENTATION_PACKAGE_NAME "${PACKAGE_TARNAME}-doc")
set(CPACK_DEBIAN_DOCUMENTATION_PACKAGE_SECTION "doc")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE ${PACKAGE_URL})
set(CPACK_DEBIAN_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_DEBIAN_PACKAGE_SECTION "devel")
set(CPACK_DEBIAN_RUNTIME_FILE_NAME "DEB-DEFAULT")
set(CPACK_DEBIAN_RUNTIME_PACKAGE_NAME "${PACKAGE_TARNAME}")
set(CPACK_DEBIAN_RUNTIME_PACKAGE_SECTION "libs")
set(CPACK_NSIS_PACKAGE_NAME ${PACKAGE_STRING})
set(CPACK_NSIS_URL_INFO_ABOUT ${PACKAGE_URL})
set(CPACK_PACKAGE_CONTACT ${PACKAGE_BUGREPORT})
set(CPACK_PACKAGE_DISPLAY_NAME ${PACKAGE_STRING})
set(CPACK_PACKAGE_INSTALL_DIRECTORY "${PACKAGE_TARNAME}-${PACKAGE_VERSION}")
set(CPACK_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_PACKAGE_VENDOR "GNU Project")
set(CPACK_PACKAGE_VERSION ${PACKAGE_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${LIBICONV_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${LIBICONV_VERSION_MINOR})
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_BINARY_DIR}/COPYING.LIB.txt)
set(CPACK_RPM_COMPONENT_INSTALL ON)
set(CPACK_RPM_development_FILE_NAME "RPM-DEFAULT")
set(CPACK_RPM_development_PACKAGE_NAME "${PACKAGE_TARNAME}-devel")
set(CPACK_RPM_development_PACKAGE_REQUIRES "${PACKAGE_TARNAME}")
set(CPACK_RPM_documentation_FILE_NAME "RPM-DEFAULT")
set(CPACK_RPM_documentation_PACKAGE_GROUP "Documentation")
set(CPACK_RPM_documentation_PACKAGE_NAME "${PACKAGE_TARNAME}-doc")
set(CPACK_RPM_PACKAGE_GROUP "Development/Libraries")
set(CPACK_RPM_PACKAGE_LICENSE "LGPL")
set(CPACK_RPM_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_RPM_PACKAGE_URL ${PACKAGE_URL})
set(CPACK_RPM_runtime_FILE_NAME "RPM-DEFAULT")
set(CPACK_RPM_runtime_PACKAGE_NAME "${PACKAGE_TARNAME}")
set(CPACK_WIX_PROPERTY_ARPHELPLINK ${PACKAGE_BUGREPORT})
set(CPACK_WIX_PROPERTY_ARPURLINFOABOUT ${PACKAGE_URL})

include(CPack)
