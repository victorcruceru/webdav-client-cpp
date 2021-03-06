#***************************************************************************
#                         __    __   _____       _____
#   Project              |  |  |  | |     \     /  ___|
#                        |  |__|  | |  |\  \   /  /
#                        |        | |  | )  ) (  (
#                        |   /\   | |  |/  /   \  \___
#                         \_/  \_/  |_____/     \_____|
#
# Copyright (C) 2018, The WDC Project, <rusdevops@gmail.com>, et al.
#
# This software is licensed as described in the file LICENSE, which
# you should have received as part of this distribution.
#
# You may opt to use, copy, modify, merge, publish, distribute and/or sell
# copies of the Software, and permit persons to whom the Software is
# furnished to do so, under the terms of the LICENSE file.
#
# This software is distributed on an "AS IS" basis, WITHOUT WARRANTY OF ANY
# KIND, either express or implied.
#
############################################################################

cmake_minimum_required(VERSION 3.4)


set(WDC_VERSION_MAJOR 1)
set(WDC_VERSION_MINOR 1)
set(WDC_VERSION_PATCH 5)
set(WDC_VERSION_TWEAK 1)
set(WDC_VERSION ${WDC_VERSION_MAJOR}.${WDC_VERSION_MINOR}.${WDC_VERSION_PATCH})

project(WDC VERSION ${WDC_VERSION})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" OFF)
option(BUILD_EXAMPLES "Build Examples" OFF)
option(WDC_VERBOSE "Print verbose information" OFF)

find_package(Boost REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(CURL REQUIRED)
find_package(pugixml REQUIRED)

file(GLOB WDC_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/sources/*.cpp")

add_library(libwdc ${${PROJECT_NAME}_SOURCES})
set_target_properties(libwdc PROPERTIES PREFIX "")
set_target_properties(libwdc PROPERTIES IMPORT_PREFIX "")
if(WDC_VERBOSE)
  target_compile_definitions(libwdc PUBLIC WDC_VERBOSE=1)
endif()

target_link_libraries(libwdc
  PUBLIC OpenSSL::SSL OpenSSL::Crypto CURL::libcurl pugixml
)

target_include_directories(libwdc
  PRIVATE $<TARGET_PROPERTY:Boost::boost,INTERFACE_INCLUDE_DIRECTORIES>
)
target_include_directories(libwdc PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(CONFIG_INSTALL_DIR "lib/cmake/${PROJECT_NAME}")
set(INCLUDE_INSTALL_DIR "include")

set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${VERSION_CONFIG}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
  "${PROJECT_CONFIG}"
   INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(TARGETS libwdc
  EXPORT "${TARGETS_EXPORT_NAME}"
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)

install(
  FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
  DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(EXPORT "${TARGETS_EXPORT_NAME}"
  NAMESPACE "${NAMESPACE}"
  DESTINATION "${CONFIG_INSTALL_DIR}"
)

if(BUILD_TESTS)
  find_package(Boost REQUIRED system filesystem)

  enable_testing()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-psabi")
  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check libwdc Boost::filesystem Boost::system pthread)

  if(${CMAKE_BUILD_TYPE} MATCHES "Coverage")
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")
    include(CodeCoverage)

    set(CMAKE_CXX_FLAGS "-g -O0 -Wall -fprofile-arcs -ftest-coverage")
    set(LCOV_REMOVE_EXTRA "'tests/*'")
    add_executable(unit_tests ${${PROJECT_NAME}_SOURCES} ${${PROJECT_NAME}_TEST_SOURCES})
    target_link_libraries(unit_tests Boost::filesystem Boost::system libwdc OpenSSL::SSL OpenSSL::Crypto CURL::libcurl pugixml)

    setup_target_for_coverage(unit_tests_coverage unit_tests coverage)
  else()
    add_test(NAME unit_tests COMMAND check "-s" "-r" "compact" "--use-colour" "yes")
  endif()

endif()

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    set(EXAMPLE_TARGET_NAME example_${EXAMPLE_NAME})
    add_executable(${EXAMPLE_TARGET_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_TARGET_NAME} libwdc pthread)
    set_target_properties(${EXAMPLE_TARGET_NAME} PROPERTIES OUTPUT_NAME ${EXAMPLE_NAME})
    install(TARGETS ${EXAMPLE_TARGET_NAME}
            RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

include(CPackConfig.cmake)
