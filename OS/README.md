# OS
欢迎来到操作系统第一课。在真正打造操作系统前，有一条必经之路：你知道程序是如何运行的吗？一个熟练的编程老手只需肉眼看着代码，就能对其运行的过程了如指掌。

但对于初学者来说，这常常是很困难的事，这需要好几年的程序开发经验，和在长期的程序开发过程中对编程基本功的积累。

我记得自己最初学习操作系统的时候，面对逻辑稍微复杂的一些程序，在编写、调试代码时，就会陷入代码的迷宫，找不到东南西北。

不知道你现在处在什么阶段，是否曾有同样的感受？我常常说，扎实的基本功就像手里的指南针，你可以一步步强大到不依赖它，但是不能没有。

因此今天，我将带领你从“Hello World”起，扎实基本功，探索程序如何运行的所有细节和原理

``` c++
#include "stdio.h"
int main(int argc, const char **argv) {
    printf("Hello world\n");
    return 0;
}
```

计算机硬件是无法直接运行这个 C 语言文本程序代码的，需要 C 语言编译器，把这个代码编译成具体硬件平台的二进制代码。再由具体操作系统建立进程，把这个二进制文件装进其进程的内存空间中，才能运行。

![img](https://static001.geekbang.org/resource/image/f2/4a/f2b10135ed52436888a793327e4d5a4a.jpg)

其实，我们也可以手动控制以上这个编译流程，从而留下中间文件方便研究：
- gcc HelloWorld.c -E -o HelloWorld.i 预处理：加入头文件，替换宏。
- gcc HelloWorld.c -S -c HelloWorld.s 编译：包含预处理，将 C 程序转换成汇编程序。
- gcc HelloWorld.c -c HelloWorld.o 汇编：包含预处理和编译，将汇编程序转换成可链接的二进制程序。
- gcc HelloWorld.c -o HelloWorld 链接：包含以上所有操作，将可链接的二进制程序和其它别的库链接在一起，形成可执行的程序文件。

图灵机是一个抽象的模型，它是这样的：有一条无限长的纸带，纸带上有无限个小格子，小格子中写有相关的信息，纸带上有一个读头，读头能根据纸带小格子里的信息做相关的操作并能来回移动。

![img](https://static001.geekbang.org/resource/image/69/7d/6914497643dbb0aaefffc32b865dcf7d.png)

![img](https://static001.geekbang.org/resource/image/43/87/43812abfe104d6885815825f07622e87.jpg)

根据冯诺依曼体系结构构成的计算机，必须具有如下功能：
- 把程序和数据装入到计算机中；
- 必须具有长期记住程序、数据的中间结果及最终运算结果；
- 完成各种算术、逻辑运算和数据传送等数据加工处理；
- 根据需要控制程序走向，并能根据指令控制机器的各部件协调操作；
- 能够按照要求将处理的数据结果显示给用户。

为了完成上述的功能，计算机必须具备五大基本组成部件：
- 装载数据和程序的输入设备；
- 记住程序和数据的存储器；
- 完成数据加工处理的运算器；
- 控制程序执行的控制器；
- 显示处理结果的输出设备

![img](https://static001.geekbang.org/resource/image/bd/26/bde34df011c397yy42dc00fe6bd35226.jpg)

是不是非常简单？这次我们发现读头不再来回移动了，而是靠地址总线寻找对应的“纸带格子”。读取写入数据由数据总线完成，而动作的控制就是控制总线的职责了。

![img](https://static001.geekbang.org/resource/image/39/14/3991a042107b90612122b14596c65614.jpeg)

![img](https://static001.geekbang.org/resource/image/5d/6e/5d4889e7bf20e670ee71cc9b6285c96e.jpg)

重点回顾以上，对应图中的伪代码你应该明白了：现代电子计算机正是通过内存中的信息（指令和数据）做出相应的操作，并通过内存地址的变化，达到程序读取数据，控制程序流程（顺序、跳转对应该图灵机的读头来回移动）的功能。
这和图灵机的核心思想相比，没有根本性的变化。只要配合一些 I/O 设备，让用户输入并显示计算结果给用户，就是一台现代意义的电子计算机。
到这里，我们理清了程序运行的所有细节和原理。还有一点，你可能有点疑惑，即 printf 对应的 puts 函数，到底做了什么？而这正是我们后面的课程要探索的！

![img](https://static001.geekbang.org/resource/image/f2/bd/f2d31ab7144bf309761711efa9d6d4bd.jpg?wh=4335*3170)

简单解释一下，PC 机 BIOS 固件是固化在 PC 机主板上的 ROM 芯片中的，掉电也能保存，PC 机上电后的第一条指令就是 BIOS 固件中的，它负责检测和初始化 CPU、内存及主板平台，然后加载引导设备（大概率是硬盘）中的第一个扇区数据，到 0x7c00 地址开始的内存空间，再接着跳转到 0x7c00 处执行指令，在我们这里的情况下就是 GRUB 引导程序。

我们先来写一段汇编代码。这里我要特别说明一个问题：为什么不能直接用 C？
C 作为通用的高级语言，不能直接操作特定的硬件，而且 C 语言的函数调用、函数传参，都需要用栈。
栈简单来说就是一块内存空间，其中数据满足后进先出的特性，它由 CPU 特定的栈寄存器指向，所以我们要先用汇编代码处理好这些 C 语言的工作环境。

``` assembly
;eintr
mbt_hdr_flags equ 0x00010003
mbt_hdr_magic equ 0x1badb002  ; 多协议引导头魔数
mbt_hdr2_magic equ 0xe85250d6  ; 第二版多协议引导头魔数
global _start ; 导出_start符号
extern main; 导出外部的main函数的符号
[section .start.text] ;定义.start.text代码段
[bits 32] ;汇编成32未代码
_start;
  jmp _entry
align 8
mbt_hdr:
  dd mbt_hdr_magic
  dd mbt_hdr_flags
  dd -(mbt_hdr_magic+mbt_hdr_flags)
  dd mbt_hdr
  dd _start
  dd 0
  dd 0
  dd _entry
; 以上是grub所需要的头
align 8
mbt2_hdr:
  dd mbt_hdr2_magic
  dd 0
  dd mbt2_hdr_end - mbt2_hdr
  dd -(mbt_hdr2_magic + 0 + (mbt2_hdr_end - mbt2_hdr))
  dw 2, 0
  dd 24
  dd mbt2_hdr
  dd _start
  dd 0
  dd 0
  dw 3, 0
  dd 12
  dd _entry
  dd 0
  dw 0, 0
  dd 8
mbt2_hdr_end:
;以上是grub2所需要的头
align 8

_entry:
	;关中断
	cli
	;关不可屏蔽中断
	in al, 0x70
	or al, 0x80
	out 0x70,al
	;重新加载gdt
	lgdt [gdt_ptr]
	jmp dword 0x8 :_32bits_mode

_32bits_mode:
	;下面初始化c语言可能会用到的寄存器
	mov ax, 0x10
	mov ds, ax
	mov ss, ax
	mov es, ax
	mov fs, ax
	mov gs, ax
	xor eax,eax
	xor ebx,ebx
	xor ecx,ecx
	xor edx,edx
	xor edi,edi
	xor esi,esi
	xor ebp,ebp
	xor esp,esp
	;初始化栈，c语言需要栈才能工作
	mov esp,0x9000
	;调用c语言函数main
	call main
	;让cpu停止执行指令
halt_step:
	halt
	jmp halt_step


gdt_start:
knull_dsc: dq 0
kcode_dsc: dq 0x00cf9e000000ffff
kdata_dsc: dq 0x00cf92000000ffff
k16cd_dsc: dq 0x00009e000000ffff
k16da_dsc: dq 0x000092000000ffff
gdt_end:

gdt_ptr:
gdtlen	dw gdt_end-gdt_start-1
gdtbase	dd gdt_start

```

以上的汇编代码（/lesson01/HelloOS/entry.asm）分为 4 个部分：
1. 代码 1~40 行，用汇编定义的 GRUB 的多引导协议头，其实就是一定格式的数据，我们的 Hello OS 是用 GRUB 引导的，当然要遵循 GRUB 的多引导协议标准，让 GRUB 能识别我们的 Hello OS。之所以有两个引导头，是为了兼容 GRUB1 和 GRUB2。
2. 代码 44~52 行，关掉中断，设定 CPU 的工作模式。你现在可能不懂，没事儿，后面 CPU 相关的课程我们会专门再研究它。
3. 代码 54~73 行，初始化 CPU 的寄存器和 C 语言的运行环境。
4. 代码 78~87 行，GDT_START 开始的，是 CPU 工作模式所需要的数据，同样，后面讲 CPU 时会专门介绍。

### 控制计算机屏幕
接着我们再看下显卡，这和我们接下来要写的代码有直接关联。

计算机屏幕显示往往是显卡的输出，显卡有很多形式：集成在主板的叫集显，做在 CPU 芯片内的叫核显，独立存在通过 PCIE 接口连接的叫独显，性能依次上升，价格也是。

独显的高性能是游戏玩家们所钟爱的，3D 图形显示往往要涉及顶点处理、多边形的生成和变换、纹理、着色、打光、栅格化等。而这些任务的计算量超级大，所以独显往往有自己的 RAM、多达几百个运算核心的处理器。因此独显不仅仅是可以显示图像，而且可以执行大规模并行计算，比如“挖矿”。

我们要在屏幕上显示字符，就要编程操作显卡。其实无论我们 PC 上是什么显卡，它们都支持一种叫 VESA 的标准，这种标准下有两种工作模式：字符模式和图形模式。显卡们为了兼容这种标准，不得不自己提供一种叫 VGABIOS 的固件程序。

下面，我们来看看显卡的字符模式的工作细节。

它把屏幕分成 24 行，每行 80 个字符，把这（24*80）个位置映射到以 0xb8000 地址开始的内存中，每两个字节对应一个字符，其中一个字节是字符的 ASCII 码，另一个字节为字符的颜色值。如下图所示：

![img](https://static001.geekbang.org/resource/image/78/f5/782ef574b96084fa44a33ea1f83146f5.jpg?wh=3530*1605)

``` c++
#ifndef VGASTR_H_
#define VGASTR_H_

static inline void _strwrite(char* string) {
  char *p_strdst = (char*)(0xb8000); // 指向显存的开始地址
  while(*string) {
    *p_strdst = *string++;
    p_strdst += 2;
  }
  return;
}

inline void printf(char* fmt, ...) {
  _strwrite(fmt);
  return;
}

#endif // VGASTR_H_

```


``` c++
#include "vgastr.h"

int main(int argc, char **argv) {
  printf("Hello OS!");
  return 0;
}

```

下面我们用一张图来描述我们 Hello OS 的编译过程，如下所示：
![img](https://static001.geekbang.org/resource/image/cb/34/cbd634cd5256e372bcbebd4b95f21b34.jpg?wh=4378*4923)

## 内核结构与设计

从用户和应用程序的角度来看，内核之中有什么并不重要，能提供什么服务才是重要的，所以内核在用户和上层应用眼里，就像一个大黑盒，至于黑盒里面有什么，怎么实现的，就不用管了。

不过，作为内核这个黑盒的开发者，我们要实现它，就必先设计它，而要设计它，就必先搞清楚内核中有什么。

从抽象角度来看，内核就是计算机资源的管理者，当然管理资源是为了让应用使用资源。既然内核是资源的管理者，我们先来看看计算机中有哪些资源，然后通过资源的归纳，就能推导出内核这个大黑盒中应该有什么。

计算机中资源大致可以分为两类资源，一种是硬件资源，一种是软件资源。先来看看硬件资源有哪些，如下：
1. 总线，负责连接各种其它设备，是其它设备工作的基础。
2. CPU，即中央处理器，负责执行程序和处理数据运算。
3. 内存，负责储存运行时的代码和数据。
4. 硬盘，负责长久储存用户文件数据。
5. 网卡，负责计算机与计算机之间的通信。
6. 显卡，负责显示工作。
7. 各种 I/O 设备，如显示器，打印机，键盘，鼠标等。
![img](https://static001.geekbang.org/resource/image/28/14/28cc064d767d792071a789a5b4e7d714.jpg)
而计算机中的软件资源，则可表示为计算机中的各种形式的数据。如各种文件、软件程序等。

内核作为硬件资源和软件资源的管理者，其内部组成在逻辑上大致如下：
1. 管理 CPU，由于 CPU 是执行程序的，而内核把运行时的程序抽象成进程，所以又称为进程管理。
2. 管理内存，由于程序和数据都要占用内存，内存是非常宝贵的资源，所以内核要非常小心地分配、释放内存。
3. 管理硬盘，而硬盘主要存放用户数据，而内核把用户数据抽象成文件，即管理文件，文件需要合理地组织，方便用户查找和读写，所以形成了文件系统。
4. 管理显卡，负责显示信息，而现在操作系统都是支持 GUI（图形用户接口）的，管理显卡自然而然地就成了内核中的图形系统。
5. 管理网卡，网卡主要完成网络通信，网络通信需要各种通信协议，最后在内核中就形成了网络协议栈，又称网络组件。
6. 管理各种 I/O 设备，我们经常把键盘、鼠标、打印机、显示器等统称为 I/O（输入输出）设备，在内核中抽象成 I/O 管理器。

内核除了这些必要组件之外，根据功能不同还有安全组件等，最值得一提的是，各种计算机硬件的性能不同，硬件型号不同，硬件种类不同，硬件厂商不同，内核要想管理和控制这些硬件就要编写对应的代码，通常这样的代码我们称之为驱动程序。

硬件厂商就可以根据自己不同的硬件编写不同的驱动，加入到内核之中

以上我们已经大致知道了内核之中有哪些组件，但是另一个问题又出现了，即如何组织这些组件，让系统更加稳定和高效，这就需要我们从现有的一些经典内核结构里找灵感了。

### 宏内核

其实看这名字，就已经能猜到了，宏即大也，这种最简单适用，也是最早的一种内核结构。

宏内核就是把以上诸如管理进程的代码、管理内存的代码、管理各种 I/O 设备的代码、文件系统的代码、图形系统代码以及其它功能模块的代码，把这些所有的代码经过编译，最后链接在一起，形成一个大的可执行程序。

这个大程序里有实现支持这些功能的所有代码，向用户应用软件提供一些接口，这些接口就是常说的系统 API 函数。而这个大程序会在处理器的特权模式下运行，这个模式通常被称为宏内核模式。结构如下图所示。

![img](https://static001.geekbang.org/resource/image/eb/6b/eb8e9487475f960dccda0fd939999b6b.jpg)

尽管图中一层一层的，这并不是它们有层次关系，仅仅表示它们链接在一起。

为了理解宏内核的工作原理，我们来看一个例子，宏内核提供内存分配功能的服务过程，具体如下：
1. 应用程序调用内存分配的 API（应用程序接口）函数。
2. 处理器切换到特权模式，开始运行内核代码。
3. 内核里的内存管理代码按照特定的算法，分配一块内存。
4. 把分配的内存块的首地址，返回给内存分配的 API 函数。
5. 内存分配的 API 函数返回，处理器开始运行用户模式下的应用程序，应用程序就得到了一块内存的首地址，并且可以使用这块内存了。

上面这个过程和一个实际的操作系统中的运行过程，可能有差异，但大同小异。当然，系统 API 和应用程序之间可能还有库函数，也可能只是分配了一个虚拟地址空间，但是我们关注的只是这个过程。

上图的宏内核结构有明显的缺点，因为它没有模块化，没有扩展性、没有移植性，高度耦合在一起，一旦其中一个组件有漏洞，内核中所有的组件可能都会出问题。

开发一个新的功能也得重新编译、链接、安装内核。其实现在这种原始的宏内核结构已经没有人用了。这种宏内核唯一的优点是性能很好，因为在内核中，这些组件可以互相调用，性能极高。

为了方便我们了解不同内核架构间的优缺点，下面我们看一个和宏内核结构对应的反例。

### 微内核
微内核架构正好与宏内核架构相反，它提倡内核功能尽可能少：仅仅只有进程调度、处理中断、内存空间映射、进程间通信等功能（目前不懂没事，这是属于管理进程和管理内存的功能模块，后面课程里还会专门探讨的）。

这样的内核是不能完成什么实际功能的，开发者们把实际的进程管理、内存管理、设备管理、文件管理等服务功能，做成一个个服务进程。和用户应用进程一样，只是它们很特殊，宏内核提供的功能，在微内核架构里由这些服务进程专门负责完成。

微内核定义了一种良好的进程间通信的机制——消息。应用程序要请求相关服务，就向微内核发送一条与此服务对应的消息，微内核再把这条消息转发给相关的服务进程，接着服务进程会完成相关的服务。服务进程的编程模型就是循环处理来自其它进程的消息，完成相关的服务功能。其结构如下所示：

![img](https://static001.geekbang.org/resource/image/4b/64/4b190d617206379ee6cd77fcea231c64.jpg)

为了理解微内核的工程原理，我们来看看微内核提供内存分配功能的服务过程，具体如下：

1. 应用程序发送内存分配的消息，这个发送消息的函数是微内核提供的，相当于系统 API，微内核的 API（应用程序接口）相当少，极端情况下仅需要两个，一个接收消息的 API 和一个发送消息的 API。
2. 处理器切换到特权模式，开始运行内核代码。
3. 微内核代码让当前进程停止运行，并根据消息包中的数据，确定消息发送给谁，分配内存的消息当然是发送给内存管理服务进程。
4. 内存管理服务进程收到消息，分配一块内存。
5. 内存管理服务进程，也会通过消息的形式返回分配内存块的地址给内核，然后继续等待下一条消息。
6. 微内核把包含内存块地址的消息返回给发送内存分配消息的应用程序。
7. 处理器开始运行用户模式下的应用程序，应用程序就得到了一块内存的首地址，并且可以使用这块内存了。

微内核的架构实现虽然不同，但是大致过程和上面一样。同样是分配内存，在微内核下拐了几个弯，一来一去的消息带来了非常大的开销，当然各个服务进程的切换开销也不小。这样系统性能就大打折扣。

但是微内核有很多优点，首先，系统结构相当清晰利于协作开发。其次，系统有良好的移植性，微内核代码量非常少，就算重写整个内核也不是难事。最后，微内核有相当好的伸缩性、扩展性，因为那些系统功能只是一个进程，可以随时拿掉一个服务进程以减少系统功能，或者增加几个服务进程以增强系统功能。

微内核的代表作有 MACH、MINIX、L4 系统，这些系统都是微内核，但是它们不是商业级的系统，商业级的系统不采用微内核主要还是因为性能差。

好了，粗略了解了宏内核和微内核两大系统内核架构的优、缺点，以后设计我们自己的系统内核时，心里也就有了底了，到时就可以扬长避短了，下面我们先学习一点其它的东西，即分离硬件相关性，为设计出我们自己的内核架构打下基础。

### 分离硬件的相关性
我们会经常听说，Windows 内核有什么 HAL 层、Linux 内核有什么 arch 层。这些 xx 层就是 Windows 和 Linux 内核设计者，给他们的系统内核分的第一个层。

今天如此庞杂的计算机，其实也是一层一层地构建起来的，从硬件层到操作系统层再到应用软件层这样构建。分层的主要目的和好处在于屏蔽底层细节，使上层开发更加简单。

计算机领域的一个基本方法是增加一个抽象层，从而使得抽象层的上下两层独立地发展，所以在内核内部再分若干层也不足为怪。分离硬件的相关性，就是要把操作硬件和处理硬件功能差异的代码抽离出来，形成一个独立的软件抽象层，对外提供相应的接口，方便上层开发。

为了让你更好理解，我们举进程管理中的一个模块实现细节的例子：进程调度模块。通过这个例子，来看看分层对系统内核的设计与开发有什么影响。

一般操作系统理论课程都会花大量篇幅去讲进程相关的概念，其实说到底，进程是操作系统开发者为了实现多任务而提出的，并让每个进程在 CPU 上运行一小段时间，这样就能实现多任务同时运行的假象。

当然，这种假象十分奏效。要实现这种假象，就要实现下面这两种机制：
1. 进程调度，它的目的是要从众多进程中选择一个将要运行的进程，当然有各种选择的算法，例如，轮转算法、优先级算法等。
2. 进程切换，它的目的是停止当前进程，运行新的进程，主要动作是保存当前进程的机器上下文，装载新进程的机器上下文。

我们不难发现，不管是在 ARM 硬件平台上还是在 x86 硬件平台上，选择一个进程的算法和代码是不容易发生改变的，需要改变的代码是进程切换的相关代码，因为不同的硬件平台的机器上下文是不同的。所以，这时最好是将进程切换的代码放在一个独立的层中实现，比如硬件平台相关层，当操作系统要运行在不同的硬件平台上时，就只是需要修改硬件平台相关层中的相关代码，这样操作系统的移植性就大大增强了。

如果把所有硬件平台相关的代码，都抽离出来，放在一个独立硬件相关层中实现并且定义好相关的调用接口，再在这个层之上开发内核的其它功能代码，就会方便得多，结构也会清晰很多。操作系统的移植性也会大大增强，移植到不同的硬件平台时，就构造开发一个与之对应的硬件相关层。这就是分离硬件相关性的好处。

### 我们的选择
从前面内容中，我们知道了内核必须要完成的功能，宏内核架构和微内核架构各自的优、缺点，最后还分析了分离硬件相关层的重要性，其实说了这么多，就是为了设计我们自己的操作系统内核。

虽然前面的内容，对操作系统设计这个领域还远远不够，但是对于我们自己从零开始的操作系统内核这已经够了。

首先大致将我们的操作系统内核分为三个大层，分别是：
1. 内核接口层。
2. 内核功能层。
3. 内核硬件层。

#### 内核接口层
内核接口层，定义了一系列接口，主要有两点内容，如下：
1. 定义了一套 UNIX 接口的子集，我们出于学习和研究的目的，使用 UNIX 接口的子集，优点之一是接口少，只有几个，并且这几个接口又能大致定义出操作系统的功能。
2. 这套接口的代码，就是检查其参数是否合法，如果参数有问题就返回相关的错误，接着调用下层完成功能的核心代码。

#### 内核功能层
内核功能层，主要完成各种实际功能，这些功能按照其类别可以分成各种模块，当然这些功能模块最终会用具体的算法、数据结构、代码去实现它，内核功能层的模块如下：
1. 进程管理，主要是实现进程的创建、销毁、调度进程，当然这要设计几套数据结构用于表示进程和组织进程，还要实现一个简单的进程调度算法。
2. 内存管理，在内核功能层中只有内存池管理，分两种内存池：页面内存池和任意大小的内存池，你现在可能不明白什么是内存池，这里先有个印象就行，后面课程研究它的时候再详细介绍。
3. 中断管理，这个在内核功能层中非常简单：就是把一个中断回调函数安插到相关的数据结构中，一旦发生相关的中断就会调用这个函数。
4. 设备管理，这个是最难的，需要用一系列的数据结构表示驱动程序模块、驱动程序本身、驱动程序创建的设备，最后把它们组织在一起，还要实现创建设备、销毁设备、访问设备的代码，这些代码最终会调用设备驱动程序，达到操作设备的目的。

#### 内核硬件层
内核硬件层，主要包括一个具体硬件平台相关的代码，如下：
1. 初始化，初始化代码是内核被加载到内存中最先需要运行的代码，例如初始化少量的设备、CPU、内存、中断的控制、内核用于管理的数据结构等。
2. CPU 控制，提供 CPU 模式设定、开、关中断、读写 CPU 特定寄存器等功能的代码。
3. 中断处理，保存中断时机器的上下文，调用中断回调函数，操作中断控制器等。
4. 物理内存管理，提供分配、释放大块内存，内存空间映射，操作 MMU、Cache 等。
5. 平台其它相关的功能，有些硬件平台上有些特殊的功能，需要额外处理一下。

![img](https://static001.geekbang.org/resource/image/6c/3c/6cf68bebe4f114f00f848d1d5679d33c.jpg)


从上述文字和图示，可以发现，我们的操作系统内核没有任何设备驱动程序，甚至没有文件系统和网络组件，内核所实现的功能很少。这吸取了微内核的优势，内核小出问题的可能性就少，扩展性就越强。

同时，我们把文件系统、网络组件、其它功能组件作为虚拟设备交由设备管理，比如需要文件系统时就写一个文件系统虚拟设备的驱动，完成文件系统的功能，需要网络时就开发一个网络虚拟设备的驱动，完成网络功能。这些驱动一旦被装载，就是内核的一部分了，并不是像微内核一样作为服务进程运行。这又吸取了宏内核的优势，代码高度耦合，性能强劲。

这样的内核架构既不是宏内核架构也不是微内核架构，而是这两种架构综合的结果，可以说是混合内核架构，也可以说这是我们自己的内核架构……

好了，到这里为止，我们已经设计了内核，确定了内核的功能并且设计了一种内核架构用来组织这些功能，这离完成我们自己的操作系统内核又进了一步。

## 业界成熟的内核架构长什么样？
Linux 的基本思想是一切都是文件：每个文件都有确定的用途，包括用户数据、命令、配置参数、硬件设备等对于操作系统内核而言，都被视为各种类型的文件。Linux 支持多用户，各个用户对于自己的文件有自己特殊的权利，保证了各用户之间互不影响。

多任务则是现代操作系统最重要的一个特点，Linux 可以使多个程序同时并独立地运行。Linux 发展到今天，不是哪一个人能做到的，更不是一群计算机黑客能做到的，而是由很多世界级的顶尖科技公司联合开发，如 IBM、甲骨文、红帽、英特尔、微软，它们开发 Linux 并向 Linux 社区提供补丁，使 Linux 工作在它们的服务器上，向客户出售业务服务。

Linux 发展到今天其代码量近 2000 万行，可以用浩如烟海来形容，没人能在短时间内弄清楚。但是你也不用害怕，我们可以先看看 Linux 内部的全景图，从全局了解一下 Linux 的内部结构，如下图。

![img](https://static001.geekbang.org/resource/image/92/cb/92ec3d008c77bb66a148772d3c5ea9cb.png?wh=2160*1620)

![img](https://static001.geekbang.org/resource/image/97/c9/97e7e66f9dcbddb1294ef9b7552fbac9.jpg?wh=3046x1502)

我们就能发现 Linux 这么多模块挤在一起，之间的通信主要是函数调用，而且函数间的调用没有一定的层次关系，更加没有左右边界的限定。函数的调用路径是纵横交错的，从图中的线条可以得到印证。继续深入思考你就会发现，这些纵横交错的路径上有一个函数出现了问题，就麻烦大了，它会波及到全部组件，导致整个系统崩溃。

当然调试解决这个问题，也是相当困难的。同样，模块之间没有隔离，安全隐患也是巨大的。当然，这种结构不是一无是处，它的性能极高，而性能是衡量操作系统的一个重要指标。

这种结构就是传统的内核结构，也称为宏内核架构。

### Darwin-XNU 内核

我们先来看看 Darwin，Darwin 是由苹果公司在 2000 年开发的一个开放源代码的操作系统。

一个经久不衰的公司，必然有自己的核心竞争力，也许是商业策略，也许是技术产品，又或是这两者的结合。而作为苹果公司各种产品和强大的应用生态系统的支撑者——Darwin，更是公司核心竞争力中的核心。

苹果公司有台式计算机、笔记本、平板、手机，台式计算机、笔记本使用了 macOS 操作系统，平板和手机则使用了 iOS 操作系统。Darwin 作为 macOS 与 iOS 操作系统的核心，从技术实现角度说，它必然要支持 PowerPC、x86、ARM 架构的处理器。

Darwin 使用了一种微内核（Mach）和相应的固件来支持不同的处理器平台，并提供操作系统原始的基础服务，上层的功能性系统服务和工具则是整合了 BSD 系统所提供的。苹果公司还为其开发了大量的库、框架和服务，不过它们都工作在用户态且闭源。

下面我们先从整体看一下 Darwin 的架构。Darwin架构图什么？两套内核？惊不惊喜？由于我们是研究 Darwin 内核，所以上图中我们只需要关注内核 - 用户转换层以下的部分即可。显然它有两个内核层——Mach 层与 BSD 层。Mach 内核是卡耐基梅隆大学开发的经典微内核，意在提供最基本的操作系统服务，从而达到高性能、安全、可扩展的目的，而 BSD 则是伯克利大学开发的类 UNIX 操作系统，提供一整套操作系统服务。那为什么两套内核会同时存在呢？

MAC OS X（2011 年之前的称呼）的发展经过了不同时期，随着时代的进步，产品功能需求增加，单纯的 Mach 之上实现出现了性能瓶颈，但是为了兼容之前为 Mach 开发的应用和设备驱动，就保留了 Mach 内核，同时加入了 BSD 内核。Mach 内核仍然提供十分简单的进程、线程、IPC 通信、虚拟内存设备驱动相关的功能服务，BSD 则提供强大的安全特性，完善的网络服务，各种文件系统的支持，同时对 Mach 的进程、线程、IPC、虚拟内核组件进行细化、扩展延伸。

那么应用如何使用 Darwin 系统的服务呢？应用会通过用户层的框架和库来请求 Darwin 系统的服务，即调用 Darwin 系统 API。

在调用 Darwin 系统 API 时，会传入一个 API 号码，用这个号码去索引 Mach 陷入中断服务表中的函数。此时，API 号码如果小于 0，则表明请求的是 Mach 内核的服务，API 号码如果大于 0，则表明请求的是 BSD 内核的服务，它提供一整套标准的 POSIX 接口。

就这样，Mach 和 BSD 就同时存在了。Mach 中还有一个重要的组件 Libkern，它是一个库，提供了很多底层的操作函数，同时支持 C++ 运行环境。

依赖这个库的还有 IOKit，IOKit 管理所有的设备驱动和内核功能扩展模块。驱动程序开发人员则可以使用 C++ 面向对象的方式开发驱动，这个方式很优雅，你完全可以找一个成熟的驱动程序作为父类继承它，要特别实现某个功能就重载其中的函数，也可以同时继承其它驱动程序，这大大节省了内存，也大大降低了出现 BUG 的可能。

如果你要详细了解 Darwin 内核的话，可以自行阅读相应的代码。而在这里，你只要从全局认识一下它的结构就行了。


### Windows NT 内核
接下来我们再看下 NT 内核。现代 Windows 的内核就是 NT，我们不妨先看看 NT 的历史。

如果你是 90 后，大概没有接触过 MS-DOS，它的交互方式是你在键盘上输入相应的功能命令，它完成相应的功能后给用户返回相应的操作信息，没有图形界面。在 MS-DOS 内核的实现上，也没有应用现代硬件的保护机制，这导致后来微软基于它开发的图形界面的操作系统，如 Windows 3.1、Windows95/98/ME，极其不稳定，且容易死机。加上类 UNIX 操作系统在互联网领域大行其道，所以微软急需一款全新的操作系统来与之竞争。所以，Windows NT 诞生了。Windows NT 是微软于 1993 年推出的面向工作站、网络服务器和大型计算机的网络操作系统，也可做 PC 操作系统。

它是一款全新从零开始开发的新操作系统，并应用了现代硬件的所有特性，“NT”所指的便是“新技术”（New Technology）。而普通用户第一次接触基于 NT 内核的 Windows 是 Windows 2000，一开始用户其实是不愿意接受的，因为 Windows 2000 对用户的硬件和应用存在兼容性问题。随着硬件厂商和应用厂商对程序的升级，这个兼容性问题被缓解了，加之 Windows 2000 的高性能、高稳定性、高安全性，用户很快便接受了这个操作系统。这可以从 Windows 2000 的迭代者 Windows XP 的巨大成功，得到验证。

现在，NT 内核在设计上层次非常清晰明了，各组件之间界限耦合程度很低。下面我们就来看看 NT 内核架构图，了解一下 NT 内核是如何“庄严宏伟”。如下图：
![img](https://static001.geekbang.org/resource/image/c5/c9/c547b6252736375fcdb1456e6dfaa3c9.jpg?wh=4268*4905)

这样看 NT 内核架构，是不是就清晰了很多？但这并不是我画图画得清晰，事实上的 NT 确实如此。

这里我要提示一下，上图中我们只关注内核模式下的东西，也就是传统意义上的内核。

当然微软自己在 HAL 层上是定义了一个小内核，小内核之下是硬件抽象层 HAL，这个 HAL 存在的好处是：不同的硬件平台只要提供对应的 HAL 就可以移植系统了。小内核之上是各种内核组件，微软称之为内核执行体，它们完成进程、内存、配置、I/O 文件缓存、电源与即插即用、安全等相关的服务。每个执行体互相独立，只对外提供相应的接口，其它执行体要通过内核模式可调用接口和其它执行体通信或者请求其完成相应的功能服务。所有的设备驱动和文件系统都由 I/O 管理器统一管理，驱动程序可以堆叠形成 I/O 驱动栈，功能请求被封装成 I/O 包，在栈中一层层流动处理。Windows 引以为傲的图形子系统也在内核中。

显而易见，NT 内核中各层次分明，各个执行体互相独立，这种“高内聚、低偶合”的特性，正是检验一个软件工程是否优秀的重要标准。而这些你都可以通过微软公开的 WRK 代码得到佐证，如果你觉得 WRK 代码量太少，也可以看一看REACT OS这个号称“开源版”的 NT。

### 重点回顾
到这里，我们了解了 Linux、Darwin-XNU 和 Windows 的发展历史，也清楚了它们内部的组件和结构，并对它们的架构进行了对比，对比后我们发现：Linux 性能良好，结构异常复杂，不利于问题的排查和功能的扩展，而 Darwin-XNU 和 Windows 结构良好，层面分明，利于功能扩展，不容易产生问题且性能稳定。







## 虚幻与真实：程序中的地址如何转换？



## Cache与内存：程序放在哪儿？



## 



## 




## 




## 




## 




## 




## 





## 



## 




## 





## 




## 




## 


##




## 


## 



## 



## 



## 




## 




## 




## 




## 




## 





## 



## 




## 





## 




## 




## 




## 


## 



## 



## 



## 




## 




## 




## 




## 




## 





## 



## 




## 





## 




## 




## 




## 


## 



## 



## 



## 




## 




## 




## 




## 




## 





## 



## 




## 





## 




## 




## 

