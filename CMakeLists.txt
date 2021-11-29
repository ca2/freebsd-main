cmake_minimum_required(VERSION 3.17)
project(basis)


set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)
message(STATUS "CMAKE_SYSTEM_NAME is ${CMAKE_SYSTEM_NAME}")


set(ROOT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})


add_compile_options("$<$<CONFIG:DEBUG>:-DDEBUG>")


if(${CMAKE_SYSTEM_NAME} STREQUAL "FreeBSD")
    set(FREEBSD TRUE)
    set(PLATFORM_NAME "freebsd")
    set(PLATFORM_POSIX TRUE)


    if($ENV{XDG_CURRENT_DESKTOP} STREQUAL "KDE")
        set(KDE_DESKTOP TRUE)
        set(WITH_XCB TRUE)
        add_compile_definitions(WITH_XCB=1)
    elseif($ENV{XDG_CURRENT_DESKTOP} STREQUAL "ubuntu:GNOME")
        set(GNOME_DESKTOP TRUE)
        set(WITH_X11 TRUE)
        add_compile_definitions(WITH_X11=1)
    elseif($ENV{XDG_CURRENT_DESKTOP} STREQUAL "GNOME")
        set(GNOME_DESKTOP TRUE)
        set(WITH_X11 TRUE)
        add_compile_definitions(WITH_X11=1)
    endif()

    include(FindPkgConfig)
    set(default_draw2d "draw2d_cairo")
    set(default_imaging "imaging_freeimage")
    set(default_write_text "write_text_pango")
    set(default_audio "audio_alsa")
    set(default_node "node_freebsd")




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


    set(APPLICATION_BUILD_RESOURCE_OBJECT_PRE "$(LD) -r -o ")
    set(APPLICATION_BUILD_RESOURCE_OBJECT_POS "/_matter.zip.o -z noexecstack --format=binary _matter.zip")


    if(GNOME_DESKTOP)

        list(APPEND app_common_dependencies
                desktop_environment_gnome
                windowing_x11)

    endif()

    if(KDE_DESKTOP)

        list(APPEND app_common_dependencies
                desktop_environment_kde
                windowing_xcb)


        set(WITH_XCB TRUE)
        add_compile_definitions(WITH_XCB=1)

        set(QT_MIN_VERSION "5.3.0")
        set(KF5_MIN_VERSION "5.2.0")

        # apt install extra-cmake-modules
        # dnf install extra-cmake-modules
        find_package(ECM 1.0.0 REQUIRED NO_MODULE)
        set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

        # apt install libkf5notifications-dev
        # dnf install kf5-knotifications-devel

        #include(KDEInstallDirs)
        #include(KDECMakeSettings)
        #include(KDECompilerSettings NO_POLICY_SCOPE)
        #    find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS
        # CoreAddons      # KAboutData
        #          I18n            # KLocalizedString
        #         WidgetsAddons   # KMessageBox
        #      Notifications
        #     )
        #include(FeatureSummary)

        # Find Qt modules
        #find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS
        #  Core    # QCommandLineParser, QStringLiteral
        #  Widgets # QApplication
        #  )
        find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS
           # CoreAddons      # KAboutData
           #          I18n            # KLocalizedString
           #         WidgetsAddons   # KMessageBox
           CoreAddons
           Notifications
           ConfigWidgets
           KIO
           IconThemes
           )
        find_package(LibKWorkspace CONFIG REQUIRED)

        # Find KDE modules

        #feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
        #        find_package(KDE5 REQUIRED)
        message(STATUS "Adding KDE/xcb dependency.")
        #        file (STRINGS $ENV{HOME}/__config/knotifications/cflags.txt knotifications_cflags)
        #        file (STRINGS $ENV{HOME}/__config/knotifications/libs.txt knotifications_libs)
        #        if(knotifications_cflags STREQUAL "")
        #            set(knotifications_cflags -I/usr/include/KF5/KNotifications)
        #        endif()
        #        if(knotifications_libs STREQUAL "")
        #            set(knotifications_cflags -I/usr/include/KF5/KNotifications)
        #        endif()

    endif()


    list(APPEND app_common_dependencies
       node_freebsd)


elseif(LINUX)



    set(APPLICATION_BUILD_RESOURCE_OBJECT_PRE "ld -r -b binary -o ")
    set(APPLICATION_BUILD_RESOURCE_OBJECT_POS "/_matter.zip.o _matter.zip")

    if(GNOME_DESKTOP)

        list(APPEND app_common_dependencies
                node_gnome
                windowing_x11)

    endif()

    if(KDE_DESKTOP)

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

set(APPLICATION_BUILD_HELPER "${ROOT_SOURCE_DIR}/archive/tool-${PLATFORM_NAME}/bin/application_build_helper")

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
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/platform-posix)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/platform-posix/_include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/third/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/archive/third-${PLATFORM_NAME}/third/include/include)


add_subdirectory(source/app/acme)
add_subdirectory(source/app/crypto_openssl)
add_subdirectory(archive/platform-posix/acme_posix)
add_subdirectory(archive/platform-freebsd/acme_freebsd)
add_subdirectory(source/console/application_build_helper)
add_subdirectory(source/console/hello)
add_subdirectory(source/console/helloearth)
add_subdirectory(source/app/apex)
add_subdirectory(archive/platform-posix/apex_posix)
add_subdirectory(archive/platform-freebsd/apex_freebsd)
add_subdirectory(source/app/aqua)
add_subdirectory(source/console/hellosolarsystem)
add_subdirectory(source/console/readlines)
add_subdirectory(archive/platform-posix/write_text_pango)
if(GNOME_DESKTOP)
add_subdirectory(archive/platform-posix/windowing_x11)
add_subdirectory(archive/platform-posix/node_gnome)
add_subdirectory(archive/platform-posix/node_gtk)
add_subdirectory(archive/platform-${PLATFORM_NAME}/desktop_environment_gnome)
endif()
if(KDE_DESKTOP)
add_subdirectory(archive/platform-posix/windowing_xcb)
add_subdirectory(archive/platform-posix/node_kde)
add_subdirectory(archive/platform-${PLATFORM_NAME}/desktop_environment_kde)
endif()
add_subdirectory(archive/platform-freebsd/node_freebsd)
add_subdirectory(source/app/aura)
add_subdirectory(archive/platform-posix/aura_posix)
add_subdirectory(archive/platform-freebsd/aura_freebsd)
add_subdirectory(source/app/draw2d_cairo)
add_subdirectory(source/app/app)
add_subdirectory(source/app/message_box)
add_subdirectory(source/app/gpu_opengl)
add_subdirectory(source/app/shader)
add_subdirectory(source/app/axis)
add_subdirectory(source/app/database_sqlite3)
add_subdirectory(source/app/experience_core)
add_subdirectory(source/app/base)
add_subdirectory(source/app-simple/application)




