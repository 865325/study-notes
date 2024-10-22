#### cmake

##### 只有一个源文件的程序编译

```c++
/* hello.cpp */
#include <iostream>
using namespace std;
 
int main()
{
    cout << "Hello 今天是2024/10/22" << endl;
    return 0;
}
```

```cmake
# 文件名 CMakeLists.txt，构建方法 cmake && make
# cmake 最低版本要求
cmake_minimum_required(VERSION 2.8)

# 本项目的工程名
project(cmake_test)

# 第一个参数为生成的可执行文件为hello，后面的参数为需要的依赖
add_executable(hello hello.cpp)
```

##### 同一个项目下多个源文件

```c++
/* hello1.h */
#ifndef __HELLO1_H__
#define __HELLO1_H__

#include <iostream>

void hello1();

#endif
```

```c++
/* hello1.cpp */
#include "hello1.h"

void hello1()
{
	std::cout << "hello 1" << std::endl;
}
```

```cmake
cmake_minimum_required(VERSION 2.8)
project(cmake_test)

# 第一个参数为生成的可执行文件为hello，后面的参数为需要的依赖
add_executable(hello hello.cpp hello1.cpp)
```

##### 同一目录下很多源文件

```cmake
cmake_minimum_required(VERSION 2.8)
project(cmake_test)

# 非递归模式，将指定目录下，具备特定扩展名的文件名存入SRC_FILES变量中
# 指定扩展名，.c, .cpp, .cxx等等，不包括头文件
aux_source_directory(. SRC_FILES)
message("SRC_FILES: ${SRC_FILES}")

# 第一个参数为生成的可执行文件为hello，后面的参数为需要的依赖
add_executable(hello ${SRC_FILES})
```

##### 头文件在其他文件夹

```cmake
# 文件架构
# ├── CMakeLists.txt
# ├── hello1.cpp
# ├── hello.cpp
# └── include
#     └── hello1.h

cmake_minimum_required(VERSION 2.8)
project(cmake_test)

# 指定编译器在编译源文件时，去哪个目录查找头文件
include_directories(./include)

# 非递归模式，将指定目录下，具备特定扩展名的文件名存入SRC_FILES变量中
# 指定扩展名，.c, .cpp, .cxx等等，不包括头文件
aux_source_directory(. SRC_FILES)
message("SRC_FILES: ${SRC_FILES}")

add_executable(hello ${SRC_FILES})
```

##### 头文件和源文件分离，且包含多个文件夹

```cmake
# ├── CMakeLists.txt
# ├── include1
# │   └── hello1.h
# ├── include2
# │   └── hello2.h
# ├── src
# │   └── hello.cpp
# ├── src1
# │   └── hello1.cpp
# └── src2
#     └── hello2.cpp

cmake_minimum_required(VERSION 2.8)
project(cmake_test)

# 指定编译器在编译源文件时，去哪个目录查找头文件
include_directories(./include1 ./include2)

# 非递归模式，将指定目录下，具备特定扩展名的文件名存入SRC_FILES变量中
# 指定扩展名，.c, .cpp, .cxx等等，不包括头文件
aux_source_directory(./src1 SRC_FILES1)
aux_source_directory(./src2 SRC_FILES2)
aux_source_directory(./src MAIN_FILES)

add_executable(hello ${SRC_FILES1} ${SRC_FILES2} ${MAIN_FILES})
```

##### 生成动态库和静态库

```cmake
# ├── build
# │   └── CMakeLists.txt
# ├── include
# │   └── hello1.h
# ├── lib
# └── src
#     └── hello1.cpp

cmake_minimum_required(VERSION 2.8)
project(cmake_test)

# PROJECT_BINARY_DIR 是一个 CMake 变量，包含项目的主构建目录的绝对路径
# 构建目录: 如果你在项目的根目录下创建一个单独的构建目录（如 build），并在该目录中运行 CMake，那么 PROJECT_BINARY_DIR 将指向这个 build 目录
# 引入头文件路径
include_directories(${PROJECT_BINARY_DIR}/../include)

# 整合源文件
aux_source_directory(${PROJECT_BINARY_DIR}/../src SRC_LIST)

# 生成静态库或动态库
# 参数1：生成的库的名称
# 参数2：静态或动态 STATIC SHARED
# 参数3：生成需要的源文件
add_library(file_static STATIC ${SRC_LIST})
add_library(file_shared SHARED ${SRC_LIST})

# 设置最终生成的库名称，这里将库名统一命名为 myfunc
set_target_properties(file_static PROPERTIES OUTPUT_NAME "myfunc")
set_target_properties(file_shared PROPERTIES OUTPUT_NAME "myfunc")

# 设置库生成位置
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../lib)

# 运行结果
# ├── build
# │   └── CMakeLists.txt
# ├── CMakeLists.txt
# ├── include
# │   └── hello1.h
# ├── lib
# │   ├── libmyfunc.a
# │   └── libmyfunc.so
# ├── main_src
# │   └── hello.cpp
# └── src
#     └── hello1.cpp
```

##### 链接库文件

```cmake
# ├── build
# │   └── CMakeLists.txt
# ├── CMakeLists.txt
# ├── include
# │   └── hello1.h
# ├── lib
# │   ├── libmyfunc.a
# │   └── libmyfunc.so
# ├── main_src
# │   └── hello.cpp
# └── src
#     └── hello1.cpp

cmake_minimum_required(VERSION 2.8)
project(cmake_test)

include_directories(${PROJECT_BINARY_DIR}/../include)

aux_source_directory(${PROJECT_BINARY_DIR}/../main_src MAIN_SRC)

# 查找库文件
# 参数1：变量，存储找到的库文件
# 参数2：要查找的库文件
# 参数3：库文件存放的目录
find_library(FUNC_LIB myfunc ${PROJECT_BINARY_DIR}/../lib)

# 设置可执行生成位置
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../bin)

# 生成可执行文件
add_executable(hello ${MAIN_SRC})

# 将库链接到可执行文件
target_link_libraries(hello ${FUNC_LIB})
```

##### 添加编译选项

```cmake
add_compile_options(-std=c++11 -Werror)
```

