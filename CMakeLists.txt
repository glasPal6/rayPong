cmake_minimum_required(VERSION 3.11)
project(rayPong)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_BUILD_TYPE Release)
# set(CMAKE_BUILD_TYPE Debug)
# set(CMAKE_BUILD_TYPE RelWithDebInfo)

set(SOURCES
    src/main.c
)

add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME}
PRIVATE
    include/
    include/raylib
)

# ----------------------------------------------
# Raylib
# ----------------------------------------------
set(RAYLIB_LOCAL_LIB "${CMAKE_SOURCE_DIR}/lib/libraylib.a")

if(EXISTS ${RAYLIB_LOCAL_LIB})
    message(STATUS "Using local Raylib from ${CMAKE_SOURCE_DIR}/lib")

    target_link_libraries(${PROJECT_NAME}
    PRIVATE
        ${RAYLIB_LOCAL_LIB}
    )

    if(APPLE)
        target_link_libraries(${PROJECT_NAME}
        PRIVATE
            ${CMAKE_SOURCE_DIR}/lib/libEGL.dylib
            ${CMAKE_SOURCE_DIR}/lib/libGLESv2.dylib
            "-framework CoreVideo"
            "-framework IOKit"
            "-framework CoreFoundation"
            "-framework Cocoa"
        )
    elseif(UNIX AND NOT APPLE)
        if(EXISTS ${CMAKE_SOURCE_DIR}/lib/libEGL.so)
            target_link_libraries(${PROJECT_NAME}
            PRIVATE
                ${CMAKE_SOURCE_DIR}/lib/libEGL.so
                ${CMAKE_SOURCE_DIR}/lib/libGLESv2.so
            )
        endif()
    endif()
else()
    message(STATUS "Local Raylib not found, downloading...")

    set(RAYLIB_VERSION 5.0)

    include(FetchContent)
    FetchContent_Declare(
        raylib
        DOWNLOAD_EXTRACT_TIMESTAMP OFF
        URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
    )
    FetchContent_GetProperties(raylib)
    if (NOT raylib_POPULATED)
        set(FETCHCONTENT_QUIET NO)
        FetchContent_Populate(raylib)
        set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
        add_subdirectory(${raylib_SOURCE_DIR} ${raylib_BINARY_DIR})
    endif()

    target_link_libraries(${PROJECT_NAME}
    PRIVATE
        raylib
    )
endif()

# Copy ANGLE dylibs next to executable on macOS
if(APPLE AND EXISTS ${CMAKE_SOURCE_DIR}/lib/libEGL.dylib)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${CMAKE_SOURCE_DIR}/lib/libEGL.dylib
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/libEGL.dylib
    )
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${CMAKE_SOURCE_DIR}/lib/libGLESv2.dylib
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/libGLESv2.dylib
    )
endif()

# Copy assets next to executable
if(EXISTS ${CMAKE_SOURCE_DIR}/src/assets)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
            ${CMAKE_SOURCE_DIR}/src/assets
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/assets
    )
endif()

# ----------------------------------------------
# Compiler options
# ----------------------------------------------

target_compile_options(${PROJECT_NAME}
PRIVATE
    -Werror
    -Wall
    -Wpedantic
    -Wconversion
    -Wextra
    -O3
    -mtune=native
)

if(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME}
    PRIVATE
        m
        pthread
        dl
    )
elseif(APPLE)
    target_link_libraries(${PROJECT_NAME}
    PRIVATE
        m
        pthread
    )
endif()
