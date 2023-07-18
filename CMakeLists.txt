cmake_minimum_required(VERSION 3.18)

project(tinyNET)

set(CONAN_DISABLE_CHECK_COMPILER ON)

if (NOT EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
    if (NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
        message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
        file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/master/conan.cmake" "${CMAKE_BINARY_DIR}/conan.cmake")
    endif ()
    include(${CMAKE_BINARY_DIR}/conan.cmake)
    if (CONAN_PROFILE_BUILD AND CONAN_PROFILE_HOST)
        conan_cmake_install(PATH_OR_REFERENCE ${CMAKE_CURRENT_LIST_DIR} BUILD missing
                            SETTINGS_HOST build_type=${CMAKE_BUILD_TYPE}
                            SETTINGS_BUILD build_type=${CMAKE_BUILD_TYPE}
                            PROFILE_BUILD ${CONAN_PROFILE_BUILD}
                            PROFILE_HOST ${CONAN_PROFILE_HOST})
    else ()
        conan_cmake_autodetect(settings)
        conan_cmake_install(PATH_OR_REFERENCE ${CMAKE_CURRENT_LIST_DIR} BUILD missing SETTINGS ${settings})
    endif ()
endif ()
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

add_definitions(-DTINYNET_IMPORTS_IGNORE)
add_definitions(-DTINYSAK_IMPORTS_IGNORE)

if (BUILD_SHARED_LIBS)
    add_definitions(-DTINYNET_EXPORTS)
endif ()

if(UNIX)
    if(NOT APPLE)
        add_link_options("LINKER:--exclude-libs,ALL" "LINKER:--as-needed")
        set(CMAKE_POSITION_INDEPENDENT_CODE ON)
    endif()
    add_compile_options(-fvisibility=hidden)
endif()

if (IOS)
    enable_language(OBJC)
    add_definitions(-D__OBJC__)
endif ()

include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/src
    ${CMAKE_CURRENT_LIST_DIR}/src/third_party
    ${CMAKE_CURRENT_LIST_DIR}/src/third_party/iphone/include
)

file(GLOB_RECURSE TINY_NET_SOURCE_WIN src "*win32*.*")
file(GLOB_RECURSE TINY_NET_SOURCE src/*.h src/*.c)
file(GLOB_RECURSE THIY_NET_THIRD_PARTY src/third_party/*)

if (NOT WIN32)
    list(REMOVE_ITEM TINY_NET_SOURCE ${TINY_NET_SOURCE_WIN})
endif ()
list(REMOVE_ITEM TINY_NET_SOURCE ${THIY_NET_THIRD_PARTY})

if (IOS)
    set_source_files_properties(${TINY_NET_SOURCE} PROPERTIES LANGUAGE OBJC)
endif ()

add_library(${PROJECT_NAME} ${TINY_NET_SOURCE})

if (BUILD_SHARED_LIBS)
    if (WIN32)
        target_link_libraries(${PROJECT_NAME} tinySAK Winmm Ws2_32 Iphlpapi)
    endif ()
endif ()
