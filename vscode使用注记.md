# vscode使用注记

## 安装vscode

后续重新安装时补上

## 安装MinGW编译器

后续重新安装时补上

## 配置.vscode文件

在桌面或者哪个位置新建文件夹和里面新建一个cpp文件（强烈建议路径和文件名全是英文，避免出现一些奇奇怪怪的问题，原来的版本有中文是不行的，新版本我没有试过，小白不建议尝试），我是在桌面建了一个test文件夹，然后在里面建立了一个main.cpp的文件。

注意，是用VScode打开文件夹，打开一个单独的cpp是不行的！！也就是说，这个cpp要在一个文件夹里！不像Dev-Cpp和C-Free可以直接调试一个单独的cpp！！

编辑main.cpp，写一个hello world。点击左侧倒数第二个小虫子的图标->**create a launch.json file**->**C++ (GDB/LLDB)**->**g++.exe -build and debug active file**，然后会在工作目录下生成一个launch.json的启动配置文件

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++.exe - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\MinGW\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++.exe build active file"
        }
    ]
}
```

launch.json中有很多属性可以设置, 通过智能提示查看有那些属性可以设置, 如果要查看属性的具体含义, 可以把鼠标悬停在属性上面, 会属性的使用说明。

**"externalConsole"**属性改为true，即可显示控制台黑框，需要在程序结束前中断，否则自动退出。

**preLaunchTask**：调试会话开始前执行的任务，一般为编译程序，c++为g++, c为gcc ，技术贴中指出这里应改为c++，直接用默认"C/C++: g++.exe build active file"，可以不用再配置task.json

**miDebuggerPath**要与MinGw的路径对应，注意miDebuggerPath是\\\\，与直接粘贴过来的路径不一样，需要加一个”\”（冒号后面的两个反斜杠“\\”）,或者使用一个“/”。保存切换到main.cpp并按F5进行调试。

此时应该是要弹框提示**configure task**，此时会在工作目录中生成task.json（让人意外的是，第二次测试时没有弹框提示，直接成功运行了。）

task.json文件中lable属性的值应改为与launch.json文件中preLaunchTask属性的值相同，在第二次测试时launch.json文件中preLaunchTask的属性采用默认值"C/C++: g++.exe build active file"，就无需再改动默认的task.json文件。

## 各种问题

1. **include文件找不到**
   + 配置c_cpp_properties.json，修改includePath，添加目标目录。
   + task.json->args中添加"-I",C:\\\\Users\\\\PC\\\\Desktop\\\\\*\*\\\\\*\*\\\\\*\*\\\\Include"。
   
2. **undefined reference to**

   需要的库文件没有链接成功

   + #pragma comment(lib, "\*\*.lib");
   + task.json->args中添加"-l\*\*"

##  参考文献

https://blog.csdn.net/bat67/article/details/76095813

