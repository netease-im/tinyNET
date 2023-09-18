cmake_minimum_required(VERSION 3.18)

project(tinyNET)

find_package(tinysak REQUIRED)

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
        target_link_libraries(${PROJECT_NAME} Winmm Ws2_32 Iphlpapi)
    endif ()
endif ()

target_link_libraries(${PROJECT_NAME} tinysak::tinysak)
