cmake_minimum_required(VERSION 3.16)
project(freebsd-main)


include(source/app/_.cmake)


add_subdirectory(source/app/launch)
add_subdirectory(source/app/store)
