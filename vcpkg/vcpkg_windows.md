# vcpkg project [windows]



#### install vcpkg,cmake ;



shell:

```shell

cmake -S .. -B . -DCMAKE_TOOLCHAIN_FILE=C:\Users\sheng\vcpkg\scripts\buildsystems\vcpkg.cmak

cmake --build . --config Release --target main
```



```cmake
cmake_minimum_required(VERSION 3.16)
project(cpp_vcpkg_project VERSION 0.1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Use vcpkg toolchain when invoking CMake:
# cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=C:/Users/sheng/vcpkg/scripts/buildsystems/vcpkg.cmake
add_executable(main src/main.cpp)
find_package(spdlog CONFIG REQUIRED)
target_link_libraries(main PRIVATE spdlog::spdlog)

```



vcpgk.json

```json
{
  "name": "cpp-vcpkg-project",
  "version-string": "0.1.0",
  "dependencies": [
    "spdlog"
  ]
}

```



```c++
#include <iostream>
#include <spdlog/spdlog.h>
#include <spdlog/sinks/basic_file_sink.h>

int main() {
    spdlog::info("Hello from vcpkg project");
    spdlog::error("This is an error message");
    // Create a file logger
    auto file_logger = spdlog::basic_logger_mt("file_logger", "logs.txt");
    file_logger->info("This message will be logged to a file");
    // Set the log level for the file logger
    file_logger->set_level(spdlog::level::err);
    // Log an error message to the file logger
    file_logger->error("This is an error message logged to a file");
    return 0;   
}

```

