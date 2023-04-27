cmake_minimum_required(VERSION 3.14)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT DEFINED IDF_TARGET)
project(test-project CXX)
endif()

option(RING_BUFFER_DEFAULT_CXXFLAGS "enable default CXXFLAGS" OFF)
option(RING_BUFFER_BUILD_TESTS "build test cases for the ring buffer library" OFF)

option(RING_BUFFER_FEATURE_DETECT "try to detect if c++ features are supported" ON)
option(RING_BUFFER_NOEXCEPT "enable conditionally noexcept ring_buffer (c++11)" OFF)
option(RING_BUFFER_CONSTEXPR "enable constexpr ring_buffer (c++14)" OFF)
option(RING_BUFFER_CONSTEXPR_DESTRUCTORS "enable constexpr destructor for ring_buffer (c++20)" OFF)

if(RING_BUFFER_DEFAULT_CXXFLAGS)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
    include(${CMAKE_CURRENT_LIST_DIR}/detect/standard.cmake)
endif()

if(RING_BUFFER_FEATURE_DETECT)
    include(${CMAKE_CURRENT_LIST_DIR}/detect/features.cmake)
endif()

if(DEFINED IDF_TARGET)
    idf_component_register(INCLUDE_DIRS include)

    set_property(TARGET ${COMPONENT_LIB} PROPERTY CXX_STANDARD 23)

    set(RING_BUFFER_TARGET_NAME ${COMPONENT_TARGET})
else()
    set(RING_BUFFER_TARGET_NAME ${COMPONENT_TARGET})

    add_library(${RING_BUFFER_TARGET_NAME} INTERFACE)
    target_include_directories(${RING_BUFFER_TARGET_NAME} INTERFACE include/)
endif()

if(RING_BUFFER_CONSTEXPR)
    target_compile_definitions(${RING_BUFFER_TARGET_NAME} INTERFACE RING_BUFFER_CONSTEXPR)
endif()
if(RING_BUFFER_NOEXCEPT)
    target_compile_definitions(${RING_BUFFER_TARGET_NAME} INTERFACE RING_BUFFER_NOEXCEPT)
endif()
if(RING_BUFFER_CONSTEXPR_DESTRUCTORS)
    target_compile_definitions(${RING_BUFFER_TARGET_NAME} INTERFACE RING_BUFFER_CONSTEXPR_DESTRUCTORS)
endif()

if(NOT DEFINED IDF_TARGET)
    if(RING_BUFFER_BUILD_TESTS)
        add_subdirectory(test)
    endif()
endif()
