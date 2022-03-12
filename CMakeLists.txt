cmake_minimum_required(VERSION 3.16)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchain_gcc-arm-none-eabi.cmake)

#-------------------------------------------------------------------------------
# Update Git Submodules
#-------------------------------------------------------------------------------
find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE GIT_SUBMOD_RESULT
        OUTPUT_QUIET
    )
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()

#-------------------------------------------------------------------------------
# Project Definition
#-------------------------------------------------------------------------------
project(bd0 VERSION 0.1 LANGUAGES C CXX ASM)

#-------------------------------------------------------------------------------
# Embedded compiler options
#-------------------------------------------------------------------------------
set(CMAKE_C_FLAGS   "-std=gnu17 ${TARGET_FLAGS} ${C_CXX_FLAGS}" CACHE INTERNAL "")
set(CMAKE_CXX_FLAGS "-std=gnu++17 ${TARGET_FLAGS} ${C_CXX_FLAGS}" CACHE INTERNAL "")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS_INIT} ${TARGET_FLAGS}" CACHE INTERNAL "")
set(CMAKE_C_FLAGS_RELEASE   "-O3" CACHE INTERNAL "")
set(CMAKE_CXX_FLAGS_RELEASE "-O3" CACHE INTERNAL "")
set(CMAKE_ASM_FLAGS_RELEASE "-O3" CACHE INTERNAL "")
set(CMAKE_C_FLAGS_DEBUG     "-Og -g3 -gdwarf-4" CACHE INTERNAL "")
set(CMAKE_CXX_FLAGS_DEBUG   "-Og -g3 -gdwarf-4" CACHE INTERNAL "")
set(CMAKE_ASM_FLAGS_DEBUG   "-Og -g3 -gdwarf-4" CACHE INTERNAL "")

add_subdirectory(software)
