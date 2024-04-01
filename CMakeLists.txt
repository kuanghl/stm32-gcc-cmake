cmake_minimum_required(VERSION 3.16) # 判定 cmake 版本符合要求，stm32-cmake 模块本身要求至少 3.16

set(CMAKE_BUILD_TYPE "Debug")   # 设置编译模式

set(STM32_TOOLCHAIN_PATH ${CMAKE_CURRENT_SOURCE_DIR}/tool_chain/xpm_packets/xpacks/.bin)    # 设置编译器路径
set(STM32_TARGET_TRIPLET arm-none-eabi)                                                     # 设置编译器名称            
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