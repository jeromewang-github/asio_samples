#
# Copyright (c) 2015-2016 Marat Abrarov (abrarov@gmail.com)
#
# Distributed under the Boost Software License, Version 1.0. (See accompanying
# file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
#

cmake_minimum_required(VERSION 3.0)
project(ma_sp_singleton_test)

set(project_base_dir     "${PROJECT_SOURCE_DIR}")
set(cxx_headers_dir      "${project_base_dir}/include")
set(cxx_sources_dir      "${project_base_dir}/src")

set(cxx_compile_definitions )
set(cxx_headers )
set(cxx_sources )
set(cxx_libraries )
set(boost_components )

# Select used Boost libraries
find_package(Boost REQUIRED)
list(APPEND boost_components
    system
    date_time)
if(NOT(${Boost_VERSION} LESS 104700))
    list(APPEND boost_components
        chrono)
    set(ma_boost_has_chrono TRUE)
else()
    set(ma_boost_has_chrono FALSE)
endif()

# Attach used Boost libraries
find_package(Boost REQUIRED COMPONENTS ${boost_components})

# Enforce linkage with shared build of Boost C++ Libraries in case
# there is no request for linkage with static build
if(NOT Boost_USE_STATIC_LIBS)
    list(APPEND cxx_compile_definitions "BOOST_ALL_DYN_LINK")
endif()

# Google Test framework
if(NOT ma_own_gtest)
    find_package(GTest QUIET)
endif()

# MSVC 2012 doesn't work with Google Test and tuple
# (even with GTest own tuple implementation)
if (MSVC AND MSVC_VERSION GREATER 1600 AND MSVC_VERSION LESS 1800)
    list(APPEND cxx_compile_definitions GTEST_HAS_TR1_TUPLE=0)
endif()

list(APPEND cxx_sources
    "${cxx_sources_dir}/sp_singleton_test.cpp")

list(APPEND cxx_libraries
    ma_sp_singleton
    ma_shared_ptr_factory
    ma_thread_group
    ma_limited_int
    ma_compat
    ma_config
    ${Boost_LIBRARIES}
    ${GTEST_BOTH_LIBRARIES})

add_executable(${PROJECT_NAME}
    ${cxx_headers}
    ${cxx_sources})
target_compile_definitions(${PROJECT_NAME}
    PRIVATE
    ${cxx_compile_definitions})
target_include_directories(${PROJECT_NAME}
    PRIVATE
    ${Boost_INCLUDE_DIRS}
    ${GTEST_INCLUDE_DIRS}
    ${cxx_headers_dir})
target_link_libraries(${PROJECT_NAME}
    PRIVATE
    ${cxx_libraries})

if(NOT ma_no_cmake_dir_source_group)
    # Group files according to file path
    ma_dir_source_group("Header Files" "${cxx_headers_dir}" "${cxx_headers}")
    ma_dir_source_group("Source Files" "${cxx_sources_dir}" "${cxx_sources}")
endif()