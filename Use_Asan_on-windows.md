# 在 Windows 上使用 AddressSanitizer (ASan)
- 编译器：LLVM的Clang20
### 背景知识(可以跳过)
- Clang 使用 MSVC 工具链：
- Clang 在 Windows 上默认使用 MSVC 工具链（lld-link），而不是   MinGW 工具链。
- MSVC 工具链对 ASan 的支持不如 Linux 或 macOS 完善。
ASan 运行时库：

- Clang 的 ASan 在 Windows 上需要特定的运行时库（如 clang_rt.asan_dynamic.dll 和 clang_rt.asan_dynamic_runtime_thunk.lib）。
如果这些库未正确链接，可能会导致编译或链接失败。
CMake 配置问题：

- 当前的 CMakeLists.txt 配置中，您使用了 MSVC 编译器（cl），但同时启用了 Clang 的 ASan，这可能导致不兼容。
## 快速解决方案
```cpp
cmake_minimum_required(VERSION 3.30)
project(AVLTest)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置编译器
set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER clang++)

# 添加测试文件和主 AVL 实现文件
add_executable(AVLTest example.cpp)
target_include_directories(AVLTest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/..)

# 检测是否支持 AddressSanitizer
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    message(STATUS "Enabling AddressSanitizer (ASan)")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
    # 添加 ASan 运行时库路径
    link_directories("D:/LLVM/clang-msvc20/lib/clang/20/lib/windows") 
    target_link_libraries(AVLTest PRIVATE clang_rt.asan_dynamic-x86_64 clang_rt.asan_dynamic_runtime_thunk-x86_64)
endif()
```
- AVL是我实验的文件，你自己改改就好了
-  注意 # 添加 ASan 运行时库路径
    link_directories("D:/LLVM/clang-msvc20/lib/clang/20/lib/windows") 这个路径是你自己电脑上的库路径
- 同时这个库**asan_dynamic_runtime_thunk-x86_64**动态库拷贝到build目录下。先cmake构建后，再拷贝，然后使用ninja生成可执行文件。
## 以下阅读了解即可
## 1. 确保 Clang 和工具链路径正确
确保 Clang 和其工具链的路径已正确添加到系统环境变量中。运行以下命令检查：

```bash
clang++ --version
```
输出应显示 Clang 的版本和目标平台（如 `x86_64-pc-windows-msvc`）。

## 2. 显式链接 ASan 运行时库
在 `CMakeLists.txt` 中，显式添加 ASan 的运行时库路径和库文件：
如果您已经确认 AddressSanitizer (ASan) 的运行时库（如 clang_rt.asan_dynamic.dll 和 clang_rt.asan_dynamic_runtime_thunk.lib）存在
```cmake
# 示例代码
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
```

## 3. 检查 Ninja 和 CMake 配置
确保您使用的是正确的生成器和工具链。重新生成构建文件时，显式指定生成器和工具链：

```bash
cmake -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Debug/Release ..
```

## 4. 禁用不兼容的选项
如果仍然遇到问题，可以尝试禁用不兼容的选项，例如 `-fuse-ld=ld` 或其他可能导致问题的链接器标志。

```cmake
# 示例代码
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-omit-frame-pointer")
```

---

## 问题分析

1. **`--out-implib`、`--major-image-version` 等选项**：
   - 这些选项是 GCC/MinGW 链接器（`ld`）的选项，而不是 `lld-link` 的选项。
   - `lld-link` 是 LLVM 的链接器，遵循 MSVC 链接器的选项格式。

2. **`libAVLTest.dll.a` 文件**：
   - 这是 GCC/MinGW 的动态库导入文件格式，而 `lld-link` 使用 `.lib` 格式。

3. **`lld-link` 的错误**：
   - `lld-link` 无法找到 `libAVLTest.dll.a`，因为它期望的是 `.lib` 文件。

---


## 额外添加
### 1. 强制使用 MinGW 的 `ld` 链接器
如果您希望使用 GCC/MinGW 的链接器，而不是 `lld-link`，可以通过以下方式配置 CMake：

在 `CMakeLists.txt` 中添加以下内容：
```cmake
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fuse-ld=ld")
```
这会强制 Clang 使用 MinGW 的 `ld` 链接器，而不是 `lld-link`。
---

### 2. 切换到 MSVC 工具链
如果您希望在 Windows 上使用 Clang 和 `lld-link`，建议切换到 MSVC 工具链，并确保链接器选项与 MSVC 格式兼容。

在 `CMakeLists.txt` 中，移除 `-fuse-ld=lld`，并确保没有 GCC/MinGW 特定的选项：
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
```

---
- 如果不成功，看下面(怎能可能呢？哈哈，只是推荐！)
## 建议
### 4. 切换到 Linux 或 WSL
如果您需要完整的 ASan 支持，建议切换到 Linux 或 Windows Subsystem for Linux (WSL)。在这些环境中，Clang 和 GCC 对 ASan 的支持更加完善。

---

## 总结

- **推荐方案**：在 Windows 上使用 Clang 时，强制使用 MinGW 的 `ld` 链接器，或者切换到 MSVC 工具链。
- **替代方案**：禁用 ASan 或切换到 Linux 环境。
