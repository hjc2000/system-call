# system-call

## 说明

基于 [bsp-interface](https://github.com/hjc2000/bsp-interface.git) 库，提供了一些系统调用的实现。

## 使用链接器选项 `--whole-archive` 

在 [cpp-lib-build-scripts](https://github.com/hjc2000/cpp-lib-build-scripts) 中的 target_import_system_call.cmake 文件中，导入本库时使用了 `-Wl,--whole-archive` 以此来避免本库提供的系统调用函数被链接器忽略掉。如下所示：

```cmake
include(target_import_bsp_interface)

function(target_import_system_call target_name visibility)
	set(lib_name "system-call")
	target_include_directories(${target_name} ${visibility} ${libs_path}/${lib_name}/include)

	target_link_libraries(${ProjectName} PUBLIC -Wl,--whole-archive)
	target_auto_link_lib(${target_name} ${lib_name} ${libs_path}/${lib_name}/lib/)
	target_link_libraries(${ProjectName} PUBLIC -Wl,--no-whole-archive)

	target_import_bsp_interface(${target_name} ${visibility})
endfunction()

```

如果不使用 `-Wl,--whole-archive` ，链接器会忽略掉本库提供的函数，然后又会报错说找不到这些函数，说这些函数是未定义的符号。

## 技巧

遇到像这种问题：

> 编译器一直忽略掉静态库中的符号，然后又说找不到，并且使用了 `-Wl,--start-group` 后仍然无效：
>
> ```cmake
> target_link_libraries(${ProjectName} PUBLIC -Wl,--start-group)
> ...
> target_link_libraries(${ProjectName} PUBLIC -Wl,--end-group)
> ```
>
> 

就可以像本库的做法。单独将这部分代码提取出来，链接该库时使用 

```cmake
target_link_libraries(${ProjectName} PUBLIC -Wl,--whole-archive)
...
target_link_libraries(${ProjectName} PUBLIC -Wl,--no-whole-archive)
```

将链接该库的 target_link_libraries 语句插入上面的 2 行中间。