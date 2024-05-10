---
{"dg-publish":true,"permalink":"/05-online-courses/c03-parallel-101/parallel-101-series-01-c-make/","noteIcon":"","created":"2024-03-23T22:12:44.912+01:00","updated":"2024-03-24T10:43:41.330+01:00"}
---

## CMake引入以及子模块
```CMake
cmake_minimum_required(VERSION 3.21)
project(hellocmake LANGUAGES CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_subdirectory(hellolib)

add_executable(hello_cmake main.cpp)

target_link_libraries(hello_cmake PUBLIC hellolib)

# Post build step: copy the executable to the root directory
add_custom_command(TARGET hello_cmake
                   POST_BUILD
                   COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:hello_cmake> ${CMAKE_SOURCE_DIR})

# Post build step: create a symlink to the compile_commands.json
file(CREATE_LINK
    "${CMAKE_BINARY_DIR}/compile_commands.json"
    "${CMAKE_SOURCE_DIR}/compile_commands.json"
    SYMBOLIC
)
```

```CMake
add_library(hellolib STATIC hello.cpp)
target_include_directories(hellolib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}) 
```
- 子模块的头文件处理
	- 方法一
		- 给使用子模块的上一层模块，添加子模块头文件的搜索目录
			- `target_include_directories(a.out PUBLIC hellolib)`
			- **通过`target_include_directories`添加的路径会被视为与系统路径等价**
	- 方法二
		- 在子模块中定义头文件搜索路径，引用它的可执行文件**CMake必须自动添加这个路径**
			- `target_include_directories(hellolib PUBLIC .)`
				- 子目录中的路径是**相对路径**
			- `target_include_directories(hellolib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})`
				- 使用`PRIVATE`则不会自动添加这一路径
				- `PUBLIC/PRIVATE`: **决定一个属性在link阶段是否传播**
- 其它常用CMake选项
```CMake
target_include_directories(myapp PUBLIC /usr/include/eigen3)  # 添加头文件搜索目录
target_link_libraries(myapp PUBLIC hellolib)                               # 添加要链接的库
target_add_definitions(myapp PUBLIC MY_MACRO=1)             # 添加一个宏定义
target_add_definitions(myapp PUBLIC -DMY_MACRO=1)         # 与 MY_MACRO=1 等价
target_compile_options(myapp PUBLIC -fopenmp)                     # 添加编译器命令行选项
target_sources(myapp PUBLIC hello.cpp other.cpp)                    # 添加要编译的源文件
```

## 第三方库
### 纯头文件引入
- 使用方法
	- 下载source code (include目录或者头文件)
	- `include_directories(spdlog/include)`
- 缺点
	- 需要重复编译
- example
	- `glm`: https://github.com/liquidhiter/learn_cmake_by_actions/tree/main/1_Examples/3rd_party_lib
	- highlights
		- how to add submodule
			- `git submodule add <repo_url> <target_folder>`
		- how to initialize a submodule
			- `git submodule init && git submodule update`
		- how to check whether submodule has been initialized or not
			- `if git submodule status | grep --quiet "^-";`
### 子模块引入
- 使用方法
	- 复制library的源码到工程的根目录
	- `add_subdirectory(library_path)`
- example
	- `fmt`: https://github.com/liquidhiter/learn_cmake_by_actions/tree/main/1_Examples/3rd_party_lib_module
	- highlights
		- pre-processor `#include <>` search path
			- the subfolder is not included, thus, the header file needs to be included with the sub-folder, 
				- see: https://github.com/liquidhiter/learn_cmake_by_actions/blob/main/1_Examples/3rd_party_lib_module/main.cpp#L3
				- see: https://github.com/fmtlib/fmt/blob/5d63e87d235b86341f8d87f79ed8eb9825f34c44/CMakeLists.txt#L309
					- `#include <fmt/core.h>`
- **可能存在菱形依赖问题**
	- 应用依赖于A，B，而B又依赖于A

### 系统中预安装的第三方库
- 使用方法
	- `find_package()`
	- `target_link_libraries()`
- 具体步骤![Z - assets/images/Pasted image 20240324101511.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240324101511.png)
	- highlights
		- package包含多个components
		- package特有的namespace
		- component使用时必须添加对应的package namespace
		- 可以添加多个组件
- example: [learn_cmake_by_actions/1_Examples/3rd_party_lib_system at main · liquidhiter/learn_cmake_by_actions (github.com)](https://github.com/liquidhiter/learn_cmake_by_actions/tree/main/1_Examples/3rd_party_lib_system)
- 不同package之间的依赖关系
	- package管理者负责进行配置，确保dependencies正确处理 ![Z - assets/images/Pasted image 20240324103349.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240324103349.png)
- 没有菱形依赖问题
	- CMake会先存缓存中搜索相应的package