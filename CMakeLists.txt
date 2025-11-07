cmake_minimum_required(VERSION 3.12)

# Extract version from libmseed.h
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/core/libmseed.h" VERSION_LINE
     REGEX "LIBMSEED_VERSION")
string(REGEX MATCH "[0-9]+\\.[0-9]+\\.[0-9]+" LIBMSEED_VERSION "${VERSION_LINE}")

project(libmseed
        VERSION ${LIBMSEED_VERSION}
        DESCRIPTION "Read and write miniSEED data records (seismological format)"
        LANGUAGES C)

# Build options
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_STATIC_LIBS "Build static libraries" ON)
option(BUILD_EXAMPLES "Build example programs" OFF)
option(BUILD_TESTS "Build test suite" OFF)
option(LIBMSEED_URL "Enable URL support via libcurl" OFF)

# Set default build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# C standard
# Use C11 or later to avoid typedef redefinition warnings in TAU test framework
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Build core library
add_subdirectory(core)

# Build examples
if(BUILD_EXAMPLES)
    add_subdirectory(example)
endif()

# Build tests
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()

# Create and install config file
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Generate the config file
configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/libmseedConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/libmseedConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libmseed
)

# Generate the version file
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/libmseedConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Install config files
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/libmseedConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/libmseedConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libmseed
)

# Generate and install pkg-config file
set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(EXEC_PREFIX ${CMAKE_INSTALL_PREFIX})
set(LIBDIR ${CMAKE_INSTALL_FULL_LIBDIR})
set(INCLUDEDIR ${CMAKE_INSTALL_FULL_INCLUDEDIR})
set(VERSION ${PROJECT_VERSION})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/mseed.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/mseed.pc
    @ONLY
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/mseed.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

# Install documentation
install(FILES README.md INSTALL.md LICENSE ChangeLog
    DESTINATION ${CMAKE_INSTALL_DOCDIR}
    OPTIONAL
)

# Install examples source code
install(DIRECTORY example/
    DESTINATION ${CMAKE_INSTALL_DOCDIR}/examples
    FILES_MATCHING PATTERN "*.c" PATTERN "*.h"
    PATTERN "Makefile*" EXCLUDE
    PATTERN "CMakeLists.txt" EXCLUDE
)

# Print configuration summary
message(STATUS "")
message(STATUS "libmseed ${PROJECT_VERSION} Configuration Summary:")
message(STATUS "  Build type:           ${CMAKE_BUILD_TYPE}")
message(STATUS "  C Compiler:           ${CMAKE_C_COMPILER}")
message(STATUS "  Install prefix:       ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  Build shared library: ${BUILD_SHARED_LIBS}")
message(STATUS "  Build static library: ${BUILD_STATIC_LIBS}")
message(STATUS "  Build examples:       ${BUILD_EXAMPLES}")
message(STATUS "  Build tests:          ${BUILD_TESTS}")
message(STATUS "  URL support:          ${LIBMSEED_URL}")
message(STATUS "")
