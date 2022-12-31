---
title: CMake编译系统
date: 2022-12-30 12:31:52
tags: Essay
---

最近在写Cuda，需要用C++，记录下用CMake编译运行的方法。

<!--more-->

# 正文

环境gcc7.5.0，ubuntu18.04，cmake3.25.0。

![](/img/Cmake编译系统/1.png)

整体结构如上图所示，u1文件夹里是写好的算法代码，eval文件夹里是基于算法编写的应用程序，包括测试、实验对比、调用程序等。
因为用到protobuf，所以需要先对proto文件处理生成对应\*.pb.cc和\*.pb.h文件，这两个文件内容比较多，先把其编译成独立的库，而不是直接和u1里的代码混编，第二步编译u1，最后编译eval里的应用程序。

有几个点注意：第一头文件和执行文件放在一起，所以不设置单独include文件夹，第二cmake版本小于3.10需要手动引入cuda库。

因此CMakeLists.txt的书写方法如下：

```CMake
cmake_minimum_required(VERSION 3.10)
project(u1_test CXX C CUDA)

```

先声明项目需要用到语言，因为是GCC7.5.0，默认C++11标准。

```CMake
include_directories("${PROJECT_SOURCE_DIR}/src")

aux_source_directory("${PROJECT_SOURCE_DIR}/src/u1" u1)
```

声明头文件额外引入路径和需要编译的库目录

```CMake
find_package(gflags REQUIRED)
find_package(glog REQUIRED)
find_package(protobuf REQUIRED)
```

声明需要用到的包，glog，gflags和protobuf是我做C++开发比较喜欢用的库。

```CMake
# protoc -I=. --cpp_out=. kernel.proto
set(proto_file_name "kernel")
get_filename_component(u1_proto "${PROJECT_SOURCE_DIR}/src/u1/proto/${proto_file_name}.proto" ABSOLUTE)
get_filename_component(u1_proto_path "${u1_proto}" PATH)
set(PROTO_GENERATE_DIR "${PROJECT_SOURCE_DIR}/src/u1/proto" )
set(u1_proto_srcs "${PROTO_GENERATE_DIR}/${proto_file_name}.pb.cc")
set(u1_proto_hdrs "${PROTO_GENERATE_DIR}/${proto_file_name}.pb.h")
add_custom_command(
  OUTPUT "${u1_proto_srcs}" "${u1_proto_hdrs}"
  COMMAND protoc
  ARGS 
    --cpp_out "${PROTO_GENERATE_DIR}" 
    -I "${u1_proto_path}"
    "${u1_proto}"
  DEPENDS "${u1_proto}"
)
```

生成kernel.proto对应的c++ API文件

```CMake
add_library(
  proto_ttt
  ${u1_proto_srcs}
  ${u1_proto_hdrs}
)

target_link_libraries(
  proto_ttt
  protobuf::libprotobuf
)

add_library(
  u1_lib
  ${u1}
)
```

然后分别生成库文件，包括定义接口的库以及算法库。

```CMake
function(add_executable_app app_name app_path)
  add_executable(
    ${app_name}
    ${app_path}
  )
  target_link_libraries(
    ${app_name}
    u1_lib
    gflags
    glog
    protobuf::libprotobuf
  )
endfunction ()

add_executable_app(test "${PROJECT_SOURCE_DIR}/src/eval/test.cu")
add_executable_app(main "${PROJECT_SOURCE_DIR}/src/eval/main.cu")
```

最后生成应用程序、测试程序和用来比较的实验数据。





