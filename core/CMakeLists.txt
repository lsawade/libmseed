# Library source files
set(LIB_SRCS
    fileutils.c
    genutils.c
    msio.c
    lookup.c
    yyjson.c
    msrutils.c
    extraheaders.c
    pack.c
    packdata.c
    tracelist.c
    gmtime64.c
    crc32c.c
    parseutils.c
    unpack.c
    unpackdata.c
    selection.c
    logging.c
)

# Public header files
set(PUBLIC_HEADERS
    libmseed.h
)

# Internal header files (for reference, not installed)
set(INTERNAL_HEADERS
    extraheaders.h
    gmtime64.h
    mseedformat.h
    msio.h
    packdata.h
    unpack.h
    unpackdata.h
    yyjson.h
)

# Handle URL support with libcurl
if(LIBMSEED_URL)
    find_package(CURL REQUIRED)
    if(CURL_FOUND)
        add_compile_definitions(LIBMSEED_URL)
        set(LIBMSEED_URL_LIBS CURL::libcurl)
        message(STATUS "URL support enabled with libcurl ${CURL_VERSION_STRING}")
    endif()
endif()

# Create library targets
if(BUILD_SHARED_LIBS)
    add_library(mseed_shared SHARED ${LIB_SRCS})

    # Set library properties
    set_target_properties(mseed_shared PROPERTIES
        OUTPUT_NAME mseed
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        PUBLIC_HEADER "${PUBLIC_HEADERS}"
        C_VISIBILITY_PRESET default
        VISIBILITY_INLINES_HIDDEN OFF
    )

    # Platform-specific settings
    if(APPLE)
        set_target_properties(mseed_shared PROPERTIES
            INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
            MACOSX_RPATH ON
            BUILD_WITH_INSTALL_RPATH OFF
        )
    elseif(UNIX)
        # Use version script on Linux for symbol visibility
        if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/libmseed.map")
            set_target_properties(mseed_shared PROPERTIES
                LINK_FLAGS "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/libmseed.map"
            )
        endif()
    endif()

    # Position independent code
    target_compile_options(mseed_shared PRIVATE -fPIC)

    # Link libraries
    if(LIBMSEED_URL)
        target_link_libraries(mseed_shared PRIVATE ${LIBMSEED_URL_LIBS})
    endif()

    # Include directories
    # Users will include as: #include <libmseed/libmseed.h>
    target_include_directories(mseed_shared
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:include>
    )

    # Add to install
    list(APPEND INSTALL_TARGETS mseed_shared)
endif()

if(BUILD_STATIC_LIBS)
    add_library(mseed_static STATIC ${LIB_SRCS})

    # Set library properties
    set_target_properties(mseed_static PROPERTIES
        OUTPUT_NAME mseed
        VERSION ${PROJECT_VERSION}
        PUBLIC_HEADER "${PUBLIC_HEADERS}"
    )

    # On Windows, avoid naming conflict between static and shared lib
    if(WIN32 AND BUILD_SHARED_LIBS)
        set_target_properties(mseed_static PROPERTIES OUTPUT_NAME mseed_static)
    endif()

    # Link libraries
    if(LIBMSEED_URL)
        target_link_libraries(mseed_static PRIVATE ${LIBMSEED_URL_LIBS})
    endif()

    # Include directories
    target_include_directories(mseed_static
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:include>
    )

    # Add to install
    list(APPEND INSTALL_TARGETS mseed_static)
endif()

# Create alias target for easier integration
if(BUILD_SHARED_LIBS)
    add_library(mseed::mseed ALIAS mseed_shared)
elseif(BUILD_STATIC_LIBS)
    add_library(mseed::mseed ALIAS mseed_static)
endif()

# Installation
include(GNUInstallDirs)

# Install library targets
# Headers will be installed to include/ (libmseed.h at top level)
install(TARGETS ${INSTALL_TARGETS}
    EXPORT libmseedTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Export targets to build tree (allows using the build directory directly)
export(EXPORT libmseedTargets
    FILE "${CMAKE_BINARY_DIR}/libmseedTargets.cmake"
    NAMESPACE mseed::
)

# Install export file
install(EXPORT libmseedTargets
    FILE libmseedTargets.cmake
    NAMESPACE mseed::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libmseed
)

# Propagate INSTALL_TARGETS to parent scope
set(INSTALL_TARGETS ${INSTALL_TARGETS} PARENT_SCOPE)
