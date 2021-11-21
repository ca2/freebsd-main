cmake_minimum_required(VERSION 3.17)
project(basis)


set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)
message(STATUS "CMAKE_SYSTEM_NAME is ${CMAKE_SYSTEM_NAME}")


add_compile_options("$<$<CONFIG:DEBUG>:-DDEBUG>")


if(${CMAKE_SYSTEM_NAME} STREQUAL "FreeBSD")
    set(FREEBSD TRUE)
    set(PLATFORM_NAME "freebsd")
    set(PLATFORM_POSIX TRUE)
    include(FindPkgConfig)
    set(default_draw2d "draw2d_cairo")
    set(default_imaging "imaging_freeimage")
    set(default_write_text "write_text_pango")
    set(default_audio "audio_alsa")
    set(default_node "node_freebsd")


    if($ENV{XDG_CURRENT_DESKTOP} STREQUAL "KDE")
        set(FREEBSD_KDE TRUE)
    elseif($ENV{XDG_CURRENT_DESKTOP} STREQUAL "GNOME")
        set(FREEBSD_GNOME TRUE)
    endif()


elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set(LINUX TRUE)
    set(PLATFORM_NAME "linux")
    set(PLATFORM_POSIX TRUE)
    include(FindPkgConfig)
    set(default_draw2d "draw2d_cairo")
    set(default_imaging "imaging_freeimage")
    set(default_write_text "write_text_pango")
    set(default_audio "audio_alsa")
elseif(WIN32)
    set(PLATFORM_NAME "windows")
    set(default_draw2d "draw2d_gdiplus")
    set(default_imaging "imaging_wic")
    set(default_write_text "write_text_win32")
    set(default_audio "audio_mmsystem")
endif()

if(FREEBSD)

    if(FREEBSD_GNOME)

        list(APPEND app_common_dependencies
                node_gnome
                windowing_x11)

    endif()

    if(FREEBSD_KDE)

        list(APPEND app_common_dependencies
                node_kde
                windowing_xcb)

    endif()

elseif(LINUX OR FREEBSD)

    if(LINUX_GNOME)

        list(APPEND app_common_dependencies
                node_gnome
                windowing_x11)

    endif()

    if(LINUX_KDE)

        list(APPEND app_common_dependencies
                node_kde
                windowing_xcb)

    endif()

elseif(WIN32)

    set(app_common_dependencies)

endif()


if (MSVC)
    set(LIBCXX_TARGETING_MSVC ON)
else ()
    set(LIBCXX_TARGETING_MSVC OFF)
endif ()


add_compile_definitions(UNICODE)
add_compile_definitions(_UNICODE)


set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/time-${PLATFORM_NAME}/cmake)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/time-${PLATFORM_NAME}/cmake)


link_directories(${LIBRARY_OUTPUT_PATH})
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/storage-${PLATFORM_NAME}/library/x64/basis)
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/storage-${PLATFORM_NAME}/third/library/x64/basis)


include_directories($ENV{HOME}/__config)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source/app)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source/app/_include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/platform-${PLATFORM_NAME})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/platform-${PLATFORM_NAME}/_include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/third/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/third/include/include)


add_subdirectory(source/app/acme)
add_subdirectory(source/app/crypto_openssl)
add_subdirectory(archive/platform-posix/acme_posix)
add_subdirectory(archive/platform-freebsd/acme_freebsd)
add_subdirectory(source/console/hello)
add_subdirectory(source/console/helloearth)
add_subdirectory(source/app/apex)
add_subdirectory(archive/platform-posix/apex_posix)
add_subdirectory(archive/platform-freebsd/apex_freebsd)
add_subdirectory(source/app/aqua)
add_subdirectory(source/console/hellosolarsystem)
add_subdirectory(source/console/readlines)
add_subdirectory(archive/platform-posix/write_text_pango)
add_subdirectory(archive/platform-posix/windowing_x11)
add_subdirectory(archive/platform-posix/windowing_xcb)
add_subdirectory(archive/platform-posix/node_gnome)
add_subdirectory(archive/platform-posix/node_kde)
add_subdirectory(archive/platform-freebsd/node_freebsd)
add_subdirectory(source/app/aura)
add_subdirectory(source/app/draw2d_cairo)
add_subdirectory(source/app/app)




