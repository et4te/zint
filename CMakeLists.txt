# (c) 2008 by BogDan Vatra < bogdan@licentia.eu >

cmake_minimum_required(VERSION 2.6)
project(zint-package)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

#set(CMAKE_VERBOSE_MAKEFILE ON)
#comment or remove the above line before release

set (ZINT_VERSION_MAJOR 2)
set (ZINT_VERSION_MINOR 4)
set (ZINT_VERSION_RELEASE 2)
set (ZINT_VERSION "${ZINT_VERSION_MAJOR}.${ZINT_VERSION_MINOR}.${ZINT_VERSION_RELEASE}" )

add_definitions (-DZINT_VERSION=\"${ZINT_VERSION}\" -Wall)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules" )

include (SetPaths.cmake)

INCLUDE (CheckCXXCompilerFlag)

if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_GNUCC)
    check_cxx_compiler_flag("-Wall" CXX_COMPILER_FLAG_WALL)
    if (CXX_COMPILER_FLAG_WALL)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
    endif (CXX_COMPILER_FLAG_WALL)

    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
endif (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_GNUCC)

IF(APPLE)
    IF (UNIVERSAL) # TODO: make univeral binary
        IF(NOT ZINT_HAS_BEEN_RUN_BEFORE and UNIVERAL)
            IF(EXISTS /Developer/SDKs/MacOSX10.5.sdk OR EXISTS /SDKs/MacOSX10.5.sdk)
                SET(CMAKE_OSX_ARCHITECTURES "ppc;i386;ppc64;x86_64" CACHE STRING "Build architectures for OSX" FORCE)
                SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility-inlines-hidden -Wl -single_module " CACHE STRING "Flags used by the compiler during all build types." FORCE)
            ELSE(EXISTS /Developer/SDKs/MacOSX10.5.sdk OR EXISTS /SDKs/MacOSX10.5.sdk)
                IF(EXISTS /Developer/SDKs/MacOSX10.4u.sdk OR EXISTS /SDKs/MacOSX10.4u.sdk)
                    SET(CMAKE_OSX_ARCHITECTURES "ppc;i386" CACHE STRING "Build architectures for OSX" FORCE)
                    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility-inlines-hidden -Wl -single_module " CACHE STRING "Flags used by the compiler during all build types." FORCE)
               ENDIF(EXISTS /Developer/SDKs/MacOSX10.4u.sdk OR EXISTS /SDKs/MacOSX10.4u.sdk)
            ENDIF(EXISTS /Developer/SDKs/MacOSX10.5.sdk OR EXISTS /SDKs/MacOSX10.5.sdk)
            message("Build architectures for OSX:${CMAKE_OSX_ARCHITECTURES}")
        ENDIF(NOT ZINT_HAS_BEEN_RUN_BEFORE)

    ELSE (UNIVERSAL)
        SET(CMAKE_OSX_SYSROOT "/")
    ENDIF (UNIVERSAL)

    IF (PNG_FOUND)
         INCLUDE_DIRECTORIES(
            "${PNG_INCLUDE_DIR}"
         )
        SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lpng")
        SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lpng")
        SET(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -lpng")
    ENDIF (PNG_FOUND)

ENDIF(APPLE)

add_subdirectory(backend)
add_subdirectory(frontend)

find_package(Qt4)

if (QT4_FOUND)
    set( QT_USE_QTGUI   TRUE )
    set( QT_USE_QTUITOOLS TRUE )
    set( QT_USE_QTXML TRUE )
    include( ${QT_USE_FILE} )
    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${QT_INCLUDE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
        )
    add_subdirectory(backend_qt4)
    add_subdirectory(frontend_qt4)
endif(QT4_FOUND)

CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)

ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

# install(FILES cmake/modules/FindZint.cmake DESTINATION ${CMAKE_ROOT}/Modules COMPONENT Devel)

# This needs to be run very last so other parts of the scripts can take
# advantage of this.
IF(NOT ZINT_HAS_BEEN_RUN_BEFORE)
    SET(ZINT_HAS_BEEN_RUN_BEFORE 1 CACHE INTERNAL "Flag to track whether this is the first time running CMake or if CMake has been configured before")
ENDIF(NOT ZINT_HAS_BEEN_RUN_BEFORE)


