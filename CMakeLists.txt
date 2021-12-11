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
    elseif($ENV{XDG_CURRENT_DESKTOP} STREQUAL "ubuntu:GNOME")
        set(GNOME_DESKTOP TRUE)
    elseif($ENV{XDG_CURRENT_DESKTOP} STREQUAL "GNOME")
        set(GNOME_DESKTOP TRUE)
    endif()

    include(FindPkgConfig)
    set(default_draw2d "draw2d_cairo")
    set(default_imaging "imaging_freeimage")
    set(default_write_text "write_text_pango")
    set(default_audio "audio_alsa")
    set(default_node "node_freebsd")
    set(default_music_midi "music_midi_rtmidi")

    set(FILE_SYSTEM_INOTIFY TRUE)
    #set(USE_KQUEUE_FILE_WATCHER_DIRECTLY TRUE)




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


set(default_gpu "gpu_opengl")


if(FREEBSD)


    if(GNOME_DESKTOP)

        list(APPEND app_common_dependencies
                desktop_environment_gnome
                windowing_x11)

        set(WITH_X11 TRUE)
        add_compile_definitions(WITH_X11=1)

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

set(APPLICATION_BUILD_HELPER "${EXECUTABLE_OUTPUT_PATH}/application_build_helper")

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
add_subdirectory(source/app/folder_zip)
add_subdirectory(source/app/app)
add_subdirectory(source/app/message_box)
add_subdirectory(source/app/gpu_opengl)
add_subdirectory(source/app/shader)
add_subdirectory(source/app/axis)
add_subdirectory(source/app/database_sqlite3)
add_subdirectory(source/app/experience_core)
add_subdirectory(source/app/base)
add_subdirectory(source/app-simple/application)
add_subdirectory(source/app/imaging_freeimage)
add_subdirectory(source/app-simple/drawing)
add_subdirectory(source/app-simple/networking_application)
add_subdirectory(source/app-simple/notify_drawing)
add_subdirectory(source/app/bred)
add_subdirectory(source/app/core)
add_subdirectory(source/app-simple/shader)
add_subdirectory(source/app-simple/form)
add_subdirectory(source/app-complex/drawing)
add_subdirectory(source/app/experience_tranquillum)
add_subdirectory(source/app-complex/form)
add_subdirectory(source/game/estamira)
add_subdirectory(source/game/light)
add_subdirectory(source/app/regular_expression_pcre2)
add_subdirectory(source/app-core/gcom)
add_subdirectory(source/app-core/hellomultiverse)
add_subdirectory(source/app-core/flag)
add_subdirectory(source/app-cidadedecuritiba/alphaclock)
add_subdirectory(source/app-core/clockverse)
add_subdirectory(source/app-core/projection)
add_subdirectory(source/app-core/desk)
add_subdirectory(source/app-core/api_ca2)
add_subdirectory(source/app-core/weather)
add_subdirectory(source/app-core/audio)
add_subdirectory(archive/platform-posix/audio_alsa)
add_subdirectory(source/app-core/audio_resample_swresample)
add_subdirectory(source/app-core/audio_decode_wave)
add_subdirectory(source/app-core/audioaura)
add_subdirectory(source/app-core/auraclick)
add_subdirectory(source/app-core/auratype)
add_subdirectory(source/app-core/chronometer)
add_subdirectory(source/app-veriwell/audio_decode_libmpg123)
add_subdirectory(source/app-core/api_azure)
add_subdirectory(source/app-core/api_google)
add_subdirectory(source/app-core/text_to_speech_azure)
add_subdirectory(source/app-core/text_to_speech_google)
add_subdirectory(source/app-core/clockverse_audio)
add_subdirectory(source/app-core/timer)
add_subdirectory(source/app-core/todo)
add_subdirectory(source/app-core/multimedia)
add_subdirectory(source/app-veriwell/multimedia)
#add_subdirectory(source/app-posix/music_midi_alsa)
add_subdirectory(source/app-veriwell/music_midi_rtmidi)
add_subdirectory(source/app/experience_anthill)
add_subdirectory(source/app-veriwell/waven)
add_subdirectory(source/app-veriwell/video_decode_ffmpeg)
add_subdirectory(source/app-veriwell/verisimplevideo)
add_subdirectory(source/app/compress_zlib)
add_subdirectory(source/app-veriwell/musical_player_lite)
add_subdirectory(source/app-veriwell/musical_player)
add_subdirectory(source/app-veriwell/keyboard)
add_subdirectory(source/app/programming)
add_subdirectory(source/app-core/announcecom)
add_subdirectory(source/app-core/resident_client)
add_subdirectory(source/app-core/resident_desktop)
add_subdirectory(source/app-core/addressbook)
add_subdirectory(source/app-core/file_manager)
add_subdirectory(source/app-core/hash_hub)
add_subdirectory(source/app-core/note)
add_subdirectory(source/app-core/photo)
add_subdirectory(source/app-core/picture)
add_subdirectory(source/game/game_of_life)
add_subdirectory(source/game/tictactoe)
add_subdirectory(source/game/tetris)
add_subdirectory(source/game/arkanoid)
add_subdirectory(source/game/sokoban)
add_subdirectory(source/game/pacman)
add_subdirectory(source/game/coincube)
add_subdirectory(source/game/arch001)




