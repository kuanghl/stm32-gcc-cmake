# stm32-gcc-cmake

ubuntu上使用gcc + cmake + gcc_arm_none_eabi编译STM32项目的示例项目。

可以使用LL HAL库。

## 依赖

```sh
sudo apt install gcc-arm-none-eabi

# repo
https://github.com/ObKo/stm32-cmake.git
https://github.com/STMicroelectronics/STM32CubeF1.git

# 使用版本分支commit id
STM32CubeF1 : c750eab6990cac35ab05020793b0221ecc1a8ce5
stm32-cmake : 6a0d25571178c89b75c46c831e54fe279600353d

# !!! STM32CubeF1 STM32CubeFxx后续版本不支持stm32-cmake
```

## 使用说明

### 构建
```sh
# case 1 直接使用
git clone git@github.com:kuanghl/stm32-gcc-cmake.git
cd stm32-gcc-cmake
make

# case 2 自构建
mkdir stm32-demo && cd stm32-demo
git clone https://github.com/ObKo/stm32-cmake.git
cd stm32-cmake && git checkout -b old 6a0d25571178c89b75c46c831e54fe279600353d
git clone https://github.com/STMicroelectronics/STM32CubeF1.git
cd STM32CubeF1 && git checkout -b old c750eab6990cac35ab05020793b0221ecc1a8ce5
# todo... 编写CMakeLists.txt Makefile
mkdir build && cd build && cmake .. && make -j8
```

### 烧录

```sh
# todo...
```


## Overview

### 目录结构

本节介绍示例项目目录结构。

```raw
.                   # 根目录
├── .clang-format   # 配合编辑器或集成开发环境自动格式化 c/c++ 源文件
├── .git            # 表示为项目目录创建了一个 git 仓库
│   └── ...         #
├── .gitignore      # 指定 git 仓库的排除路径，主要是 `build/`
├── CMakeLists.txt  # 指定项目编译方式
├── src             # 源文件放在这个目录下
│   ├── library.h   #
│   ├── library.c   #
│   └── main.c      #
├── Makefile        # 简化编译指令
├── README.md       # 项目说明文件
├── STM32CubeF1     # 子模块。提供 cmake 模板，供 CMakeLists.txt 使用
│   └── ...         #
├── build           # 构建目录，从 git 排除
│   └── ...         #
└── stm32-cmake     # 子模块（[产品页](https://www.st.com/zh/embedded-software/stm32cubef1.html)）。STM32F1 产品系列的软件包，在 CMakeLists.txt 中引用
    └── ...         #
```

### 配置文件示例

本节介绍示例项目配置文件。按文件名字典顺序排序。

- [`.clang-format`](.clang-format)

  为编辑器或集成开发环境提供 c/c++ 源文件格式化规则。此项目包含在项目中有助于项目编码风格的一致性。

  本文件通过 clion 生成。详细配置项参见 [.clang-format 文档](https://clang.llvm.org/docs/ClangFormat.html)。

  示例：略。

- [`.gitignore`](.gitignore)

  示例：

  ```.gitignore
  build/          # cmake 常用的命令行构建路径
  cmake-build-**/ # clion 默认的 cmake 构建路径

  .*/             # 常见 ide/editor 创建的配置文件路径，排除这类路径使项目对开发工具中立
  ```

- [`CMakeLists.txt`](CMakeLists.txt)

  cmake 项目配置。

  示例和说明：

```c
cmake_minimum_required(VERSION 3.16) # 判定 cmake 版本符合要求，stm32-cmake 模块本身要求至少 3.16

set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/stm32-cmake/cmake/stm32_gcc.cmake)  # 引用 stm32-cmake

project(stm32_test CXX C ASM)        # 设定项目名字和支持语言
set(CMAKE_CXX_STANDARD 20)           # 设定语法级别
set(CMAKE_CXX_STANDARD_REQUIRED ON)  # C++标准包
set(CMAKE_INCLUDE_CURRENT_DIR TRUE)  # 引用根目录

set(STM32_CUBE_F1_PATH ${CMAKE_CURRENT_SOURCE_DIR}/STM32CubeF1) # 查找 STMCubeF1 库
find_package(CMSIS COMPONENTS STM32F1 REQUIRED)                 # 逐设备引用 CMSIS
find_package(HAL COMPONENTS STM32F1 RCC GPIO CORTEX LL_RCC LL_GPIO REQUIRED) # 逐模块引用 HAL


include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)                       # 暴露stm32f1xx_hal_conf.h
add_executable(stm32_test_app ${CMAKE_CURRENT_SOURCE_DIR}/src/main.c)      # 一个构建目标,默认生成.elf 

target_link_libraries(stm32_test_app # 将 CMSIS、LL、HAL 链接到库
    HAL::STM32::F1::RCC          # HAL库
    HAL::STM32::F1::GPIO         # HAL库
    HAL::STM32::F1::CORTEX       # HAL库   
    HAL::STM32::F1::LL_RCC       # LL库
    HAL::STM32::F1::LL_GPIO      # LL库
    CMSIS::STM32::F103C8         # 对于同时适用多种芯片的库，不用在这里链接芯片相关库
    STM32::NoSys)                #

add_library(stm32_test STATIC                                # 构建一个静态库。静态库只需要构建一次，可以在多个目标之间复用
    ${CMAKE_CURRENT_SOURCE_DIR}/src/library.h                # 库的头文件，头文件不需要编译，但写在这里 VSCode + CMake 插件才会为其提供正确的高亮和跳转
    ${CMAKE_CURRENT_SOURCE_DIR}/src/library.c)               # 库的源文件，支持 c 

target_link_libraries(stm32_test_app stm32_test) # 链接库

stm32_print_size_of_target(stm32_test_app)       # 打印目标文件容量
stm32_generate_binary_file(stm32_test_app)  # 生成.bin
stm32_generate_hex_file(stm32_test_app)     # 生成.hex
stm32_generate_srec_file(stm32_test_app)    # 生成.srec
```

- [`Makefile`](Makefile)

  简化使用 cmake 构建项目的步骤。

  > **注意**：不要从 markdown 文件中拷贝 Makefile。Makefile 文件要求脚本指令前使用 `tab`，markdown 中为规范格式已替换为空格。

  示例和说明：

  ```Makefile
  .PHONY : build # 按 release 构建
  build:
      mkdir -p build/release
      cd build/release \
      && cmake -DCMAKE_BUILD_TYPE=Release \
         ../.. \
      && make -j2

  .PHONY : build-debug # 按 debug 构建
  debug:
      mkdir -p build/debug
      cd build/debug \
      && cmake -DCMAKE_BUILD_TYPE=Debug \
         ../.. \
      && make -j2

  .PHONY : clean # 删除构建目录
  clean:
      rm -rf build
  ```
