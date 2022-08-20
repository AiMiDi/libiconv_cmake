if(POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW)
endif()
CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

project ("libiconv_cmake" VERSION 1.17.0)

option(USE_SHARED_LIB "use shared lib" OFF)

INCLUDE (CheckIncludeFiles)
INCLUDE (CheckFunctionExists)
INCLUDE (CheckLibraryExists) 
INCLUDE (CheckTypeSize) 
INCLUDE (CheckSymbolExists)

# Define to the number of bits in type 'ptrdiff_t'.
CHECK_TYPE_SIZE("ptrdiff_t" SIZEOF_PTRDIFF_T)
IF (SIZEOF_PTRDIFF_T)
SET(BITSIZEOF_PTRDIFF_T "(${SIZEOF_PTRDIFF_T}*8)")
ENDIF (SIZEOF_PTRDIFF_T)

# Define to the number of bits in type 'sig_atomic_t'. 
CHECK_TYPE_SIZE("sig_atomic_t" SIZEOF_SIG_ATOMIC_T)
IF (SIZEOF_SIG_ATOMIC_T)
SET(BITSIZEOF_SIG_ATOMIC_T "(${SIZEOF_SIG_ATOMIC_T}*8)")
ENDIF (SIZEOF_SIG_ATOMIC_T)

# Define to the number of bits in type 'size_t'.
CHECK_TYPE_SIZE("size_t" SIZEOF_SIZE_T)
IF (SIZEOF_SIZE_T)
SET(BITSIZEOF_SIZE_T "(${SIZEOF_SIZE_T}*8)")
ENDIF (SIZEOF_SIZE_T)

# Define to the number of bits in type 'wchar_t'.
CHECK_TYPE_SIZE("wchar_t" SIZEOF_WCHAR_T)
IF (SIZEOF_WCHAR_T)
SET(BITSIZEOF_WCHAR_T "(${SIZEOF_WCHAR_T}*8)")
ENDIF (SIZEOF_WCHAR_T)

# Define to the number of bits in type 'wint_t'.
CHECK_TYPE_SIZE("wint_t" SIZEOF_WINT_T)
IF (SIZEOF_WINT_T)
SET(BITSIZEOF_WINT_T "(${SIZEOF_WINT_T}*8)")
ENDIF (SIZEOF_WINT_T)

# Define to one of `_getb67', `GETB67', `getb67' for Cray-2 and Cray-YMP
# systems. This function is required for `alloca.c' support on those systems.
SET(CRAY_STACKSEG_END OFF)

# Define to 1 if using `alloca.c'.
UNSET(C_ALLOCA)

# Define as good substitute value for EILSEQ. 
SET(EILSEQ 84)

# Define to 1 to enable a few rarely used encodings.
UNSET(ENABLE_EXTRA)

# Define to 1 if translation of program messages to the user's native
# language is requested.
UNSET(ENABLE_NLS)

# Define to 1 if the package shall run at any location in the filesystem. 
SET(ENABLE_RELOCATABLE ON)
#UNSET(ENABLE_RELOCATABLE)

# Define to 1 when using the gnulib module canonicalize-lgpl.
UNSET(GNULIB_CANONICALIZE_LGPL) 

# Define to 1 when using the gnulib module sigpipe.
UNSET(GNULIB_SIGPIPE) 

# Define to 1 if you have `alloca', as a function or macro.
CHECK_FUNCTION_EXISTS(alloca HAVE_ALLOCA)

# Define to 1 if you have <alloca.h> and it should be used (not on Ultrix).
CHECK_INCLUDE_FILES(alloca.h HAVE_ALLOCA_H)

# Define to 1 if you have the `canonicalize_file_name' function.
CHECK_FUNCTION_EXISTS(canonicalize_file_name HAVE_CANONICALIZE_FILE_NAME)

# Define to 1 if you have the MacOS X function CFLocaleCopyCurrent in the
# CoreFoundation framework. 
CHECK_FUNCTION_EXISTS(CFLocaleCopyCurrent HAVE_CFLOCALECOPYCURRENT)

# Define to 1 if you have the MacOS X function CFPreferencesCopyAppValue in
# the CoreFoundation framework.
CHECK_FUNCTION_EXISTS(CoreFoundation HAVE_CFPREFERENCESCOPYAPPVALUE)

# Define if the GNU dcgettext() function is already present or preinstalled.
CHECK_FUNCTION_EXISTS(dcgettext HAVE_DCGETTEXT)

# Define to 1 if you have the declaration of `canonicalize_file_name', and to
# 0 if you don't. 
CHECK_FUNCTION_EXISTS(canonicalize_file_name HAVE_DECL_CANONICALIZE_FILE_NAME)

# Define to 1 if you have the declaration of `clearerr_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(clearerr_unlocked HAVE_DECL_CLEARERR_UNLOCKED)

# Define to 1 if you have the declaration of `feof_unlocked', and to 0 if you
# don't.
CHECK_FUNCTION_EXISTS(feof_unlocked HAVE_DECL_FEOF_UNLOCKED)

# Define to 1 if you have the declaration of `ferror_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(ferror_unlocked HAVE_DECL_FERROR_UNLOCKED)

# Define to 1 if you have the declaration of `fflush_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fflush_unlocked HAVE_DECL_FFLUSH_UNLOCKED)

# Define to 1 if you have the declaration of `fgets_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fgets_unlocked HAVE_DECL_FGETS_UNLOCKED)

# Define to 1 if you have the declaration of `fputc_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fputc_unlocked HAVE_DECL_FPUTC_UNLOCKED)

# Define to 1 if you have the declaration of `fputs_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fputs_unlocked HAVE_DECL_FPUTS_UNLOCKED)

# Define to 1 if you have the declaration of `fread_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fread_unlocked HAVE_DECL_FREAD_UNLOCKED)

# Define to 1 if you have the declaration of `fwrite_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(fwrite_unlocked HAVE_DECL_FWRITE_UNLOCKED)

# Define to 1 if you have the declaration of `getchar_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(getchar_unlocked HAVE_DECL_GETCHAR_UNLOCKED)

# Define to 1 if you have the declaration of `getc_unlocked', and to 0 if you
# don't. 
CHECK_FUNCTION_EXISTS(getc_unlocked HAVE_DECL_GETC_UNLOCKED)

# Define to 1 if you have the declaration of `putchar_unlocked', and to 0 if
# you don't. 
CHECK_FUNCTION_EXISTS(putchar_unlocked HAVE_DECL_PUTCHAR_UNLOCKED)

# Define to 1 if you have the declaration of `putc_unlocked', and to 0 if you
# don't. 
CHECK_FUNCTION_EXISTS(putc_unlocked HAVE_DECL_PUTC_UNLOCKED)

# Define to 1 if you have the declaration of `strerror', and to 0 if you
# don't. 
CHECK_FUNCTION_EXISTS(strerror HAVE_DECL_STRERROR)

# Define to 1 if you have the declaration of `strerror_r', and to 0 if you
# don't. 
CHECK_FUNCTION_EXISTS(strerror_r HAVE_DECL_STRERROR_R)

# Define to 1 if you have the <dlfcn.h> header file. 
CHECK_INCLUDE_FILES(dlfcn.h HAVE_DLFCN_H)

# Define if you have the declaration of environ. 
CHECK_FUNCTION_EXISTS(environ HAVE_ENVIRON_DECL)

# Define to 1 if you have the <errno.h> header file. 
CHECK_INCLUDE_FILES(errno.h HAVE_ERRNO_H)

# Define to 1 if you have the `getcwd' function. 
CHECK_FUNCTION_EXISTS(getcwd HAVE_GETCWD)

# Define to 1 if you have the `getc_unlocked' function. 
CHECK_FUNCTION_EXISTS(getc_unlocked HAVE_GETC_UNLOCKED)

# Define if the GNU gettext() function is already present or preinstalled. 
CHECK_FUNCTION_EXISTS(gettext HAVE_GETTEXT)

# Define if you have the iconv() function and it works. 
SET(HAVE_ICONV ON)

# Define to 1 if you have the <inttypes.h> header file. 
CHECK_INCLUDE_FILES(inttypes.h HAVE_INTTYPES_H)

# Define if you have <langinfo.h> and nl_langinfo(CODESET). 
CHECK_INCLUDE_FILES(langinfo.h HAVE_LANGINFO_CODESET)

# Define to 1 if the system has the type `long long int'. 
CHECK_TYPE_SIZE("long long int" HAVE_LONG_LONG_INT)

# Define to 1 if you have the <mach-o/dyld.h> header file. 
CHECK_INCLUDE_FILES(mach-o/dyld.h HAVE_MACH_O_DYLD_H)

# Define if the 'malloc' function is POSIX compliant. 
IF (WIN32)
UNSET(HAVE_MALLOC_POSIX)
ELSE (WIN32)
SET(HAVE_MALLOC_POSIX ON)
ENDIF (WIN32)

# Define to 1 if you have the `mbrtowc' function. 
CHECK_FUNCTION_EXISTS(mbrtowc HAVE_MBRTOWC)

# Define to 1 if you have the `mbsinit' function. 
CHECK_FUNCTION_EXISTS(mbsinit HAVE_MBSINIT)

# Define to 1 if <wchar.h> declares mbstate_t. 
CHECK_SYMBOL_EXISTS(mbstate_t wchar.h HAVE_MBSTATE_T)

# Define to 1 if you have the `memmove' function. 
CHECK_FUNCTION_EXISTS(memmove HAVE_MEMMOVE)

# Define to 1 if you have the <memory.h> header file. 
CHECK_INCLUDE_FILES(memory.h HAVE_MEMORY_H)

# Define to 1 if you have the `pathconf' function. 
CHECK_FUNCTION_EXISTS(pathconf HAVE_PATHCONF)

# Define to 1 if you have the <random.h> header file. 
CHECK_INCLUDE_FILES(random.h HAVE_RANDOM_H)

# Define to 1 if you have the `readlink' function. 
CHECK_FUNCTION_EXISTS(readlink HAVE_READLINK)

# Define to 1 if you have the <search.h> header file. 
CHECK_INCLUDE_FILES(search.h HAVE_SEARCH_H)

# Define to 1 if you have the `setenv' function. 
CHECK_FUNCTION_EXISTS(setenv HAVE_SETENV)

# Define to 1 if you have the `setlocale' function. 
CHECK_FUNCTION_EXISTS(setlocale HAVE_SETLOCALE)

# Define to 1 if you have the <signal.h> header file. 
CHECK_INCLUDE_FILES(signal.h HAVE_SIGNAL_H)

# Define to 1 if 'sig_atomic_t' is a signed integer type. 
SET(HAVE_SIGNED_SIG_ATOMIC_T 0)

# Define to 1 if 'wchar_t' is a signed integer type. 
SET(HAVE_SIGNED_WCHAR_T 0)

# Define to 1 if 'wint_t' is a signed integer type. 
SET(HAVE_SIGNED_WINT_T 0)

# Define to 1 if the system has the type `sigset_t'. 
CHECK_SYMBOL_EXISTS(sigset_t signal.h HAVE_SIGSET_T)

# Define to 1 if stdbool.h conforms to C99. 
CHECK_INCLUDE_FILES(stdbool.h HAVE_STDBOOL_H)

# Define to 1 if you have the <stdint.h> header file. 
CHECK_INCLUDE_FILES(stdint.h HAVE_STDINT_H)

# Define to 1 if you have the <stdio.h> header file. 
CHECK_INCLUDE_FILES(stdio.h HAVE_STDIO_H)

# Define to 1 if you have the <stdlib.h> header file. 
CHECK_INCLUDE_FILES(stdlib.h HAVE_STDLIB_H)

# Define to 1 if you have the `strerror_r' function. 
CHECK_INCLUDE_FILES(strerror_r HAVE_STRERROR_R)

# Define to 1 if you have the <strings.h> header file. 
CHECK_INCLUDE_FILES(strings.h HAVE_STRINGS_H)

# Define to 1 if you have the <string.h> header file. 
CHECK_INCLUDE_FILES(string.h HAVE_STRING_H)

# Define to 1 if the system has the type `struct random_data'. 
UNSET(HAVE_STRUCT_RANDOM_DATA)

# Define to 1 if you have the <sys/bitypes.h> header file. 
CHECK_INCLUDE_FILES(sys/bitypes.h HAVE_SYS_BITYPES_H)

# Define to 1 if you have the <sys/inttypes.h> header file. 
CHECK_INCLUDE_FILES(sys/inttypes.h HAVE_SYS_INTTYPES_H)

# Define to 1 if you have the <sys/param.h> header file. 
CHECK_INCLUDE_FILES(sys/param.h HAVE_SYS_PARAM_H)

# Define to 1 if you have the <sys/socket.h> header file. 
CHECK_INCLUDE_FILES(sys/socket.h HAVE_SYS_SOCKET_H)

# Define to 1 if you have the <sys/stat.h> header file. 
CHECK_INCLUDE_FILES(sys/stat.h HAVE_SYS_STAT_H)

# Define to 1 if you have the <sys/types.h> header file. 
CHECK_INCLUDE_FILES(sys/types.h HAVE_SYS_TYPES_H)

# Define to 1 if you have the `tsearch' function. 
CHECK_FUNCTION_EXISTS(tsearch HAVE_TSEARCH)

# Define to 1 if you have the <unistd.h> header file. 
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)

# Define to 1 if the system has the type `unsigned long long int'. 
CHECK_TYPE_SIZE("unsigned long long int" HAVE_UNSIGNED_LONG_LONG_INT)

# Define to 1 or 0, depending whether the compiler supports simple visibility
# declarations. 
SET(HAVE_VISIBILITY 0)

# Define to 1 if you have the <wchar.h> header file. 
CHECK_INCLUDE_FILES(wchar.h HAVE_WCHAR_H)

# Define if you have the 'wchar_t' type. 
CHECK_SYMBOL_EXISTS(wchar_t wchar.h HAVE_WCHAR_T)

# Define to 1 if you have the `wcrtomb' function. 
CHECK_FUNCTION_EXISTS(wcrtomb HAVE_WCRTOMB)

# Define to 1 if you have the <winsock2.h> header file. 
IF (WIN32)
SET(HAVE_WINSOCK2_H ON)
ELSE (WIN32)
UNSET(HAVE_WINSOCK2_H)
ENDIF (WIN32)

# Define if you have the 'wint_t' type. 
CHECK_TYPE_SIZE("wint_t" HAVE_WINT_T)

# Define to 1 if the system has the type `_Bool'. 
CHECK_TYPE_SIZE(_Bool HAVE__BOOL)

# Define to 1 if you have the `_NSGetExecutablePath' function. 
CHECK_FUNCTION_EXISTS(_NSGetExecutablePath HAVE__NSGETEXECUTABLEPATH)

# Define as const if the declaration of iconv() needs const. 
SET(ICONV_CONST "const")

# iconv.h
SET(DLLVARIABLE "")
SET(EILSEQ "84")
SET(USE_MBSTATE_T "0")
SET(BROKEN_WCHAR_H "0")
SET(HAVE_WCHAR_T "1")

CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/config-cmake.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)
CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/include/iconv.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h @ONLY)
CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/libcharset/include/libcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/libcharset.h)
CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/libcharset/include/localcharset.h.in ${CMAKE_CURRENT_BINARY_DIR}/libcharset/include/localcharset.h)

# Definitions
if (USE_SHARED_LIB)
    add_definitions(-DLIBICONV_SHARED_BUILD)
endif()

IF (WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    add_definitions(-D_WIN32_NATIVE)
ENDIF (WIN32)

IF (MSVC)
    add_definitions(/wd4018)
    add_definitions(/wd4273)
    add_definitions(/wd4090)
    add_definitions(/wd4013)
    add_definitions(/wd4244)
    add_definitions(/wd4311)
    add_definitions(/wd4267)
ENDIF (MSVC)

add_definitions(-DICONV_NO_LIBDIR)
add_definitions(-DLIBDIR)
 
# Includes
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/include ${CMAKE_CURRENT_SOURCE_DIR}/srclib ${CMAKE_CURRENT_SOURCE_DIR}/lib)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/libcharset/include ${CMAKE_CURRENT_SOURCE_DIR}/libcharset/include ${CMAKE_CURRENT_SOURCE_DIR}/libcharset/lib)  

IF (WIN32)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/msvc)
ENDIF (WIN32)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

# Build

if (NOT DEFINED CMAKE_DEBUG_POSTFIX)
  set(CMAKE_DEBUG_POSTFIX "_Debug")
endif()

if (USE_SHARED_LIB)
   add_library(iconv lib/iconv.c libcharset/lib/localcharset.c)
   set_target_properties(iconv PROPERTIES VERSION ${LIBICONV_VERSION} SOVERSION ${LIBICONV_SOVERSION} DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}")
else()
   add_library(iconv STATIC lib/iconv.c libcharset/lib/localcharset.c)
   set_target_properties(iconv PROPERTIES INTERFACE_COMPILE_DEFINITIONS "LIBICONV_STATIC" DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}")
endif()

target_include_directories(iconv
    PUBLIC
        $<INSTALL_INTERFACE:include>
    )
    
# Install
install(FILES 
	${CMAKE_CURRENT_BINARY_DIR}/include/iconv.h
	DESTINATION include
	)

install(TARGETS iconv
    EXPORT Targets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
    )


