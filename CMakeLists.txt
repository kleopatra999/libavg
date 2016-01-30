project(libavg CXX C)
cmake_minimum_required(VERSION 3.2)

message(STATUS "CMAKE_SYSTEM_NAME: ${CMAKE_SYSTEM_NAME}")

if(${CMAKE_SYSTEM_NAME} MATCHES RaspberryPi)
    set(PLATFORM_RPI TRUE)
endif()
if(${CMAKE_SYSTEM_NAME} MATCHES Linux)
    set(PLATFORM_LINUX TRUE)
endif()
if(${CMAKE_SYSTEM_NAME} MATCHES Darwin)
    set(PLATFORM_MACOSX TRUE)
endif()
if(${CMAKE_SYSTEM_NAME} MATCHES Windows)
    set(PLATFORM_WINDOWS TRUE)
endif()

find_package(Boost 1.58 REQUIRED system thread)
find_package(PkgConfig REQUIRED)
pkg_search_module(LIBXML2 REQUIRED libxml-2.0 libxml2)
#pkg_search_module(GLES glesv2)
#pkg_search_module(EGL egl)
pkg_search_module(GDK_PIXBUF REQUIRED gdk-pixbuf-2.0)
pkg_search_module(SDL2 REQUIRED sdl2)
#pkg_search_module(PANGOFT2 REQUIRED pangoft2)
#pkg_search_module(RSVG REQUIRED librsvg-2.0)
#pkg_search_module(FONTCONFIG REQUIRED fontconfig)



# platform choices

if(${PLATFORM_LINUX})
    set(AVG_ENABLE_GLX_X11 TRUE)
endif()

if(${PLATFORM_MACOSX})
    # TODO In the original automake setup there was no further distinction
    #  besides the frameworks and options listed here.
    # I (payload) decided to postpone the refactoring into features
    #  until later™.
    # PLATFORM_MACOSX_* is currently only added in target graphics.
    # I think this propagates to all other targets,
    #  but isn't the proper way.
    set(AVG_ENABLE_CGL_QUARTZ TRUE)
    pkg_check_module(PLATFORM_MACOSX REQUIRED
        MultitouchSupport Cocoa CoreAudio AudioUnit
        ForceFeedback CoreVideo IOKit)
    list(APPEND PLATFORM_MACOSX_CFLAGS
        "-F/System/Library/PrivateFrameworks ")
    list(APPEND PLATFORM_MACOSX_INCLUDE_DIRS
        "/System/Library/PrivateFrameworks")
    # TODO payload: @uzadow, this is a flag in src/wrapper/Makefile.am. Needed?
    #   -read_only_relocs suppress
endif()

if(${PLATFORM_WINDOWS})
    message(FATAL_ERROR
        "Windows is not supported by the CMake build setup right now. "
        "Use the prebuilt Vistual Studio file './win/libavg.sln'. ")
endif()

if(${PLATFORM_RPI})
    set(ENV{PKG_CONFIG_PATH} ./pkg-config/rpi)
    set(AVG_ENABLE_EGL_BCM TRUE)
endif()



# feature configuration

if(${AVG_ENABLE_GLX_X11})
    pkg_check_modules(GLX_X11 REQUIRED x11 gl)
endif()
if(${AVG_ENABLE_CGL_QUARTZ})
    # pkg_check_modules in PLATFORM_MACOSX check
endif()
if(${AVG_ENABLE_EGL_BCM})
    pkg_check_modules(EGL_BCM REQUIRED egl bcm)
endif()

add_subdirectory(src)
