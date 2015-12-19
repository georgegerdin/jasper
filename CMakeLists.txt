cmake_minimum_required(VERSION 2.8)

project(jasper C)

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include/jasper" CACHE PATH "Installation directory for headers")

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)
enable_testing()

check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(stdint.h    HAVE_STDINT_H)
check_include_file(stddef.h    HAVE_STDDEF_H)

#
# Check to see if we have large file support
#
set(CMAKE_REQUIRED_DEFINITIONS -D_LARGEFILE64_SOURCE=1)
# We add these other definitions here because CheckTypeSize.cmake
# in CMake 2.4.x does not automatically do so and we want
# compatibility with CMake 2.4.x.
if(HAVE_SYS_TYPES_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_SYS_TYPES_H)
endif()
if(HAVE_STDINT_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDINT_H)
endif()
if(HAVE_STDDEF_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDDEF_H)
endif()
check_type_size(off64_t OFF64_T)
if(HAVE_OFF64_T)
   add_definitions(-D_LARGEFILE64_SOURCE=1)
endif()
set(CMAKE_REQUIRED_DEFINITIONS) # clear variable

#
# Check for fseeko
#
check_function_exists(fseeko HAVE_FSEEKO)
if(NOT HAVE_FSEEKO)
    add_definitions(-DNO_FSEEKO)
endif()

#
# Check for unistd.h
#
check_include_file(unistd.h Z_HAVE_UNISTD_H)


if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
	add_definitions(-DJAS_WIN_MSVC_BUILD)
endif()

configure_file(	${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include/jasper/jas_config.h.in
		${CMAKE_CURRENT_BINARY_DIR}/jasper/jas_config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include)

#SOURCE FLES
file(GLOB LIBJASPER_BASE_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/base/*.c)
file(GLOB LIBJASPER_BMP_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/bmp/*.c)
file(GLOB LIBJASPER_JP2_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jp2/*.c)
file(GLOB LIBJASPER_JPC_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jpc/*.c)
set(      LIBJASPER_JPG_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jpg/jpg_dummy.c ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jpg/jpg_val.c)
file(GLOB LIBJASPER_MIF_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/mif/*.c)
file(GLOB LIBJASPER_PGX_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/pgx/*.c)
file(GLOB LIBJASPER_PNM_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/pnm/*.c)
file(GLOB LIBJASPER_RAS_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/ras/*.c)

#HEADER FILES
set(      LIBJASPER_BMP_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/bmp/bmp_cod.h)
set(	  LIBJASPER_JAS_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include/jasper/jas_cm.h
                             ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include/jasper/jas_icc.h
                             ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include/jasper/jas_tmr.h)
file(GLOB LIBJASPER_JP2_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jp2/*.h)
file(GLOB LIBJASPER_JPC_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jpc/*.h)
set(      LIBJASPER_JPG_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/jpg/jpg_cod.h)
set(      LIBJASPER_MIF_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/mif/mif_cod.h)
set(      LIBJASPER_PGX_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/pgx/pgx_cod.h)
set(      LIBJASPER_PNM_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/pnm/pnm_cod.h)
set(      LIBJASPER_RAS_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/ras/ras_cod.h)                             

file(GLOB LIBJASPER_PUBLIC_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/src/libjasper/include/*.h)

set(LIBJASPER_SRCS			${LIBJASPER_BASE_SRCS}
							${LIBJASPER_BMP_SRCS}
							${LIBJASPER_JP2_SRCS}
							${LIBJASPER_JPC_SRCS}
							${LIBJASPER_JPG_SRCS}
							${LIBJASPER_MIF_SRCS}
							${LIBJASPER_PGX_SRCS}
							${LIBJASPER_PNM_SRCS}
							${LIBJASPER_RAS_SRCS})
							
set(LIBJASPER_HDRS			${LIBJASPER_BMP_HDRS}
							${LIBJASPER_JAS_HDRS}
							${LIBJASPER_JP2_HDRS}
							${LIBJASPER_JPC_HDRS}
							${LIBJASPER_JPG_HDRS}
							${LIBJASPER_MIF_HDRS}
							${LIBJASPER_PGX_HDRS}
							${LIBJASPER_PNM_HDRS}
							${LIBJASPER_RAS_HDRS})
						

add_library(jasper SHARED ${LIBJASPER_SRCS} ${LIBJASPER_HDRS})
add_library(jasperstatic STATIC ${LIBJASPER_SRCS} ${LIBJASPER_HDRS})

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(TARGETS jasper jasperstatic
        RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
endif()

if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(FILES ${LIBJASPER_PUBLIC_HDRS} DESTINATION "${INSTALL_INC_DIR}")
endif()























