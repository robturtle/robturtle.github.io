---
title: 且谈命令行
date: 2016-07-05 05:31:00
tags:
---

如果是一个不具备背景知识的人来学习一个新的知识领域，示例总是最快的入门材料；不过若是在有老司机带领上路的情况下呢，直入最本质的模型才是最具可持续性的方式。

虽说是面向零基础的，不过如果具备最基本的操作系统相关知识，特别是进程和文件系统，还是对理解很有帮助的。不过即使之前没接触过也没关系，必要的信息都穿插在下面的内容里了。

本文不涉及 shell 使用方面的讨论，如需要学习 shell 实战，见此：[http://linuxcommand.org/learning_the_shell.php](http://linuxcommand.org/learning_the_shell.php) 。 

<!-- more -->

## Machine & Primitive Language

先考虑什么是计算：

```
input -> process -> output
```

唔所有的计算都可以抽象成这个模型：接受输入，处理之后再输出。这个模型在计算机早期年代是非常具象的，输入可以是打了孔的纸带，输出可以是一个显示器或者一个打印机，而中间的处理逻辑，一开始就是机器本身的电路逻辑。一开始所有的计算机器都是专用的。

显然如果我们可以替换处理逻辑的话，输入输出设备和其他的基础设施都可以重用了，我们称这种具备替换逻辑能力的计算机为通用（general purpose) 计算机。当然我们也可以说我们是造了一个接受处理逻辑为输入的专用计算机。这就带来一个问题，既然程序要作为输入，我们就必须给它指定一种格式。于是我们就人为指定一些 0 和 1 的特殊排列代表某个机器操作。于是我们便有了指令集。我们可以说这个通用计算机能够理解这些指令，或者说，这些指令是机器的母语，或者叫 primitive language，通过这种语言的沟通我们可以使用机器具备的全部能力。

## OS & C Language

由于我们现在接触的命令行都是在符合 POSIX 标准的 Unix-like 系统下的，我们就直接跳过 IBM 大型机的蛮荒时代了。操作系统的一个重要作用就是通过对机器进行一系列抽象包装而在其基础上提供给用户更便利的计算模型。其中最重要的就是模拟多任务的能力以及把数据里不同部分映射到一个层级树的文件系统的能力。在多任务模型里，我们把一个正在运行中的处理逻辑称为进程，我们有一个调度程序来给每个进程分配运行时间并控制它们的产生和销毁。同时调度程序还保存了不同进程各自需要的变量的值，我们称为进程的环境或者上下文。这保证了不同进程都够在暂停的时候依然能够保存正确的状态。

由于我们的 Unix 系统是由 C 语言实现的，所以我们可以说 Unix 这个系统是建立于某个实际的计算机器之上的虚拟机器，而这个虚拟机器能够理解的母语为 C 。那么上面那些所有由 OS 提供的能力如进程和文件系统的能力，我们就能够通过 C 来使用它们。

那么讲了这么些废话，和命令行什么关系呢？命令行就是一个提供了用户能够和 OS 交流并使用 OS 的能力的一个最简陋的实现。即 OS 负责处理逻辑而命令行负责控制输入输出。当输出结束，命令行便开始等待下一个输入，如此周而复始的循环，便是 Read-Evaluate-Print-Loop (REPL)。这样一种对 OS 简陋的包装就像给一个机器加了简陋的铁皮外壳，我们就它 shell 好咯。

## Shell in the nutshell

通过上面的表述我们知道，OS，C，和 shell 不过是同一种计算模型的不同层次的表现。所以它们从设计的起源之日起就是深刻联系着的。我们现在来看看 shell 它究竟包含着那些内容。

一个 shell 它本身也只是一个普通程序，不过在以前，这是系统启动之后运行的第一个程序。如果我们想执行其他程序，我们必须通过提供给 shell 命令，让 shell 来代理切换程序运行的过程。从这个角度看，shell 提供了一个动态切换进程的模型。

另外，shell 又重新提供了一套它自己能够理解的 primitive language，这个语言包含了 if 和 for 等分支循环语句，并提供了变量赋值的能力。我们称这个语言为 shell script language。这个语言还具备一个独特的特点：它只有一种数据类型，那就是字符串。也因此它提供的基本运算就是连接字符串或者截取字符串。从这个角度看，shell 就是 shell script 这个语言的 REPL 解释器。

上面提到的两个方面紧密地结合在一起，提供了 shell 的切换进程运行能力。

### shell script 语法

在进入进程切换的具体细节前，我们先考察 shell script 里的基本语法。shell script 的语法相当简单，输入的字符串被用空格分隔开，每个分隔开的部分被称为一个参数(argument)。比如这行命令：

```shell
echo hello shell
```

就被分割为3个参数，分别是 `echo` `hello` 和 `shell`。正因为空格被用来分割参数了，我们的赋值语句就只好规定中间不能加空格了，那么我们赋值语句就像这样：

```shell
a=hello
```

以为所有的输入都默认是字符串字面量，为了求取变量 `a` 的值，我们必须使用 `$a` 的写法，或者 `${a}` 。

### 命令执行模型

#### 定位文件

现在我们看看一个命令是怎么被执行的，同样是这个命令：

```shell
echo hello world
```

 当参数被分割好后，第一个参数将被作为 shell 用来切换程序的线索，一般来说，有两种情况：

1. 第一个参数符合文件系统的路径格式，要么是相对路径 `./a.out` ，要么是绝对路径 `/usr/bin/echo`，那么 shell 将直接以该路径代表的文件作为将要执行的程序；
2. 否则，shell 将在 `PATH` 变量中列举的文件夹里依次寻找直到找到同名文件并将之作为将要执行的程序。

PATH 也是个简单的字符串，它有个特殊的格式，就是用 ":" 来分隔不同的路径。我们可以用 `echo` 命令来查看我们当前的 PATH 值：

```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

比如说当我们输入 `python` 的时候，它就依次在 `/usr/local/bin`, `/usr/bin`，`/bin`, `/usr/sbin` 和 `/sbin` 里寻找名字叫 `python` 的文件。我们可以用 `which` 这个命令来查询究竟是在哪里这个名叫 `python` 的文件被找到了。

```shell
$ which python
/usr/bin/python
```

#### 创建进程

文件找到了，我们就可以为这个文件代表的程序创建进程了。在之前我们说过，每个进程都有一个变量上下文，我们在创建新进程的时候，这些环境变量就都被拷贝到新的子进程里了。在 shell script 里，环境变量就是指所有被用 `export` 关键字修饰过的变量，比如：

```shell
a=yooo # a 此时不是环境变量，在子进程里不会存在这个变量
export a # 此时 a 变成了环境变量，所有新创建的子进程里都存在这个变量
```

我们可以通过 `env` 这个命令来查看此时的所有环境变量：

```shell
$ env
LANG=en_US.UTF-8
PWD=/Users/YangLiu
SHELL=/bin/zsh
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/Users/YangLiu/bin:/Library/Frameworks/Python.framework/Versions/Current/bin:/Users/YangLiu/.cabal/bin:/Users/YangLiu/.meteor
HOME=/Users/YangLiu
CLASSPATH=/Users/YangLiu/include/java//javarepl.jar:
a=yooo
```

一般来说，我们是用 `fork` 这个 C 函数来创建新进程的，这个函数会把原始进程原封不动地拷贝一份。因此这些变量自然就被子进程继承了。此时， shell 会考察之前定位好的用来执行的文件类型，分两种情况：

1. 如果文件类型是二进制机器码文件，就将这些机器码覆盖原始的进程代码，将命令行参数传给该新代码并开始执行；
2. 否则，不覆盖子进程代码。这样即相当于用原始的 shell 程序本身来执行文件里的内容。

我们看看 C 语言的 main 函数签名：

```c
int main(int argc, const char * const argv[]);
```

这个 argv 就是由 shell 传入的参数列表咯。比如：

```shell
 $ python -v
```

shell 先在 PATH 定位到文件 `/usr/bin/python`， 发现它是个二进制文件，就将它的代码载入，并将参数传入。对于 python 这个 c 程序的 main 函数来说，它此时所接受到的参数就是这样的：

```shell
argc == 2            # argument count: 总共有2个参数
argv[0] == "python"
argv[1] == "-v"
```

于是该程序便可以根据参数不同来执行不同的行为。

举另一个例子，当传入参数不是二进制文件时：假如我们写了文本文件 `lalala`，我们现在用 `cat` 命令来查看该文件里的内容：

```shell
# file: lalala
echo $0
echo $1
```

由于它不是二进制文件，新进程的代码段也不会被覆盖。这就相当于我们制造了一个 shell 的影分身，然后执行这些 shell script 语句。此时参数列表也被传递进来了，不过和 C 里面的 argv 不一样，这里面我们用 `$0` `$1` 等特殊变量来依次表示。

于是我们执行的结果就像这样：

```shell
$ ./lalala hahaha
./lalala
hahaha
```

#### 思考题

有了上述知识，我们就能够理解一个新手常见的坑了，比如我们有人写一个这样的文件：

```shell
# file: wrong
cd /
```

当我们执行这个文件的时候，我们发现 shell 并没有按照我们的预期切换目录，考虑一下这是为什么？

#### 小结

以上便是 shell 的命令执行模型，总结一下：

1. 当命令行输入之后，如果它是一个 shell script 语句（比如 `a=hello` 赋值语言，或 `if` 语句等）则由它本身执行该效果。所有 `which` 报告为 built-in 的命令也将由 shell 自身执行效果，比如 `cd` 和 `echo`。
2. 否则，将命令行按空格分割成参数列表，将第一个参数作为线索寻找对应文件。如果它是一个路径则直接找到，否则，在 `PATH` 变量里依次寻找该文件。
3. 当找到文件，创建一个本 shell 的镜像进程。如果是二进制，则讲该文件代码覆盖原始代码，否则不覆盖。
4. 接着将参数列表传递给新的程序，并开始执行。
5. 当程序执行完毕后，控制权将重新交回原始 shell 。

## 进程生命周期

我们知道进程的计算模型可以简化成 `input -> process -> output` 的形式，那么，一个进程什么时候终止呢？大致有几种情况：

1. 进程可以不需要输入，处理完任务后即返回；比如 `ls` 。
2. 进程需要输入，当在输入中发现某个指定字符时结束读取，执行处理和输出两步后返回；比如 `read` 在遇到回车符时停止读取输入。
3. 进程读取完全部输入。这样我们就需要一个标记输入已结束的状态。我们使用 `EOF` (End of File) 来表示，在 C 里常用 -1 表示。即相当于输入在读取到 `EOF` 这一虚拟字符时停止读取输入。（但其实`EOF` 不是字符，这就导致了 C 新手可能会碰到的用 char 类型接受 `getc()` 函数的陷阱，详见《C traps & pitfalls》）。
4. 当进程被操作系统的信号中断时，默认退出。（既然是默认即说明，可设置忽略某些信号，可设置适当的中断处理函数避免退出）。

这里说的返回，就是 c 里 main 函数里的 return 语句。其中 0 代表一切正常，反之代表出现了错误。我们可以用 `$?` 这一特殊变量查询上一个执行进程的返回值。

```shell
$ ls
...

$ echo $?
0

$ ls nonsense
ls: nonsense: No such file or directory

$ echo $?
1
```

### readline 按键约定

当输入是文件时，我们可以很容易判断输入何时结束。可是当输入时标准输入（即键盘）时，我们没法用直接的方法告诉系统输入何时结束。所以我们有了一系列的按键约定：

1. `Ctrl-D` 代表 `EOF` 。我们一般喜欢用 `^D` 这样的简写来表示这是一个 `Ctrl`+`D` 的按键。要注意一点：Unix 中要求所有文件必须以空行结尾。对于标准输入这一虚拟文件来说，它同样遵守这一约定。因此：`^D` 仅当你在一个空行时才有效。
2. `^C` 代表 `SIGINT` 这一中断信号。用来强制退出某进程。
3. `^Z` 代表 `SIGTSTP` 信号。用来挂起进程。被挂起的进程可以使用 `bg` 命令在后台继续运行。

除此以外还有一系列模仿 Emacs 按键绑定的按键系列。这些被制作成一个叫做 `readline` 的库供大部分 shell 使用。

考虑命令行，它作为一个普通程序，它的结束模式是怎样的呢？显然是读取到输入结束为止。因此，我们在标准输入里输入`^D` 即等价于退出该命令行。一般来说大部分的 REPL 都遵循这一约定（`python` 程序以前需要 readline 库的支持，不知道现在是否已经内置了）。因此，我们可以通过连续按`^D` 从多个嵌套 REPL 里退出回来：

```shell
$ sh
sh-3.2$ bash
bash-3.2$ python
>>> ^D
bash-3.2$ ^D
sh-3.2$ ^D
$ ^D
[ 退出最外层 shell 了 ]
```

对于其他读取到输入结尾的程序来说，我们也需要用 `^D` 来指示结束。比如我们用 `cat` 来写入一个文件：

```shell
$ cat > file
lalala
hahaha
yooo
^D

$ cat file
lalala
hahaha
yooo
```

## 其他琐碎

### 参数分类

前面我们说了，命令行的语法实质就是空格分隔的参数列表，这在 main 函数的签名里也可以看出来，所有的参数都是平等的。不过我们也可以根据参数的形式和位置来进一步分类。对一个典型的命令行的命令，我们可以初略地把语法写成这样：

```shell
command -options arguments -options -- arguments
```

1. 因为第一个参数被用来寻找需要执行的程序，我们称这个参数为 `command`，所以我们常用这第一个参数指代命令或程序本身；
2. 我们把 '-' 打头的参数称为 `option` 即选项，他们一般是用来调整程序的行为的；
3. 其他非 '-' 打头的参数我们继续称为 `argument` 即参数，我们一般用 `argument` 来指示程序的输入。（但也未必，这完全取决于作者的喜好。比如 `git clone` 里这个 `clone` 就不是表示输入。我们一般把这种比较”后现代“风格的参数起个名字叫 `subcommand` 即子命令。从这个例子也可以看出我们完全可以根据需要构建自己需要的抽象分类）
4. 一般来说我们约定中有个特殊 `option` 是 “--“，在它之后的所有参数都不被看作选项。这是为了避免某些脑残把一些文件起了个 '-' 打头的名字之类的。

现在我们看看一个具体的例子：

```shell
gcc -std=c++11 libmylib.a -I ~/include -- --brain_fucked_filename.c
```

对各个参数分类的工作留作练习。另外要注意，有些参数是从属于前面的选项的而不是属于命令的。在这里就是 `~/include` 从属于 `-I` 这个选项。

### Shebang

前面我们说了，如果文件不是二进制，将由 shell 本身来执行文件里的内容。可有的时候，我们希望用其他的解释器来执行。所以 Unix-like 的 shell 里一般提供一个功能，我们可以用一种特殊的称为 shebang 的首行注释来提示 shell 应该载入哪个其他的程序代码。比如我们可以：

```python
#!/usr/bin/python
import sys
print "hello, python"
```

shell 就会先载入 python 的代码，再执行剩下的内容。不过有一个问题，就是在其他人的机器里，python 未必是安装在这个地方，而 shebang 又不支持 PATH lookup，只能写文件的绝对路径。那么怎么模拟 PATH 查找的过程呢？我们可以用之前提过的 `env` 命令：

```shell
$ /usr/bin/env ls
```

等价于：

```shell
$ ls
```

所以我们的文件就可以写成这样：

```python
#!/usr/bin/env python
import sys
print "hello, python"
```

现在我们看一个好玩的例子：

```shell
#!/usr/bin/env ls
# file: blahblah
```

```shell
$ ./blahblah
```

考虑一下，输出是什么？

```shell
$ ./blahblah .
```

这样呢？

如果不能理解为何会输出这样的结果，不妨把上面的命令执行流程再看一遍。

### redirect & pipe

此处关于 shell 如何控制输入的来源和输出的目标地，比较简单，自行查阅资料即可。

### background execution

我们可以通过把 `&` 加在命令行后面让程序在后台运行。不过嘛。。。你如果想让你的程序作为守护进程(daemon) 而不被挂起或杀死的话，这样做是不保险的。具体来说，除非系统 `huponexit` 为 `off`，或者用户退出前和该进程解绑，该进程才不会被退出时的 `SIGHUP` 信号终止。而解绑的进程最好还不能和标准 IO 有交互。嘛，研究得那么麻烦，还不如去好好学一学`screen` 或 `tmux` 呢。

另外对许多 web 框架，它们也提供一些运行守护进程的方法。比如`node` 的 `forever`。