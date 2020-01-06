cmake_minimum_required(VERSION 3.0)
project(B64 VERSION 1.2.1 LANGUAGES C)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(B64_BUILD_BASE64 "B64: Build base64 program" OFF)

add_library(b64 STATIC
    include/b64/cdecode.h
    include/b64/cencode.h
    include/b64/decode.h
    include/b64/encode.h
    src/cdecode.c
    src/cencode.c)

target_include_directories(b64
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

if(B64_BUILD_BASE64)
    enable_language(CXX)

    add_executable(base64
        base64/base64.cc)

    target_link_libraries(base64
        PRIVATE
            b64)

    target_compile_definitions(base64
        PRIVATE
            BUFFERSIZE=16777216)
endif()

write_basic_package_version_file(
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion)

if(NOT CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
    add_library(${PROJECT_NAME}::b64 ALIAS b64)

    if(B64_BUILD_BASE64)
        add_executable(${PROJECT_NAME}::base64 ALIAS base64)
    endif()

    file(WRITE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" "")
    set(${PROJECT_NAME}_DIR "${PROJECT_BINARY_DIR}" CACHE PATH
        "The directory containing a CMake configuration file for ${PROJECT_NAME}.")
    return()
endif()

configure_file(
    Config.cmake
    ${PROJECT_NAME}Config.cmake
    @ONLY)

install(
    TARGETS b64
    EXPORT ${PROJECT_NAME}Targets)

if(B64_BUILD_BASE64)
    install(
        TARGETS base64
        EXPORT ${PROJECT_NAME}Targets)
endif()

install(
    DIRECTORY include/b64
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(
    FILES
        AUTHORS
        BENCHMARKS
        CHANGELOG
        LICENSE
        README
        TODO
    DESTINATION ${CMAKE_INSTALL_DOCDIR})

install(
    DIRECTORY examples
    DESTINATION ${CMAKE_INSTALL_DOCDIR})

install(
    EXPORT ${PROJECT_NAME}Targets
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    NAMESPACE ${PROJECT_NAME}::)

install(
    FILES
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
