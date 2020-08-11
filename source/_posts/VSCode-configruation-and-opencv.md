---
title: VSCodeC++环境搭建并配置OpenCV
date: 2020-08-09 20:42:38
categories: Others
tag:
     - C++
---

Win10 VSCode搭建C++环境，并配置openCV4，记录一下，避免每次都要查好多博客。

<!-- more -->

### 预安装环境

- #### 编译C++


1. VSCode，[下载](https://code.visualstudio.com/)；

2. mingw64，推荐离线版的，[x86_64-posix-seh下载](https://sourceforge.net/projects/mingw-w64/files/Toolchains targetting Win64/Personal Builds/mingw-builds/8.1.0/threads-posix/seh/x86_64-8.1.0-release-posix-seh-rt_v6-rev0.7z)，注意是posix，因为win32版本编译openCV会出问题，gcc版本是8.1.0，把bin加入到环境变量，**gcc --version**验证；

- #### 配置openCV

1. Opencv4，[下载](https://github.com/opencv/opencv/releases)；

2. Cmake，[下载](https://github.com/opencv/opencv/releases)。

### 编译openCV

打开Cmake输入源文件路径和编译路径，在**configure**中选MinGW，按照图2，3配置。

![file](VSCode-configruation-and-opencv\dir_config.png)

<center>图1  Cmake编译路径配置</center>

![file](VSCode-configruation-and-opencv\compiler.png)

<center>图2  选择编译方式</center>

![file](VSCode-configruation-and-opencv\compiler1.png)

<center>图3  编译器路径</center>

**configure**结束后，根据需要勾选掉不需要的，我勾选掉了*ENABLE_PRECONPILED_HEADERS*，*BUILD_opencv_ts* ，*WITH_CUDA*  和*OPENCV_ENABLE_ALLOCATOR_STATS*，然后**generate**。

CD到刚刚build好的目录，执行**minGW32-make**开始编译，编译遇到错误可以回去勾选掉有问题的选项重新**configure**和**generate**，然后接着编译。

### VSCode配置

VSCode安装C/C++扩展，创建一个.vscode文件夹，创建3个文件。

+ **c_cpp_properties.json**

```
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceRoot}",
                "D:/Programs/mingw64/include/**",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include-fixed",
                "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/../../../../x86_64-w64-mingw32/include",
                "D:/Programs/opencv/mingw_build/install/include",
                "D:/Programs/opencv/mingw_build/install/include/opencv2"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "__GNUC__=6",
                "__cdecl=__attribute__((__cdecl__))"
            ],
            "intelliSenseMode": "msvc-x64",
            "browse": {
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": "",
                "path": [
                    "${workspaceRoot}",
                    "D:/Programs/mingw64/include/**",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/include-fixed",
                    "D:/Programs/mingw64/bin/../lib/gcc/x86_64-w64-mingw32/8.1.0/../../../../x86_64-w64-mingw32/include",
                    "D:/Programs/opencv/mingw_build/install/include"
                ]
            }
        }
    ],
    "version": 4
}
```

将目录改为自己mingw对应的路径。

+ **launch.json**

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，这里只能为cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，一般设置为false
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录，一般为${workspaceRoot}即代码所在目录 workspaceRoot已被弃用，现改为workspaceFolder
            "environment": [],
            "externalConsole": true, // 调试时是否显示控制台窗口，一般设置为true显示控制台
            "MIMode": "gdb",
            "miDebuggerPath": "D:/Programs/mingw64/bin/gdb.exe", // miDebugger的路径，注意这里要与MinGw的路径对应
            "preLaunchTask": "g++", // 调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}

```

注意**miDebuggerPath**的路径。

+ **tasks.json**

```
{
    "version": "2.0.0",
    "command": "g++",
    "args": [
        "-g",
        "${file}",
        "-o",
        "${fileBasenameNoExtension}.exe",
        "-I", "D:/Programs/opencv/mingw_build/install/include",
        "-I", "D:/Programs/opencv/mingw_build/install/include/opencv2",
        "-I", "D:/Programs/opencv/mingw_build/install/x64/mingw/bin",
        "-L", "D:/Programs/opencv/mingw_build/install/x64/mingw/lib",
        "-l", "opencv_calib3d430",
        "-l", "opencv_core430",
        "-l", "opencv_dnn430",
        "-l", "opencv_features2d430",
        "-l", "opencv_flann430",
        "-l", "opencv_highgui430",
        "-l", "opencv_imgcodecs430",
        "-l", "opencv_imgproc430",
        "-l", "opencv_ml430",
        "-l", "opencv_objdetect430",
        "-l", "opencv_photo430",
        "-l", "opencv_stitching430",
        "-l", "opencv_video430",
        "-l", "opencv_videoio430",
    ],
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": [
            "relative",
            "${workspaceFolder}"
        ],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    },
    "group": {
        "kind": "build",
        "isDefault": true
    }
}
```

重点在这里，首先要修改对应路径，**-l(i大写)"代表头文件位置,-L代表库文件位置，-l(l小写)代表库文件，不用写后缀。**注意不要弄L和I！！！到这里基本就可以用了。

### 参考

1. VScode搭建OpenCV环境，[链接](https://www.cnblogs.com/kensporger/p/12320622.html)
2. Windows10下Opencv4+CMake+MinGW64+VSC安装教程，[链接](https://www.cnblogs.com/uestc-mm/p/12758110.html)
3. vscode windows10 opencv c++ 环境配置，[链接](https://blog.csdn.net/dss875914213/article/details/103338355)
4. OpenCV4.3.0 编译踩坑填坑，[链接](https://www.jianshu.com/p/629d5ae6e18c)
5. Win10下Qt+OpenCV+Cmake编译错误记录与解决【gcc: error: long: No such file or directory】，[链接](https://blog.csdn.net/nohopenolove/article/details/106411477)