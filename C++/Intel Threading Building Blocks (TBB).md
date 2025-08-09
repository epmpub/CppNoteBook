#  Intel Threading Building Blocks (TBB)

The Intel Threading Building Blocks (TBB) memory allocator is a high-performance allocator designed specifically for multi-threaded applications. Here's a summary of its key features and usage:

## Key Features of Intel TBB Memory Allocator

- **Thread-Scalable**: Designed to reduce memory allocation contention in multi-threaded programs
- **Cache-Friendly**: Optimizes memory access patterns for modern CPU architectures
- **False Sharing Avoidance**: Prevents performance degradation from false sharing
- **Seamless Integration**: Can replace standard allocators with minimal code changes
- **Memory Pools**: Uses thread-local memory pools to minimize synchronization overhead

## Basic Usage in C++

```cpp
#include <tbb/scalable_allocator.h>
#include <vector>

int main() {
    // Using TBB allocator with STL containers
    std::vector<int, tbb::scalable_allocator<int>> vec;
    
    // Continue using vec as a normal vector
    vec.push_back(42);
    
    return 0;
}
```

## Global Replacement of malloc/free

```cpp
#include <tbb/scalable_allocator.h>

// Replace global operators new and delete
void* operator new(size_t size) {
    return scalable_malloc(size);
}

void operator delete(void* ptr) noexcept {
    scalable_free(ptr);
}

// Your code will now use TBB allocator by default
```

## When to Use It

The TBB allocator is particularly beneficial for:

- Multi-threaded applications with intensive memory allocation/deallocation
- Programs that create and destroy many objects of various sizes
- High-performance computing applications where allocation bottlenecks exist

For optimal performance, you'll need to link against the TBB library when compiling your application.



## How to use

# Installing Intel Threading Building Blocks (TBB)

Installing Intel TBB varies by platform. Here are instructions for the most common operating systems:

## Windows

### Using vcpkg:

```
vcpkg install tbb
```

### Using pre-built binaries:

1. Download the latest release from the official oneAPI Base Toolkit from Intel's website
2. Run the installer and select TBB components
3. Add the TBB bin directory to your PATH environment variable

## macOS

### Using Homebrew:

```
brew install tbb
```

## Linux

### Ubuntu/Debian:

```
sudo apt-get update
sudo apt-get install libtbb-dev
```

### Fedora/RHEL/CentOS:

```
sudo dnf install tbb-devel
```

### Arch Linux:

```
sudo pacman -S intel-tbb
```

## Building from Source (all platforms)

1. Clone the repository:

```
git clone https://github.com/oneapi-src/oneTBB.git
```

1. Create a build directory:

```
cd oneTBB
mkdir build && cd build
```

1. Configure with CMake:

```
cmake ..
```

1. Build:

```
cmake --build .
```

1. Install:

```
cmake --install .
```

## Verifying the Installation

Create a simple test program:

```cpp
#include <tbb/tbb.h>
#include <iostream>

int main() {
    std::cout << "TBB version: " << TBB_VERSION_MAJOR << "."
              << TBB_VERSION_MINOR << std::endl;
    return 0;
}
```

Compile with:

```
g++ -o tbb_test test.cpp -ltbb
```

If it compiles and runs without errors, TBB is properly installed.