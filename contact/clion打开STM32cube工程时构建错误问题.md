# clion打开STM32cube工程时构建错误问题

使用nijia构建此项目时若报错

> FAILED:CMakeFiles/2023_hero_cube.elf.dir/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_C

只需在clion自动生成的cmake文件中打开硬件浮点即可

```cmake
add_compile_definitions(ARM_MATH_CM4;ARM_MATH_MATRIX_CHECK;ARM_MATH_ROUNDING)
add_compile_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
add_link_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
//CMakeList文件中去除此段的注释
```


