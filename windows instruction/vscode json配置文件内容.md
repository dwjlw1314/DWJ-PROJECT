更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387

<font color=#FF0000 size=4> <p align="center">tasks.json</p></font>

```cmake format tasks.json
{
	"version": "2.0.0",
	"tasks": [
        {
            "type": "shell",
            "label": "CreateBuildDir",
            "command": "mkdir",
						// 传给上面命令的参数，windows下稍有不同
            "args": [
                "-p",
                "build"
            ],
            "windows": {
                "options": {
                    "shell": {
                        "executable": "powershell.exe"
                    }
                },
                "args": [
                    "-Force",
                    "build"
                ]
            },
            "options": {
                "cwd": "${workspaceFolder}/CAAIImagePro/FFmpegDecode/"
            },
            "problemMatcher": [
                "$gcc"
            ]
        },
        {
            "type": "shell",
            "label": "cmakeRun",
            "command": "cmake",
            "args": [
                "../"
            ],
            "options": {
                "cwd": "${workspaceFolder}/CAAIImagePro/FFmpegDecode/build"
            },
            "dependsOn": [
                "CreateBuildDir"
            ],
            "problemMatcher": [],
            "group": "build"
        },
        {
            "type": "shell",
            "label": "makeRun",
            "command": "make",
            "args": [],
            "options": {
                "cwd": "${workspaceFolder}/CAAIImagePro/FFmpegDecode/build"
            },
            "dependsOn": [
                "cmakeRun"
            ],
            "problemMatcher": [],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

<font color=#FF0000 size=4> <p align="center">c_cpp_properties.json</p></font>

```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
								//添加头文件识别目录，使源文件中引用的头文件可见
                "${workspaceFolder}/**",
                "/usr/local/cuda/include",
            ],
						//添加宏定义
            "defines": ["HAVE_GPU"],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu11",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```

<font color=#FF0000 size=4> <p align="center">launch.json</p></font>

```
{
    // 使用 IntelliSense 了解相关属性
    // 悬停以查看现有属性的描述
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gjsy",  //生成和调试活动文件
             // 设定编译器的类型，MinGW64是g++，这里是cppdgb，这个是规定的，不是随便写，比如msvc编译器就是cppvsdbg
            "type": "cppdbg",
            "request": "launch",
            // program 这个是你的可执行程序位置，这里可以根据自己的tasks.json生成
            "program": "${workspaceFolder}\\build\\${workspaceRootFolderName}.exe",
            // ${xxxx}是vscode内置的变量，可以方便获取到需要的路径或者文件名，这里列举一部分
            // ${workspaceFolder} :表示当前workspace文件夹路径，也即/home/Coding/Test
						// ${workspaceRootFolderName}:表示workspace的文件夹名，也即Test
						// ${file}:文件自身的绝对路径，也即/home/Coding/Test/.vscode/tasks.json
						// ${relativeFile}:文件在workspace中的路径，也即.vscode/tasks.json
						// ${fileBasenameNoExtension}:当前文件的文件名，不带后缀，也即tasks
						// ${fileBasename}:当前文件的文件名，tasks.json
						// ${fileDirname}:文件所在的文件夹路径，也即/home/Coding/Test/.vscode
						// ${fileExtname}:当前文件的后缀，也即.json
						// ${lineNumber}:当前文件光标所在的行号
						// ${env:PATH}:系统中的环境变量
            "args": [],
						// 调试是否停留在入口处
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            // 调试器的路径
            "miDebuggerPath": "D:\\program\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            // preLaunchTask 表示在执行调试前要完成的任务
            // 这里的 makeRun 是 tasks.json 中 lable 标记的任务名称
            "preLaunchTask": "makeRun",
        }
    ]
}
```
