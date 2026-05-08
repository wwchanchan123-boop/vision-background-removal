cmake_minimum_required(VERSION 3.28)

set(PLUGIN_NAME "vision-background-removal")
set(PLUGIN_VERSION "1.0.0")
set(PLUGIN_AUTHOR "Andreas Kuschner")

project(${PLUGIN_NAME} VERSION ${PLUGIN_VERSION})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_OBJCXX_STANDARD 17)
set(CMAKE_OBJCXX_STANDARD_REQUIRED ON)

if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
    set(CMAKE_OSX_DEPLOYMENT_TARGET "13.0")
    set(CMAKE_OBJCXX_FLAGS "${CMAKE_OBJCXX_FLAGS} -fobjc-arc")
endif()

# OBS Headers (从 deps 下载)
set(OBS_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/deps/obs-studio/libobs")

# Source files
set(PLUGIN_SOURCES
    src/plugin-main.mm
    src/background-removal-filter.mm
    src/vision-processor.mm
)

set(PLUGIN_HEADERS
    src/plugin-main.h
    src/background-removal-filter.h
    src/vision-processor.h
)

add_library(${PLUGIN_NAME} MODULE
    ${PLUGIN_SOURCES}
    ${PLUGIN_HEADERS}
)

target_include_directories(${PLUGIN_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${OBS_INCLUDE_DIR}
    ${OBS_INCLUDE_DIR}/util
    ${OBS_INCLUDE_DIR}/graphics
)

# macOS 系统框架（去掉 libobs.framework）
if(APPLE)
    find_library(COCOA_FRAMEWORK Cocoa REQUIRED)
    find_library(COREVIDEO_FRAMEWORK CoreVideo REQUIRED)
    find_library(COREMEDIA_FRAMEWORK CoreMedia REQUIRED)
    find_library(COREIMAGE_FRAMEWORK CoreImage REQUIRED)
    find_library(VISION_FRAMEWORK Vision REQUIRED)
    find_library(METAL_FRAMEWORK Metal REQUIRED)
    find_library(METALPERFORMANCESHADERS_FRAMEWORK MetalPerformanceShaders REQUIRED)
    find_library(ACCELERATE_FRAMEWORK Accelerate REQUIRED)
    find_library(VIDEOTOOLBOX_FRAMEWORK VideoToolbox REQUIRED)

    target_link_libraries(${PLUGIN_NAME} PRIVATE
        ${COCOA_FRAMEWORK}
        ${COREVIDEO_FRAMEWORK}
        ${COREMEDIA_FRAMEWORK}
        ${COREIMAGE_FRAMEWORK}
        ${VISION_FRAMEWORK}
        ${METAL_FRAMEWORK}
        ${METALPERFORMANCESHADERS_FRAMEWORK}
        ${ACCELERATE_FRAMEWORK}
        ${VIDEOTOOLBOX_FRAMEWORK}
    )

    set_target_properties(${PLUGIN_NAME} PROPERTIES
        BUNDLE TRUE
        BUNDLE_EXTENSION "plugin"
        MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Info.plist.in"
        PREFIX ""
        SUFFIX ".so"
    )
endif()

# Compiler options
target_compile_options(${PLUGIN_NAME} PRIVATE
    -Wall -Wextra -Wno-unused-parameter
    -Wno-gnu-anonymous-struct -Wno-nested-anon-types
)

# Package
if(APPLE)
    set(OBS_PLUGIN_DIR "$ENV{HOME}/Library/Application Support/obs-studio/plugins/${PLUGIN_NAME}")
endif()

install(TARGETS ${PLUGIN_NAME} BUNDLE DESTINATION "${OBS_PLUGIN_DIR}")
install(DIRECTORY data/ DESTINATION "${OBS_PLUGIN_DIR}/data")

include(CPack)
