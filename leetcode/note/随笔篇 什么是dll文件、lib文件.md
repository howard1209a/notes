---
title: 什么是dll文件、lib文件
date: 2023-10-29 13:19:49
tags: [日常笔记]
---

# 什么是 dll 文件、lib 文件

**DLL（Dynamic Link Library，动态链接库）**，在 Windows 系统中，许多应用程序并不是一个完整的可执行文件，它们被分割成一些相对独立的动态链接库，也就是 DLL 文件，放置于系统中。当我们执行某一个程序时，相应的 DLL 文件就会被调用。一个应用程序可有多个 DLL 文件，一个 DLL 文件也可能被几个应用程序所共用，这样的 DLL 文件被称为共享 DLL 文件。**DLL 文件一般被存放在 C:\Windows\System 目录下。**

例如，在 Windows 操作系统中，Comdlg32 DLL 执行与对话框有关的常见函数。因此，每个程序都可以使用该 DLL 中包含的功能来实现“打开”对话框。这有助于促进代码重用和内存的有效使用。

**通过使用 DLL，程序可以实现模块化，由相对独立的组件组成。**例如，一个计帐程序可以按模块来销售。可以在运行时将各个模块加载到主程序中（如果安装了相应模块）。因为模块是彼此独立的，所以程序的加载速度更快，而且模块只在相应的功能被请求时才加载。

此外，可以更为容易地将更新应用于各个模块，而不会影响该程序的其他部分。例如，您可能具有一个工资计算程序，而税率每年都会更改。当这些更改被隔离到 DLL 中以后，您无需重新生成或安装整个程序就可以应用更新。

### 程序使用 DLL 的优点

**1、使用较少的资源**

当多个程序使用同一个函数库时，DLL 可以减少在磁盘和物理内存中加载的代码的重复量。这不仅可以大大影响在前台运行的程序，而且可以大大影响其他在 Windows 操作系统上运行的程序。

**2、推广模块式体系结构**

DLL 有助于促进模块式程序的开发。这可以帮助您开发要求提供多个语言版本的大型程序或要求具有模块式体系结构的程序。模块式程序的一个示例是具有多个可以在运行时动态加载的模块的计帐程序。

**3、简化部署和安装**

当 DLL 中的函数需要更新或修复时，部署和安装 DLL 不要求重新建立程序与该 DLL 的链接。此外，如果多个程序使用同一个 DLL，那么多个程序都将从该更新或修复中获益。当您使用定期更新或修复的第三方 DLL 时，此问题可能会更频繁地出现。

我最近在 github 上下载了一个项目，需要配到 fftw 的第三方库，可是看到一堆 dll 文件，lib 文件，def 文件，头都晕了，不知道这些东西是什么，怎么用，下面就我查询的资料做一个小结。

据说，出现.lib .dll 这种文件的原因是为了保护源代码，这个以后机会我做个详细的查询，再写一篇文章，这里不做细述。

用 OpenCV 的开源库来举个例子看一下就知道了：

bin 文件夹里面放的都是 dll 文件；

lib 文件夹里面放的都是伴随 dll 文件的动态 lib 文件；

staticlib 文件夹里面放的才是真正的静态 lib 文件，和 dll 文件是独立的；

所以可以看出，lib 文件是有静态 lib 和动态 llib 之分的。

## 第一部分：静态 lib 文件，动态 lib 文件和 dll 文件的区别：

### 1. 静态 lib 文件

[这篇文章(点我点我 o((>ω< ))o)](https://blog.csdn.net/woainishifu/article/details/53445535)讲过如何生成并调用 lib 文件，其实那个使用“static Library”选项生成的 lib 文件就是静态 lib 文件。我们已经知道，在调用这种类型的 lib 文件的时候，只需要配置好头文件.h 的路径和库文件.lib 的路径，自己的程序就可以正确加载这些第三方代码为自己所用。这是因为：

静态 lib 文件实际上就是任意个 obj 文件的集合，而 obj 文件就是 cpp 文件编译之后产生的一种文件，一个 cpp 文件编译之后只会产生一个 obj 文件，而多个 obj 文件就可以连接生成 lib 文件。就像上一篇文章讲的那样，如果你工程里只有一个 lib.h 和 lib.cpp，那么编译后产生的 lib 文件实际上就是 lib.obj 文件的一个集合，但是如果你工程里还有其他的很多个 cpp 文件，那么就会在编译之后生成许多 obj 文件，然后最终只链接生成一个 lib 文件。

所以，**静态 lib 文件实际上是包含了所有的导出声明和实现。你如果把这个 lib 文件链接到自己的程序之后，这个 lib 文件中的所有代码都会嵌入进来，哪怕你只用到了其中一部分，剩下没用到的也进了你的代码。**这就不难想象会造成的后果了，虽然方便，但是如果大部分你都用不到，自然会导致你的库体积没有意义地变大，失去了使用动态库的灵活性，而且发布新的版本时必须要发布新的应用程序才行，而不是简单打个补丁就好。就是因为这种缺点，才会出现动态 dll 调用这种方式。（注：这世上所有事情的出现都是有理由的，如果静态 lib 能完成我们想要的功能，而没有缺点的话，就不会有第二种替代方案 dll 的出现！）

### 2. 动态 lib 文件和 dll 文件

把这两个放在一起来说，是因为一个 dll 工程生成一个 dll 文件的时候，总是伴随着生成一个 lib 文件，这个 lib 文件其实是一个动态的 lib，它的大小比静态 lib 要小很多，因为这个 lib 文件其实只是包含了一些函数索引信息，记录了 dll 中那些函数的入口和位置，dll 中才是具体的函数实现。那么为什么有了 dll，还要有一个 lib 呢？

这就是动态库链接的过程了，首先配置好动态 lib 库目录和动态 dll 目录，以及头文件的目录。（注：如何配置这些路径，请看我的下一篇文章）然后在你的代码中 include 用到的头文件，代码完成之后有两个过程：（1）编译：这个过程只需要用到这里的动态 lib 文件**【注：在静态 lib 的情况下，仍然只是在编译阶段用到 lib 文件，只不过静态 lib 文件包含了完整的实现，所以编译生成 exe 之后就可以直接用了而已】**，然后和你的代码打包到一起。（2）运行：这个过程就需要用到 dll 文件了，上面打包好的东西里面，只是记录下了那些用到的函数的入口和具体位置，并没有真正的实现代码，所以在运行期间，就由那些入口找到正确的位于 dll 中的位置，然后直接执行那些函数就行了。

从上面过程中也可以看出一个很清楚的事实：**dll 其实就是 exe，只不过它没有 main 函数，所以不能单独执行而已。**事实上， 在实际的使用过程中我们也发现，很多应用程序都并不是一个完整的单独可执行文件，它们被分割成一些单独的相对对立的动态链接库，只有在执行应用程序的时候，用到的 dll 才会被调用。这也就是为什么你经常打开某些程序，会出现“无法加载 XXX.dll”的原因了(**我在这里栽了很多坑啊，其实就是 dll 文件的原因**) (：冷漠.jpg

## 第二部分：静态 lib 和动态 dll 使用注意事项

通过第一部分的叙述，我们可以总结如下：

### 1. 调用静态 lib 库，需要用到的文件是：

（1）.h 文件，包含函数的声明，数据结构等东西，在调用 lib 的时候，需要把该头文件包含进你的代码；

（2）.lib 文件，包含具体的实现。

### 2. 调用动态 dll 库，需要用到的文件是：

\1. .h 文件，如上，同样需要包含到你的代码；
\2. .lib 文件，包含一些函数的入口和具体位置，必须在编译阶段引入这个文件，否则会报错。【根据查到的资料，如果没有这个动态 lib 文件或者不想用 lib 文件，需要用 Win32 的 API 函数 LoadLibrary 和 GetProcAddress 来装载】
\3. .dll 文件，实际的实现，在程序运行时动态调用。（3）.dll 文件，实际的实现，在程序运行时动态调用。

**正常情况下，你发行一个软件的过程应该是这样的：（选用第二种动态调用 dll 的方式）**

你的项目分成独立的几个模块，每个模块都有一个 dll 文件，然后有一个最终的程序入口 exe 文件，最后把 dll 文件和 exe 文件发行给用户。当用户每次点击这个 exe 文件的时候，自然会动态调用用到的 dll 文件。注意这个过程就不再需要什么.h 和.lib 了，那是别人调用你的库，再进行加工写代码时才需要做的事。上面说过 dll 其实就是个不能单独打开执行的 exe 而已，所以你最终发行给用户的只能是 dll 和 exe，当然你完全可以把所有的东西只打包在一个 exe 中。但是当你的软件非常大的时候，这样进行更新维护就非常不方便，如果有问题就得重新发行一次 exe，但是如果把各个模块单独弄成 dll，你只需要打个补丁，对那些有问题的 dll 进行更换就行了。

另外一个是把你的 dll 写好给别人拿来调用，以免别人做重复的工作。这个时候你刚开始提供的时候，就需要把.h 文件，.lib 文件，.dll 文件都提供给对方，然后如果你代码里面有改动，只需要重新编译一次 dll 给对方，替换掉原来的 dll 就可以了，非常方便！！！当然前提是，你的函数接口写得好。进行修改时只需要修改内部实际的代码，并不需要对接口改来改去！

**def 文件用于确定函数的导出名称,这会在链接的时候用到**
