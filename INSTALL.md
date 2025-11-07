# Building and Installing libmseed

The library requires that C99 integer types are available on the target
computer. Specifically the int8_t, int16_t, int32_t, int64_t and their unsigned
counterpart types.

## Quick Start with CMake (Recommended)

### Building the library

```bash
# Configure the build
cmake -B build -S .

# Build the library
cmake --build build

# Install the library (default to /usr/local)
cmake --install build
```

### Using in another CMake project

After installing libmseed, you can use it in your CMake project:

```cmake
# Find the package
find_package(libmseed REQUIRED)

# Link against your target
add_executable(myapp main.c)
target_link_libraries(myapp PRIVATE mseed::mseed)
```

## CMake Build Options

The following options can be configured during the CMake configuration step:

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_SHARED_LIBS` | `ON` | Build shared libraries |
| `BUILD_STATIC_LIBS` | `ON` | Build static libraries |
| `BUILD_EXAMPLES` | `OFF` | Build example programs |
| `BUILD_TESTS` | `OFF` | Build test suite |
| `LIBMSEED_URL` | `OFF` | Enable URL support via libcurl |

### Examples

Build only static library:
```bash
cmake -B build -DBUILD_SHARED_LIBS=OFF -DBUILD_STATIC_LIBS=ON
```

Build with examples and tests:
```bash
cmake -B build -DBUILD_EXAMPLES=ON -DBUILD_TESTS=ON
cmake --build build
```

Enable URL support with libcurl:
```bash
cmake -B build -DLIBMSEED_URL=ON
```

Custom installation prefix:
```bash
cmake -B build -DCMAKE_INSTALL_PREFIX=/opt/libmseed
cmake --build build
cmake --install build
```

## Installation Layout

The install step will install:
- Library files (static and/or shared) to `lib/`
- Header file (`libmseed.h`) to `include/`
- CMake config files to `lib/cmake/libmseed/`
- pkg-config file to `lib/pkgconfig/`
- Documentation files to `share/doc/libmseed/`
- Example source code to `share/doc/libmseed/examples/`

## Using libmseed in Your Project

### Method 1: CMake find_package (Recommended)

After installing libmseed, you can use it in your CMake project:

```cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject)

# Find libmseed
find_package(libmseed REQUIRED)

# Create your executable
add_executable(myapp main.c)

# Link against libmseed
target_link_libraries(myapp PRIVATE mseed::mseed)
```

If libmseed is installed in a non-standard location:
```bash
cmake -B build -DCMAKE_PREFIX_PATH=/opt/libmseed
```

Or point directly to the CMake config directory:
```bash
cmake -B build -Dlibmseed_DIR=/opt/libmseed/lib/cmake/libmseed
```

### Method 2: Using as a subdirectory

You can also include libmseed directly in your project:

```cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject)

# Add libmseed as subdirectory
add_subdirectory(external/libmseed)

# Create your executable
add_executable(myapp main.c)

# Link against libmseed
target_link_libraries(myapp PRIVATE mseed::mseed)
```

### Method 3: pkg-config

If you prefer pkg-config:

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBMSEED REQUIRED mseed)

add_executable(myapp main.c)
target_link_libraries(myapp PRIVATE ${LIBMSEED_LIBRARIES})
target_include_directories(myapp PRIVATE ${LIBMSEED_INCLUDE_DIRS})
```

Or from the command line:
```bash
gcc myapp.c $(pkg-config --cflags --libs mseed) -o myapp
```

## Running Tests

If you built with tests enabled:

```bash
cmake -B build -DBUILD_TESTS=ON
cmake --build build
cd build
ctest
```

## Integration with FetchContent

You can use CMake's FetchContent to automatically download and build libmseed:

```cmake
include(FetchContent)

FetchContent_Declare(
    libmseed
    GIT_REPOSITORY https://github.com/EarthScope/libmseed.git
    GIT_TAG        v3.1.11
)

FetchContent_MakeAvailable(libmseed)

add_executable(myapp main.c)
target_link_libraries(myapp PRIVATE mseed::mseed)
```

## URL Support

If the **LIBMSEED_URL** option is enabled during the build, the library will be compiled with support for reading from URLs. This requires that [libcurl](https://curl.se/) be installed on the target system.

To install libcurl:
```bash
# Ubuntu/Debian
sudo apt-get install libcurl4-openssl-dev

# macOS
brew install curl

# Fedora/RHEL
sudo dnf install libcurl-devel
```

Then build with URL support:
```bash
cmake -B build -DLIBMSEED_URL=ON
cmake --build build
```

## Troubleshooting

### Library not found
If CMake cannot find libmseed, ensure it's installed and either:
1. Set `CMAKE_PREFIX_PATH` to the installation directory
2. Set `libmseed_DIR` to the directory containing `libmseedConfig.cmake`

Example:
```bash
cmake -B build -DCMAKE_PREFIX_PATH=/usr/local
```

### Cross-compilation

CMake makes cross-compilation straightforward:

```bash
cmake -B build -DCMAKE_TOOLCHAIN_FILE=/path/to/toolchain.cmake
cmake --build build
```

## Windows

On Windows, CMake can generate Visual Studio project files or use other build systems:

```bash
# Generate Visual Studio solution
cmake -B build -G "Visual Studio 16 2019"

# Or use Ninja
cmake -B build -G Ninja
cmake --build build
```

The `libmseed.def` file in `core/` is provided for building Windows DLLs.

## Requirements

- CMake 3.12 or later
- C11 compatible compiler (C99 minimum, C11 recommended)
- Optional: libcurl for URL support
