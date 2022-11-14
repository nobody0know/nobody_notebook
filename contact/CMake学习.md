# CMke学习

## 静态库与动态库的使用与区别

### 构建和链接静态/动态库

```cmake
# set minimum cmake version 
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name and language
project(recipe-03 LANGUAGES CXX)

# generate a library from sources
add_library(message
  STATIC
    Message.hpp
    Message.cpp
  )
#创建hello-world的可执行目标
add_executable(hello-world hello-world.cpp)
#将目标库链接到可执行目标
target_link_libraries(hello-world message)


### 
```

- `add_library(message STATIC Message.hpp Message.cpp)`：生成必要的构建指令，将指定的源码编译到库中。`add_library`的第一个参数是目标名。整个`CMakeLists.txt`中，可使用相同的名称来引用库。生成的库的实际名称将由CMake通过在前面添加前缀`lib`和适当的扩展名作为后缀来形成。生成库是根据第二个参数(`STATIC`或`SHARED`)和操作系统确定的。
- `target_link_libraries(hello-world message)`: 将库链接到可执行文件。此命令还确保`hello-world`可执行文件可以正确地依赖于消息库。因此，在消息库链接到`hello-world`可执行文件之前，需要完成消息库的构建。

编译成功后，构建目录包含`libmessage.a`一个静态库(在GNU/Linux上)和`hello-world`可执行文件。

CMake接受其他值作为`add_library`的第二个参数的有效值，我们来看下本书会用到的值：

- **STATIC**：用于创建静态库，即编译文件的打包存档，以便在链接其他目标时使用，例如：可执行文件。
- **SHARED**：用于创建动态库，即可以动态链接，并在运行时加载的库。可以在`CMakeLists.txt`中使用`add_library(message SHARED Message.hpp Message.cpp)`从静态库切换到动态共享对象(DSO)。
- **OBJECT**：可将给定`add_library`的列表中的源码编译到目标文件，不将它们归档到静态库中，也不能将它们链接到共享对象中。如果需要一次性创建静态库和动态库，那么使用对象库尤其有用。我们将在本示例中演示。
- **MODULE**：又为DSO组。与`SHARED`库不同，它们不链接到项目中的任何目标，不过可以进行动态加载。该参数可以用于构建运行时插件。

### 静态/动态库的区别

静态库和动态库的载入时间是不一样的。

静态库的代码在编译的过程中已经载入到可执行文件中，所以最后生成的可执行文件相对较大。

动态库的代码在可执行程序运行时才载入内存，在编译过程中仅简单的引用，所以最后生成的可执行文件相对较小。

静态库和动态库的最大区别是，静态库链接的时候把库直接加载到程序中,而动态库链接的时候，它只是保留接口，将动态库与程序代码独立，这样就可以提高代码的可复用度和降低程序的耦合度。

静态库在程序编译时会被连接到目标代码中，程序运行时将不再需要该静态库。

动态库在程序编译时并不会被连接到目标代码中，而是在程序运行是才被载入，因此在程序运行时还需要动态库存在。

无论静态库，还是动态库，都是由.o文件创建的。因此，我们必须将源程序hello.c通过gcc先编译成.o文件。 

## 为语言设定标准

```cmake
#构建 animals 库
add_library(animals
  SHARED
    Animal.cpp
    Animal.hpp
    Cat.cpp
    Cat.hpp
    Dog.cpp
    Dog.hpp
    Factory.hpp
  )
#为 animals 库设定C++14标准
set_target_properties(animals
  PROPERTIES
    CXX_STANDARD 14
    CXX_EXTENSIONS OFF
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE 1
  )
```

## 环境检测

### 操作系统检测

```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name, in this case no language required
project(recipe-01 LANGUAGES NONE)

# print custom message depending on the operating system
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  message(STATUS "Configuring on/for Linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  message(STATUS "Configuring on/for macOS")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  message(STATUS "Configuring on/for Windows")
else()
  message(STATUS "Configuring on/for ${CMAKE_SYSTEM_NAME}")
endif()
```
