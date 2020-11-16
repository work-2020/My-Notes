# Linux下写Makefile笔记

## 概述

详细帖链接：[makefile详细教程][https://www.cnblogs.com/mfryf/p/3305778.html]

Linux下GNU的make是一个命令工具，是一个解释makefile中指令的命令工具。makefile关系到了整个工程的编译规则。一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个文件夹中，makefile定义了一系列的规则来指定，哪些文件须要先编译，哪些文件须要后编译，哪些文件须要又一次编译，甚至于进行更复杂的功能操作，由于makefile就像一个Shell脚本一样，当中也能够运行操作系统的命令。

关于C程序编译的链接，首先要把源文件编译成中间代码文件，在Windows下也就是  .obj 文件，UNIX下是 .o 文件，即 Object File，这个动作叫做编译（compile）。然后再把大量的Object  File合成运行文件，这个动作叫作链接（link）。

编译时，编译器须要的是语法的正确，函数与变量的声明的正确。链接时，主要是链接函数和全局变量，所以，我们能够使用这些中间目标文件（O文件或是OBJ文件）来链接我们的应用程序。

## Makefile介绍

make命令运行时，须要一个 Makefile 文件，以告诉make命令须要怎么样的去编译和链接程序。规则是：
1）假设这个工程沒有编译过，那么我们的全部C文件都要编译并被链接。
2）假设这个工程的某几个C文件被改动，那么我们仅仅编译被改动的C文件，并链接目标程序。
3）假设这个工程的头文件被改变了，那么我们须要编译引用了这几个头文件的C文件，并链接目标程序。

## Makefile规则

target ... : prerequisites ...
			command
			...
			...

target也就是一个目标文件，能够是Object File，也能够是运行文件。还能够是一个标签（Label），对于标签这样的特性，在后续的“伪目标”章节中会有叙述。

prerequisites就是，要生成那个target所须要的文件或是目标。

command也就是make须要运行的命令。（随意的Shell命令，Makefile中的任何命令之前都必须要有一个tab缩进，否则make就会报错。）

这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说，prerequisites中假设有一个以上的文件比target文件要新的话，command所定义的命令就会被运行。这就是Makefile的规则。也就是Makefile中最核心的内容。

## 一个例子

假设一个工程有3个头文件，和8个C文件，我们为了完成前面所述的那三个规则，我们的Makefile应该是以下的这个样子的。

```makefile
edit : main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o
	cc -o edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o

main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o insert.o search.o files.o utils.o
```

反斜杠（/）是换行符的意思。这样比較便于Makefile的易读。我们能够把这个内容保存在文件为“Makefile”或“makefile”的文件里，然后在该文件夹下直接输入命令“make”就能够生成运行文件edit。假设要删除运行文件和全部的中间目标文件，那么，仅仅要简单地运行一下“make clean”就能够了。

在这个makefile中，目标文件（target）包括：运行文件edit和中间目标文件（*.o），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是运行文件 edit  的依赖文件。依赖关系的实质上就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的那一行定义了怎样生成目标文件的操作系统命令，一定要以一个Tab键作为开头。记住，make并无论命令是怎么工作的，他仅仅管运行所定义的命令。make会比較targets文件和prerequisites文件的改动日期，假设prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会运行后续定义的命令。

这里要说明一点的是，clean不是一个文件，它仅仅只是是一个动作名字，有点像C语言中的lable一样，其冒号后什么也沒有，那么，make就不会自己主动去找文件的依赖性，也就不会自己主动运行其后所定义的命令。要运行其后的命令，就要在make命令后明显得指出这个lable的名字。这样的方法非常实用，我们能够在一个makefile中定义不用的编译或是和编译无关的命令，比方程序的打包，程序的备份，等等。

## Makefile中使用变量

在上面的样例中，先让我们看看edit的规则：

```makefile
edit : main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o
	cc -o edit main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o
```

我们能够看到[.o]文件的字符串被反复了两次，假设我们的工程须要添加一个新的[.o]文件，那么我们须要在两个地方加（应该是三个地方，另一个地方在clean中）。当然，我们的makefile并不复杂，所以在两个地方加也不累，但假设makefile变得复杂，那么我们就有可能会忘掉一个须要添加的地方，而导致编译失败。所以，为了makefile的易维护，在makefile中我们能够使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏可能会更好。

比方，我们声明一个变量，叫objects, OBJECTS, objs, OBJS, obj, 或是 OBJ，反正无论什么啦，仅仅要能够表示obj文件就可以了。我们在makefile一开始就这样定义：

objects = main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

于是，我们就能够非常方便地在我们的makefile中以“$(objects)”的方式来使用这个变量了，于是我们的改良版makefile就变成以下这个样子：

```makefile
objects = main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

edit : $(objects)
	cc -o edit $(objects)
main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit $(objects)
```

于是假设有新的 .o 文件添加，我们仅仅需简单地改动一下 objects 变量就能够了。

## 让make自己主动推导

GNU的make非常强大，它能够自己主动推导文件以及文件依赖关系后面的命令，于是我们就不是必需去在每一个[.o]文件后都写上相似的命令，由于，我们的make会自己主动识别，并自己推导命令。

仅仅要make看到一个[.o]文件，它就会自己主动的把[.c]文件加在依赖关系中，假设make找到一个whatever.o，那么whatever.c，就会是whatever.o的依赖文件。而且 cc -c whatever.c 也会被推导出来，于是，我们的makefile再也不用写得这么复杂。我们的是新的makefile又出炉了。

```makefile
objects = main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

edit : $(objects)
	cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
	rm edit $(objects)
```

这个方法，也就是make的“隐晦规则”。上面文件内容中，“.PHONY”表示，clean是个伪目标文件。

## 清空目标文件的规则

每一个Makefile中都应该写一个清空目标文件（.o和运行文件）的规则，这不仅便于重编译，也非常利于保持文件的清洁。这是一个“修养”（呵呵，还记得我的《编程修养》吗）。一般的风格都是：

```makefile
clean:
	rm edit $(objects)
```

更为稳健的做法是：

```makefile
.PHONY : clean
clean :
	-rm edit $(objects)
```

前面说过，.PHONY意思表示clean是一个“伪目标”，。而在rm命令前面加了一个小减号的意思就是，或许某些文件出现故障，但不要管，继续做后面的事。当然，clean的规则不要放在文件的开头，不然，这就会变成make的默认目标，相信谁也不愿意这样。不成文的规矩是——“clean从来都是放在文件的最后”。

## Makefile中的函数

### **加前缀函数**
`$(addprefix <prefix>,<names...>)`
功能: 把前缀 \<prefix\> 加到 \<names\> 中的每个单词前面，单词中间用空格分隔。
返回: 加过前缀的文件名序列

```makefile
# Makefile 内容
all:
    @echo $(addprefix test_,/home/a.c b.c ./d.c)
```
bash 中执行 make   

```bash
$ make
test_/home/a.c test_b.c test_./d.c
```

### **取前缀函数**
`$(basename <names...>)`
功能: 从文件名序列\ <names\> 中取出各个文件名的前缀
返回: 文件名序列\<names\> 中各个文件名的前缀, 没有前缀则返回空字符串

```makefile
# Makefile 内容
all:
    @echo $(basename /home/a.c ./b.o ../c.a /home/.d .e)
```
在bash中执行make

```bash
$ make
/home/a ./b ../c /home/
```

### **过滤函数**
`$(filter <pattern...>,<text>)`
功能: 以 \<pattern\> 模式过滤字符串\ <text\>, *保留* 符合模式\<pattern\> 的单词, 可以有多个模式
返回: 符合模式 \<pattern\> 的字符串

```makefile
all:
    @echo $(filter %.o %.a,program.c program.o program.a)
```
bash 中执行 make
```bash
$ make
program.o program.a
```

## Makefile命令前缀

Makefile 中书写shell命令时可以加2种前缀 @ 和 -, 或者不用前缀.
3种格式的shell命令区别如下:

- 不用前缀 :: 输出执行的命令以及命令执行的结果, 出错的话停止执行
- 前缀 @  :: 只输出命令执行的结果, 出错的话停止执行
- 前缀 -  :: 命令执行有错的话, 忽略错误, 继续执行

## Makefile打印

1、输出打印信息的方法是：$(warning xxxxx)，$(error xxxxx)
2、输出打印变量值的方法是：$(warning $(XXX))

