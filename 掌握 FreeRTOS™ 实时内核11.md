# **掌握 FreeRTOS™ 实时内核**

这是 161204 副本，尚未涵盖 FreeRTOS V9.0.0，FreeRTOS V10.0.0 或低功耗无滴答操作。请定期查看 https://www.freertos.org/ 以获取本书的其他文档和更新。 有关 FreeRTOS V9.x.x 的信息，请参见 https://www.freertos.org/FreeRTOS-V9.html。 有关 FreeRTOS V10.x.x 的信息，请参阅 https://www.freertos.org/FreeRTOS-V10.html。 使用 FreeRTOS V9.x.x 以后创建的应用程序可以在编译时静态分配所有内核对象，从而无需包含堆内存管理器。



本文是免费提供的。 作为回报，我们要求您使用 https://www.freertos.org/RTOS-contact-and-support.html 上的业务联系电子邮件链接提供反馈，评论和更正。谢谢。



# **前言**

## 小型嵌入式系统中的多任务处理

### 关于 FreeRTOS

FreeRTOS 由 Real Time Engineers Ltd. 独家拥有，开发和维护。Real Time Engineers Ltd. 与世界领先的芯片公司密切合作十多年，为您提供屡获殊荣的商业级和完全免费的高质量软件。



FreeRTOS 非常适合使用微控制器和微处理器的深度嵌入式实时应用程序。这种类型的应用程序通常包括硬件和软件时间要求的混合。



软实时设备是指那些规定了一个时间期限，但超过这个期限不会使系统失效的设备。例如，响应键击的速度太慢可能会使系统看起来反应迟钝，令人恼火，而实际上没有使其无法使用。



硬性实时设备是指那些规定了一个时间期限，而超过这个期限将导致系统的绝对失败。例如，如果驾驶员的安全气囊对碰撞传感器输入的响应太慢，这带来的坏处会超过好处。



FreeRTOS 是一个实时内核（或实时调度程序），可以在其上构建嵌入式应用程序以满足其硬实时要求。 它允许将应用程序组织为独立执行线程的集合。 在只有一个核心的进程中，任何时候都只能执行一个线程。 内核通过检查应用程序设计者分配给每个线程的优先级来决定应该执行哪个线程。 在最简单的情况下，应用程序设计人员可以为实现硬实时要求的线程分配更高的优先级，并为实现软实时要求的线程分配更低的优先级。 这将确保硬实时线程始终在软实时线程之前执行，但优先级分配决策并不总是那么简单。



如果你还没有完全理解前一段的概念，不要担心。下面几章提供了详细的解释，并提供了许多示例，以帮助您理解如何使用实时内核，特别是如何使用 FreeRTOS。



### 价值主张

FreeRTOS 史无前例的全球成功来自其令人信服的价值主张; FreeRTOS 是专业开发的，严格的质量控制，健壮，支持，不包含任何知识产权所有权模糊，是真正自由使用在商业应用程序，没有任何要求公开您的专有源代码。你可以使用 FreeRTOS 将产品推向市场，甚至无需与实时工程师公司 (Real Time Engineers ltd.) 沟通，更不用说支付任何费用，而成千上万的人就是这样做的。如果您希望在任何时候获得额外的备份，或者您的法律团队需要额外的书面担保或赔偿，那么有一个简单的低成本商业升级路径。平和的心态伴随着这样的知识: 你可以在任何时候选择商业路线。



### 术语说明

在 FreeRTOS 中，每个执行线程都称为一个 “任务”。在嵌入式社区中对术语没有共识，但是我更喜欢 task 而不是 thread，因为 thread 在应用程序的某些领域可以有更具体的含义。



### 为什么使用实时内核

有许多成熟的技术可以在不使用内核的情况下编写优秀的嵌入式软件，如果正在开发的系统很简单，那么这些技术可能提供最合适的解决方案。在更复杂的情况下，使用内核可能是更好的选择，但是发生交叉点的地方总是主观的。



如前所述，任务优先级可以帮助确保应用程序满足其处理截止期限，但是内核也可以带来其他不太明显的好处。下面简要列出了其中一些。



- **抽象出时间信息**：内核负责执行计时，并为应用程序提供一个与时间相关的 API。这使得应用程序代码的结构更简单，并且整个代码的尺寸更小。
- **可维护性/可扩展性**：抽象出计时细节可以减少模块之间的相互依赖，并允许软件以可控和可预测的方式发展。此外，内核负责计时，因此应用程序性能不太容易受到底层硬件变化的影响。

- **模块化**：任务是独立的模块，每个模块都应该有一个明确的目标。
- **团队开发**：任务也应该有明确的接口，允许团队更容易的开发。

- **更容易测试**：如果任务是定义良好的独立模块，具有干净的接口，则可以单独测试它们。
- **代码重用**：更高的模块性和更少的相互依赖性导致代码可以更轻松地重复使用。

- **提高效率**：使用内核可以使软件完全由事件驱动，因此轮询没有发生的事件不会浪费处理时间。代码只在必须执行某些操作时才执行。与效率节省相反的是需要处理 RTOS 滴答中断，并将执行从一个任务切换到另一个任务。然而，不使用 RTOS 的应用程序通常包含某种形式的滴答中断。
- **空闲时间**：在启动调度程序时自动创建空闲任务。只要没有应用程序任务需要执行，它就执行。空闲任务可以用来测量空闲处理能力，执行背景检查，或者简单地将处理器置于低功耗模式。

- **功耗管理**：通过使用 RTOS 获得的效率提高允许处理器在低功耗模式下花费更多时间。每次空闲任务运行时将处理器置于低功耗状态，可以显著降低功耗。FreeRTOS 也有一个特殊的无滴答模式。使用无滴答模式允许处理器进入比其他方式更低的功耗模式，并在低功耗模式下保持更长时间。
- **灵活的中断处理**：通过将中断处理推迟到应用程序编写者创建的任务或 FreeRTOS 守护程序的任务中执行，可以使中断处理程序保持非常短。

- **混合处理需求**：简单的设计模式可以在应用程序中实现周期性、连续性和事件驱动处理的混合。此外，可以通过选择适当的任务和中断优先级来满足硬实时和软实时需求。



### FreeRTOS 的功能

FreeRTOS 具有以下标准功能：

- 抢占或合作操作
- 非常灵活任务优先级分配

- 灵活、快速、轻量级任务通知机制
- 队列

- 二进制信号量
- 计数信号量

- 互斥量
- 递归互斥量

- 软件定时器
- 事件组

- 滴答钩子函数
- 空闲钩子函数

- 堆栈溢出检查
- 跟踪记录

- 任务运行时统计
- 信息收集

- 可选的商业许可和支持
- 完全中断嵌套模型（适用于部分架构）

- 适用于极低功耗应用的无滴答功能
- 适当的软件管理中断堆栈（这可以帮助节省RAM）



### 许可，以及 FreeRTOS，OpenRTOS 和 SafeRTOS 系列

**FreeRTOS** 开源许可证旨在确保：

1. FreeRTOS 可以用于商业应用。
2. FreeRTOS 本身仍然对每个人免费开放。

1. FreeRTOS 用户保留其知识产权的所有权。

有关最新的开源许可信息，请访问 [http://www.FreeRTOS.org/license](http://www.freertos.org/license)。

**OpenRTOS** 是FreeRTOS的商业许可版本，由 Real Time Engineers Ltd. 许可，由第三方提供。

**SafeRTOS** 与 FreeRTOS 具有相同的使用模式，但是是根据声称符合各种国际公认的安全相关标准所必需的实践，程序和流程开发的。

## 包含的源文件和项目

### 获取随附本书的示例

本书中提供的所有示例的源代码，预配置项目文件和完整构建说明都在随附的 zip 文件中提供。如果您没有收到该书的副本，您可以从 [http://www.FreeRTOS.org/Documentation/code](http://www.freertos.org/Documentation/code) 下载 zip 文件。zip 文件可能不包含最新版本的 FreeRTOS。

本书中包含的屏幕截图是在使用 FreeRTOS Windows 移植版在 Microsoft Windows 环境中执行示例时拍摄的。使用 FreeRTOS Windows 移植版的项目已预先配置为使用 Visual Studio 的免费 Express 版本进行构建， 可以从下载 http://www.microsoft.com/express。 请注意，虽然 FreeRTOS Windows 移植版提供了方便的评估，测试和开发平台，但它并不能提供真正的实时行为。



# **FreeRTOS 的发行**

## 引言与范围

FreeRTOS 作为单个 zip 文件存档分发，包含所有官方 FreeRTOS 移植和大量预配置的演示应用程序。



### 范围

本章旨在帮助用户通过以下方式使用 FreeRTOS 文件和目录来定位自己：

- 提供 FreeRTOS 目录结构的顶级视图。
- 描述任何特定 FreeRTOS 项目实际需要哪些文件。

- 介绍演示应用程序。
- 提供有关如何创建新项目的信息。

此处的描述仅涉及官方 FreeRTOS 的发行版。 本书附带的示例使用略有不同的组织。



## 了解 FreeRTOS 的分发

### 定义：FreeRTOS 移植

FreeRTOS 可以使用大约 20 种不同的编译器构建，并且可以运行在 30 多种不同的处理器架构上。 每个受支持的编译器和处理器组合都被视为一个单独的 FreeRTOS 移植。



### 编译 FreeRTOS

FreeRTOS 可以被认为是一个库，它可以提供多任务功能，而不是裸机应用程序。

FreeRTOS 是以一组 C 源文件提供给用户。 某些源文件对所有移植都是通用的，而某些源文件则特定于移植。 构建源文件作为项目的一部分，以使 FreeRTOS API 可用于您的应用程序。 为了方便您使用，每个官方 FreeRTOS 移植都配有演示应用程序。 演示应用程序已预先配置为构建正确的源文件，并包含正确的头文件。

演示应用程序应该 “开箱即用”，虽然有些演示比其他演示更旧，并且自演示程序发布以来构建工具的自身的更新可能会引起报错。 第 1.3 节描述了演示应用程序。



### FreeRTOSConfig.h

FreeRTOS 由名为 FreeRTOSConfig.h 的头文件配置。

FreeRTOSConfig.h 用于定制 FreeRTOS 以用于特定应用程序。 例如，FreeRTOSConfig.h 包含常量，如 configUSE_PREEMPTION，它的设置定义了将使用协作调度算法还是抢占调度算法。 由于 FreeRTOSConfig.h 包含特定于应用程序的定义，因此它应位于正在构建的应用程序部分的目录中，而不是位于包含 FreeRTOS 源代码的目录中。

为每个 FreeRTOS 移植提供演示应用程序，每个演示应用程序包含一个 FreeRTOSConfig.h 文件。 因此，永远不必从头开始创建 FreeRTOSConfig.h 文件。 相反，建议从 FreeRTOS 移植提供的演示应用程序中的 FreeRTOSConfig.h 开始，然后根据需求修改它。



### FreeRTOS 的官方发行版

FreeRTOS 发布在一个 zip 文件中。 该 zip 文件包含所有 FreeRTOS 移植的源代码，以及所有 FreeRTOS 演示应用程序的项目文件。 它还包含一系列 FreeRTOS+ 生态系统组件，以及一系列 FreeRTOS+ 生态系统演示应用程序。

不要被 FreeRTOS 发行版中的文件数量吓到！ 任何一个应用程序中只需要非常少量的文件。



### FreeRTOS 发行版的顶级目录

图 1 显示并描述了FreeRTOS发行版的第一级和第二级目录。

```plain
FreeRTOS
 │ │
 │ ├─Source 包含 FreeRTOS 源文件的目录
 │ │ │ └─Demo 包含预配置和移植特定 FreeRTOS 演示项目的目录
 │ FreeRTOS-Plus
 │
 ├─Source 包含一些 FreeRTOS+ 生态系统组件的源代码的目录
 │ └─Demo 包含 FreeRTOS+ 生态系统组件的演示项目的目录
```

图1. FreeRTOS 发行版中的顶级目录

zip 文件只包含 FreeRTOS 源文件的一个副本; 所有 FreeRTOS 演示项目和所有 FreeRTOS+ 演示项目都希望在 FreeRTOS/Source 目录中找到 FreeRTOS 源文件，如果目录结构发生变化，可能无法构建。



### 所有移植共有的 FreeRTOS 源文件

核心 FreeRTOS 源代码仅包含在所有 FreeRTOS 移植共有的两个 C 文件中。 这些被称为 tasks.c 和 list.c，它们直接位于 FreeRTOS/Source 目录中，如图 2 所示。除了这两个文件之外，以下源文件位于同一目录中：

- **queue.c**：queue.c 提供队列和信号量服务，如本书后面所述。 几乎总是需要 queue.c。
- **timers.c**：timers.c 提供软件计时器功能，如本书后面所述。 如果实际中将使用软件定时器，它只需要包含在构建中。

- **event_groups.c**：event_groups.c 提供事件组功能，如本书后面所述。 如果实际要使用事件组，它只需要包含在构建中。
- **croutine.c**：croutine.c 实现了 FreeRTOS 协同例程功能。 如果实际要使用协同例程，它只需要包含在构建中。协同程序旨在用于非常小的微控制器，现在很少使用，因此不能保持与其他 FreeRTOS 功能相同的水平。本例中没有描述协同例程。

```plain
FreeRTOS
    │
    └─Source
        │
        ├─tasks.c        FreeRTOS 源文件 -总是需要
        ├─list.c         FreeRTOS 源文件 -总是需要
        ├─queue.c        FreeRTOS 源文件 -几乎总是需要
        ├─timers.c       FreeRTOS 源文件 -可选
        ├─event_groups.c FreeRTOS 源文件 -可选
        └─croutine.c     FreeRTOS 源文件 -可选
```

图2. FreeRTOS 目录树中的核心 FreeRTOS 源文件

有的人会意识到文件名可能导致命名空间冲突，因为许多项目已经包含具有相同名称的文件。 然而，我们认为现在更改文件的名称会有问题，这样做会破坏与使用 FreeRTOS 的数千个项目以及自动化工具和 IDE 插件的兼容性。



### FreeRTOS 特定移植的源文件

特定 FreeRTOS 移植的源文件包含在 FreeRTOS/Source/portable 目录中。 可移植目录按层次结构排列，首先是编译器，然后是处理器体系结构。 层次结构如图 3 所示。

如果你在使用编译器 [*compiler]* 的架构 [*architecture]* 的处理器上运行 FreeRTOS，除了核心 FreeRTOS 源文件之外，你还必须构建位于 FreeRTOS/Source/portable/[compiler]/[architecture] 目录中的文件。

正如将在第 2 章堆内存管理中所描述的，FreeRTOS 还认为堆内存分配是便携式层的一部分。 使用早于V9.0.0 的 FreeRTOS 版本的项目必须包含堆内存管理器。 从 FreeRTOS V9.0.0 开始，只有在 FreeRTOSConfig.h 中将 configSUPPORT_DYNAMIC_ALLOCATION 设置为 1 或者未定义 configSUPPORT_DYNAMIC_ALLOCATION 时才需要堆内存管理器。

FreeRTOS 提供了五个示例堆分配方案。 这五个方案名为 heap_1 到 heap_5，分别由源文件 heap_1.c 到heap_5.c 实现。 示例堆分配方案包含在 FreeRTOS/Source/portable/MemMang 目录中。 如果已将 FreeRTOS 配置为使用动态内存分配，则必须在项目中构建这五个源文件中的一个，除非您的应用程序提供了替代实现。

```plain
FreeRTOS 
  │
  └─Source
     │
     └─portable 包含所有特定移植源文件的目录
        │   
        ├─MemMang 包含 5 个备用堆分配源文件的目录
        │   
        ├─[compiler 1] 包含特定移植编译器 1 文件的目录
        │   │
        │   ├─[architecture 1] 包含移植编译器 1 架构 1 的文件
        │   ├─[architecture 2] 包含移植编译器 1 架构 2 的文件
        │   └─[architecture 3] 包含移植编译器 1 架构 3 的文件
        │
        └─[compiler 2] 包含特定移植编译器 2 文件的目录
            │
            ├─[architecture 1] 包含移植编译器 2 架构 1 的文件
            ├─[architecture 2] 包含移植编译器 2 架构 2 的文件
            └─[etc.]
```

图3. FreeRTOS 目录树中的特定移植源文件

### 包含路径

FreeRTOS 要求在编译器的 include 路径中包含三个目录。 如下：

1. FreeRTOS 核心头文件的路径，它始终是 FreeRTOS / Source / include。
2. 正在使用特定移植的 FreeRTOS 的源文件的路径。 如上所述，这是 FreeRTOS/Source/portable/[compiler]/[architecture]

1. FreeRTOSConfig.h 头文件的路径。

### 头文件

使用 FreeRTOS API 的源文件必须包含 FreeRTOS.h，然后是包含正在使用的 API 函数原型的头文件：task.h，queue.h，semphr.h，timers.h 或 event_groups.h。

## 演示应用程序



每个 FreeRTOS 移植都附带至少一个可以构建的演示应用程序，不会生成任何错误或警告，尽管某些演示比其他演示更旧，并且自演示程序发布以来构建工具的自身的更新可能会引起报错。

Linux ****用户须知：FreeRTOS 是在 Windows 主机上开发和测试的。 在 Linux 主机上构建演示项目时，偶尔会导致构建错误。 构建错误几乎总是与引用文件名时使用的字母大小写或文件路径中使用的斜杠字符的方向有关。 请使用 FreeRTOS 联系表（[http://www.FreeRTOS.org/contact](http://www.freertos.org/contact)）提醒我们任何此类错误。

演示应用程序有几个目的：

- 提供一个工作和预配置项目的示例，包含正确的文件，并设置正确的编译器选项。
- 允许以最小的设置或先验知识进行 “开箱即用” 的实验。

- 作为如何可以使用 FreeRTOS API 的演示。
- 作为可以创建真实应用程序的基础。

每个演示项目都位于 FreeRTOS/Demo 目录下的唯一子目录中。 子目录的名称表示演示项目所涉及的移植。

每个演示应用程序也由 FreeRTOS.org 网站上的网页描述。网页包含以下信息：

- 如何在 FreeRTOS 目录结构中找到演示的项目文件。
- 项目配置使用的硬件。

- 如何设置运行演示的硬件。
- 如何构建演示。

- 如何演示预期行为。

所有演示项目都创建了常见演示任务的子集，其实现包含在 FreeRTOS/Demo/Common/Minimal 目录中。 常见的演示任务纯粹是为了演示如何使用 FreeRTOS API —— 它们没有实现任何特定的有用功能。

最近的演示项目也可以构建一个初学者的 'blinky' 项目。 Blinky 项目是非常基本的。 通常，他们只创建两个任务和一个队列。

每个演示项目都包含一个名为 main.c 的文件。 它包含 main() 函数，从中创建所有演示应用程序任务。 有关该演示的特定信息，请参阅各个 main.c 文件中的注释。

FreeRTOS/Demo 目录层次结构如图 4 所示。

```plain
FreeRTOS
    │
    └─Demo        包含所有演示项目的目录
       │
       ├─[Demo x] 包含构建演示'x'的项目文件
       │
       ├─[Demo y] 包含构建演示'y'的项目文件
       │
       ├─[Demo z] 包含构建演示'z'的项目文件
       │
       └─Common   包含由所有演示应用程序构建的文件
```

图 4. 演示目录层次结构

## 创建一个 FreeRTOS 项目

### 适配其中一个提供的演示项目

每个 FreeRTOS 移植都附带至少一个预配置的演示应用程序，该应用程序应该构建时不会出现错误或警告。 建议通过适配其中一个现有项目来创建新项目。这将允许项目包含正确的文件，安装正确的中断处理程序以及正确的编译器选项集。



要从现有演示项目开始新的应用程序：



1. 打开提供的演示项目，确保按预期构建和执行。 
2. 删除定义演示任务的源文件。 可以从项目中删除位于 Demo/Common 目录中的任何文件。

1. 除了 prvSetupHardware() 和 vTaskStartScheduler() 之外，删除 main() 中的所有函数调用，如清单1.4 所示。
2. 检查项目是否仍可构建。



按照这些步骤创建一个包含正确 FreeRTOS 源文件的项目，但不定义任何功能。

```c
int main( void )
{
    /* 执行必要的任何硬件设置。 */
    prvSetupHardware();

    /* ---可以在这里创建应用程序任务 ---*/
    /* 启动已创建的任务。 */
    vTaskStartScheduler();

    /* 只有当没有足够的堆来启动调度程序时，执行才会到达这里。 */
    for( ;; );
    return 0;
}
```

清单 1. 新 main() 函数的模板



### 从头开始创建一个新项目

如前所述，建议从现有的演示项目创建新项目。 如果不希望这样，则可以使用以下过程创建新项目：



1. 使用您选择的工具链，创建一个尚未包含任何 FreeRTOS 源文件的新项目。
2. 确保可以构建新项目，下载到目标硬件并执行。

1. 只有当您确定已经有一个工作项目，将表 1 中详述的 FreeRTOS 源文件添加到项目中。
2. 将为移植提供的演示项目使用的 FreeRTOSConfig.h 头文件复制到项目目录中。

1. 将以下目录添加到路径中，项目将搜索以查找头文件：

1. 1. FreeRTOS/Source/include
   2. FreeRTOS/Source/portable/[compiler]/[architecture] 其中 [compiler] 和 [architecture] 适合您选择的移植

1. 1. 包含 FreeRTOSConfig.h 头文件的目录

1. 从相关演示项目中复制编译器设置。
2. 安装可能需要的任何 FreeRTOS 中断处理程序。 使用描述正在使用移植的网页以及为使用移植提供的演示项目作为参考。



表 1. 要包含在项目中的 FreeRTOS 源文件

| 文件              | 位置                                                         |
| ----------------- | ------------------------------------------------------------ |
| tasks.c           | FreeRTOS/Source                                              |
| queue.c           | FreeRTOS/Source                                              |
| list.c            | FreeRTOS/Source                                              |
| timers.c          | FreeRTOS/Source                                              |
| event_groups.c    | FreeRTOS/Source                                              |
| 所有 C 和汇编文件 | FreeRTOS/Source/portable/[compiler]/[architecture]           |
| heap_n.c          | FreeRTOS/Source/portable/MemMang，n 可以是 1，2，3，4，5。这个文件在 FreeRTOS V9.0.0 中是可选的。 |

使用早于 V9.0.0 的 FreeRTOS 版本的项目必须构建一个 heap_n.c 文件。从 FreeRTOS V9.0.0 开始，只有在 FreeRTOSConfig.h 中将 configSUPPORT_DYNAMIC_ALLOCATION 设置为 1 或未定义 configSUPPORT_DYNAMIC_ALLOCATION 时才需要 heap_n.c 文件。 有关更多信息，请参阅第 2 章堆内存管理。

## 数据类型和编码风格指南

### 数据类型

FreeRTOS 的每个端口都有一个唯一的 portmacro.h 头文件，其中包含两个端口特定数据类型的定义（其中包括）：TickType_t 和 BaseType_t。 表 2 中描述了这些数据类型。



表 2. FreeRTOS使用的端口特定数据类型

| 使用的宏或 typedef | 实际类型                                                     |
| ------------------ | ------------------------------------------------------------ |
| TickType_t         | FreeRTOS 配置一个称为节拍中断的周期性中断。 自 FreeRTOS 应用程序启动以来发生的节拍中断数称为节拍计数。 刻度计数用作时间的度量。 两次滴答中断之间的时间称为滴答期。 时间被指定为滴答期的倍数。 TickType_t 是用于保存刻度计数值和指定时间的数据类型。 TickType_t 可以是无符号 16 位类型，也可以是无符号 32 位类型，具体取决于 FreeRTOSConfig.h 中 configUSE_16_BIT_TICKS 的设置。如果 configUSE_16_BIT_TICKS 设置为 1，则 TickType_t 定义为 uint16_t。 如果 configUSE_16_BIT_TICKS 设置为 0，则 TickType_t 定义为 uint32_t。使用 16 位类型可以极大地提高 8 位和 16 位体系结构的效率，但严重限制了可以指定的最大块周期。 没有理由在 32 位架构上使用 16 位类型。 |
| BaseType_t         | 这始终被定义为架构的最有效数据类型。 通常，这是 32 位架构上的 32 位类型，16 位架构上的 16 位类型和 8 位架构上的8位类型。 BaseType_t 通常用于只能采用非常有限的值范围的返回类型，以及 pdTRUE/pdFALSE 类型的布尔值。 |

有些编译器会使所有不合格的 char 变量无符号，而其他编译器会对它们进行有符号处理。 出于这个原因，FreeRTOS 源代码代码显式限定 char 的每个用户都使用 signed 或 unsigned，除非 char 用于保存 ASCII 字符，或者指向 char 的指针用于指向字符串。

不使用普通 int 类型。



### 变量名

变量的前缀是它们的类型：



- c 表示 char
- s 表示 int16_t（short）

- l表示 int32_t（long）
- x 表示 BaseType_t 和任何其他非标准类型（结构，任务句柄，队列句柄等）



如果变量是无符号的，则它也带有 u 前缀。 如果变量是指针，则它也带有 p 前缀。 例如，uint8_t 类型的变量将以 uc 为前缀，而指向 char 的类型指针的变量将以 pc 为前缀。



### 函数名

函数以它们返回的类型和它们在其中定义的文件为前缀。 例如：



- *v***Task**PrioritySet() 返回一个 void，并定义在 **task**.c 中。
- *x***Queue**Receive() 返回一个 BaseType_t类型的变量，并定义在 **queue**.c 中。

- *pv***Timer**GetTimerID() 返回一个 void类型的指针，并定义在 **timers**.c 中。



文件范围（私有）函数以 prv 为前缀。



### 格式化

1 个 Tab 总是等于 4 个空格。



### 宏名

大多数宏以大写字母书写，并以小写字母为前缀，表示宏的定义位置。 表 3 提供了前缀列表。

表 3. 宏的前缀

| 前缀   | 宏定义的位置              | 例子                     |
| ------ | ------------------------- | ------------------------ |
| port   | portable.h 或 portmacro.h | **port**MAX_DELAY        |
| task   | task.h                    | **task**ENTER_CRITICAL() |
| pd     | projdefs.h                | **pd**TRUE               |
| config | FreeRTOSConfig.h          | **config**USE_PREEMPTION |
| err    | projdefs.h                | **err**QUEUE_FULL        |

请注意，信号量 API 几乎完全是作为一组宏编写的，但遵循函数命名约定，而不是宏命名约定。表 4 中定义的宏在整个 FreeRTOS 源代码中使用。



表 4. 常见的宏定义

| 宏      | 值   |
| ------- | ---- |
| pdTRUE  | 1    |
| pdFALSE | 0    |
| pdPASS  | 1    |
| pdFAIL  | 0    |

### 构造中间过度类型的理由

FreeRTOS 源代码可以使用不同的编译器进行编译，所有编译器的编写方式和生成警告的方式都有所不同。 特别是，不同的编译器希望以不同的方式使用转换。 因此，FreeRTOS 源代码包含比通常保证更多的类型转换。

# **堆内存管理**

从 FreeRTOS V9.0.0 开始 FreeRTOS 应用程序可以完全静态分配，无需包含堆内存管理器。

## 引言与范围

### 先决条件

FreeRTOS 是作为一组 C 源文件提供的，因此作为一名称职的 C 程序员是使用 FreeRTOS 的先决条件，因此本章假定读者熟悉以下概念：



- 如何构建C项目，包括不同的编译和链接阶段。
- 栈和堆是什么.

- 标准 C 库 malloc() 和 free() 函数。



### 动态内存分配及其与 FreeRTOS 的相关性

从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：本书的后续章节将介绍内核对象，如任务，队列，信号量和事件组。 为了使 FreeRTOS 尽可能易于使用，这些内核对象在编译时不是分配的，而是在运行时动态分配的; 每次创建内核对象时，FreeRTOS 都会分配 RAM，并在每次删除内核对象时释放 RAM。 此策略减少了设计和计划工作，简化了 API，并最大限度地减少了 RAM 占用空间。

本章讨论动态内存分配。 动态内存分配是一种 C 编程概念，而不是 FreeRTOS 或多任务特有的概念。 它与 FreeRTOS 相关，因为内核对象是动态分配的，而通用编译器提供的动态内存分配方案并不总是适用于实时应用程序。

可以使用标准 C 库中的 malloc() 和 free() 函数分配内存，但是由于以下一个或多个原因，它们可能不合适：

- 它们在小型嵌入式系统中并不总是可用。
- 它们的实现可能相对较大，占用了宝贵的代码空间。

- 它们很少是线程安全的。
- 它们不是确定性的; 执行功能所花费的时间因调用而异。

- 它们可能会遭受碎片的折磨。
- 它们会使链接器配置复杂化。

- 如果允许堆空间增长到其他变量使用的内存，它们可能是难以调试出来的错误来源。



### 动态内存分配的选项

从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：早期版本的FreeRTOS 使用内存池分配方案，从而在编译时预先分配不同大小的内存块池，然后由内存分配功能。 虽然这是在实时系统中使用的常见方案，但它被证明是许多支持请求的来源，主要是因为它无法有效地使用 RAM 以使其适用于非常小的嵌入式系统，因此该方案被放弃。

FreeRTOS 现在将内存分配视为可移植层的一部分（而不是核心代码库的一部分）。 这是因为不同的嵌入式系统具有不同的动态存储器分配和时序要求，因此单个动态存储器分配算法将仅适用于应用的子集。 此外，从核心代码库中删除动态内存分配使应用程序编写者能够在适当时提供自己的特定实现。

当 FreeRTOS 需要 RAM 时，它调用 pvPortMalloc()，而不是调用 malloc()。 当释放 RAM 时，内核调用 vPortFree() 而不是调用 free()。 pvPortMalloc() 与标准 C 库 malloc() 函数具有相同的原型，而 vPortFree() 与标准 C 库 free() 函数具有相同的原型。

pvPortMalloc() 和 vPortFree() 是公共函数，因此也可以从应用程序代码中调用它们。

从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：FreeRTOS 带有pvPortMalloc() 和 vPortFree() 的五个示例实现，所有这些都在本章中记录。FreeRTOS 应用程序可以使用其中一个示例实现，或提供自己的。

这五个示例分别在 heap_1.c，heap_2.c，heap_3.c，heap_4.c 和 heap_5.c 源文件中定义，所有这些文件都位于 FreeRTOS/Source/portable/MemMang 目录中。

### 范围

本章的目的是让读者更好地理解：



- 什么时候 FreeRTOS 分配RAM。
- FreeRTOS 提供的 5 个内存分配方案示例。

- 选择哪种内存分配方案。

## 内存分配方案示例

从 FreeRTOS V9.0.0 可以完全静态分配 FreeRTOS 应用程序，无需包含堆内存管理器。

### Heap_1

小型专用嵌入式系统通常只在调度程序启动之前创建任务和其他内核对象。 在这种情况下，内存仅在应用程序开始执行任何实时功能之前由内核动态分配，并且内存在应用程序的生命周期内保持分配。 这意味着所选择的分配方案不必考虑任何更复杂的内存分配问题，例如确定性和分段，而只需考虑代码大小和简单性等属性。

heap_1.c 实现了 pvPortMalloc() 的一个非常基本的版本，并没有实现 vPortFree()。 永远不会删除任务或其他内核对象的应用程序可能会使用 heap_1。

一些禁止使用动态内存分配的商业关键系统和安全关键系统也可能使用 heap_1。由于与非确定性、内存碎片和失败的分配相关的不确定性，关键系统通常禁止动态内存分配，但是heap_1总是确定性的，并且不能分割内存。

heap_1 分配方案将一个简单数组细分为更小的块，因为调用了 pvPortMalloc() 。 该数组称为FreeRTOS heap。

数组的总大小（以字节为单位）由 FreeRTOSConfig.h 中的 configTOTAL_HEAP_SIZE 定义设置。 以这种方式定义大型数组可能会使应用程序看起来消耗大量 RAM ，甚至在从数组分配任何内存之前。

每个创建的任务都需要一个任务控制块（TCB）和一个从堆中分配的栈。 图 5 演示了在创建任务时 heap_1 如何细分简单数组。

参考图 5：

- A 显示了在创建任何任务之前的数组：整个数组是空闲的。
- B 显示了在创建一个任务之后的数组。

- C 显示了子啊船舰了三个任务之后的数组。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600462-f47eedd8-7502-46fd-837d-bb8435e9b9a4.png)

图5. 每次创建任务时从 heap_1 阵列分配的 RAM

### Heap_2

heap_2 保留在 FreeRTOS 发行版中以实现向后兼容，但不建议将其用于新设计。 请优先考虑使用 heap_4 而不是heap_2，因为 heap_4 提供了增强的功能。

heap_2.c 也可以通过细分由 configTOTAL_HEAP_SIZE 标注的数组来工作。 它使用最合适的算法来分配内存，与 heap_1 不同，它允许释放内存。 同样，数组是静态声明的，因此即使在分配了阵列中的任何内存之前，也会使应用程序看起来消耗大量 RAM。

最合适的算法确保 pvPortMalloc() 使用与所请求的字节数最接近的空闲内存块。 例如，考虑场景：

- 堆包含三个可用内存块，分别为 5 个字节，25 个字节和 100 个字节。
- 调用 pvPortMalloc() 来请求 20 个字节的 RAM。

所请求的字节数适合的最小空闲 RAM 块是 25 字节块，因此 pvPortMalloc() 将 25 字节块拆分为一个 20 字节的块和一个 5 字节的块，然后返回指针 20 字节的块。 新的 5 字节块仍可用于将来调用 pvPortMalloc()。

与 heap_4 不同，heap_2 不会将相邻的空闲块组合成一个更大的块，因此更容易出现碎片。 但是，如果分配并随后释放的块总是大小相同，则碎片不是问题。 heap_2 适用于重复创建和删除任务的应用程序，前提是分配给创建的任务的堆栈大小不会更改。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600757-8b0c9bed-ef35-4132-917b-1b2b2610d5c6.png)

图 6. 在创建和删除任务时，正在分配和释放来自 heap_2 数组的 RAM

图 6 展示了创建，删除然后再创建任务时最佳拟合算法的工作原理。 参考图6：



1. A 显示了创建了三个任务之后的数组。一个大的自由块仍在数组顶部。
2. B 显示了一个任务被删除之后的数组。数组顶部的大型自由块仍然存在。 现在还有两个较小的空闲块，这是先前已分配给 TCB 和栈的任务，而现在已被删除。

1. C 显示了在创建另一个任务后的情况。 创建任务导致两次调用 pvPortMalloc()，一次分配新 TCB，另一次分配任务堆栈。使用 xTaskCreate() API 函数创建任务，如 3.4 节所述。 对 pvPortMalloc() 的调用在 xTaskCreate()内部发生。每个 TCB 的大小完全相同，因此最合适的算法确保先前分配给已删除任务的 TCB 的 RAM 块被重用以分配新任务的 TCB。分配给新创建的任务的堆栈大小与分配给先前删除的相同，因此最合适的算法确保先前分配给已删除任务堆栈的 RAM 块被重用以分配新任务的堆栈。数组顶部较大的未分配块保持不变。

heap_2 不是确定性的，但比 malloc() 和 free() 的大多数标准库实现更快。

### Heap_3

heap_3.c 使用标准库 malloc() 和 free() 函数，因此堆的大小由链接器配置定义，而 configTOTAL_HEAP_SIZE 设置没有影响。heap_3 使 malloc() 和 free() 线程安全暂时挂起 FreeRTOS 调度程序。 线程安全和调度程序暂停都是第 7 章资源管理中介绍的主题。

### Heap_4

与 heap_1 和 heap_2 类似，通过将数组细分为较小的块来实现 heap_4。 和以前一样，数组是静态声明的，并且由 configTOTAL_HEAP_SIZE 标注，因此即使在实际从数组中分配任何内存之前，也会使应用程序看起来消耗大量 RAM。

heap_4 使用首次拟合算法来分配内存。 与 heap_2 不同，heap_4 将相邻的空闲内存块（合并）组合成一个更大的块，从而最大限度地降低了内存碎片的风险。

第一适合算法确保 pvPortMalloc()使用足够大的第一个空闲内存块来保存请求的字节数。 例如，考虑以下情况：

- 堆包含三个可用内存块，它们按照它们在数组中出现的顺序分别为 5 个字节，200 个字节和 100 个字节。
- 调用 pvPortMalloc() 来请求 20 个字节的 RAM。

请求的字节数适合的第一个RAM块是 200 字节块，因此 pvPortMalloc() 将 200 字节块拆分为一个 20 字节的块，以及一个 180 字节的块，然后返回指向 20 的字节块。 新的 180 字节块仍可用于将来调用 pvPortMalloc()。

heap_4 将相邻空闲块的（合并）组合成一个更大的块，最大限度地降低了碎片的风险，并使其适用于重复分配和释放不同大小的 RAM 块的应用程序。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600952-bda5943d-c3a2-410d-823b-9b6dbe693782.png)

图 7. 正在分配并从 heap_4 数组中释放的 RAM

图 7 演示了如何分配和释放具有内存合并工作的 heap_4 首次拟合算法。 参考图 7：



1. A 显示了创建三个任务后的数组。 一个大的自由块保留在数组的顶部。
2. B 显示了删除一个任务后的数组。巨大的自由块仍在数组顶部。还有一个空闲块，是先前已分配任务 TCB 和堆栈，现在已经删除。 请注意，与演示 heap_2 时不同，删除 TCB 时释放的内存以及删除堆栈时释放的内存不会保留为两个单独的空闲块，而是组合起来创建一个更大的单个空闲块。

1. C 显示创建 FreeRTOS 队列后的情况。 队列是使用 xQueueCreate() API 函数创建的，该函数在 4.3 节中描述。 xQueueCreate() 调用 pvPortMalloc() 来分配队列使用的 RAM。 由于 heap_4 使用第一适合算法，pvPortMalloc() 将从第一个空闲 RAM 块中分配 RAM，该块足够大以容纳队列，如图 7 所示，当任务被删除时，RAM 被释放。队列不会全部消耗然而，空闲块中的 RAM，因此块被分成两部分，未使用的部分仍然可用于将来调用 pvPortMalloc()。
2. D 显示了直接从应用程序代码调用 pvPortMalloc() 之后的情况，而不是通过 FreeRTOS API 函数的间接调用。 用户分配的块足够小，可以放入第一个空闲块，即分配给队列的内存和分配给后续 TCB 的内存之间的块。 删除任务时释放的内存现在已分成三个独立的块; 第一个块保存队列，第二个块保存用户分配的内存，第三个块保持空闲。

1. E 显示队列被删除后的情况，这会自动释放已分配给已删除队列的内存。现在，用户分配块的任一侧都有可用内存。
2. F 显示用户分配的内存已被释放后的情况。用户分配的块使用的内存已与任意一侧的空闲内存组合，创建一个更大的单个空闲块。



heap_4 不是具有确定性的，但比 malloc() 和 free() 的大多数标准库实现更快。

### 设置 Heap_4 使用的数组的起始地址

本节包含高级信息。若只是为了使用 heap_4，没有必要阅读或理解本节。



有时，应用程序编写者需要将 heap_4 使用的数组放在特定的内存地址。例如，FreeRTOS 任务使用的堆栈是从堆中分配的，因此可能需要确保堆位于快速的内部存储器中， 而不是缓慢的外部存储器。



默认情况下，heap_4 使用的数组在 heap_4.c 源文件中声明，其起始地址由链接器自动设置。 但是，如果在 FreeRTOSConfig.h 中将 configAPPLICATION_ALLOCATED_HEAP 编译时配置常量设置为 1，则必须由使用 FreeRTOS 的应用程序声明该数组。 如果数组声明为应用程序的一部分，则应用程序的编写者可以设置其起始地址。



如果在 FreeRTOSConfig.h 中将 configAPPLICATION_ALLOCATED_HEAP 设置为 1，则必须在其中一个应用程序的源文件中声明一个名为 ucHeap 的 uint8_t 数组，其大小为 configTOTAL_HEAP_SIZE 设置。



将变量放在特定内存地址所需的语法取决于所使用的编译器，因此请参阅编译器的文档。 两个编译器的示例如下：



- 清单 2 显示 GCC 编译器声明数组所需的语法，并将数组放在名为 .my_heap 的内存部分中。
- 清单 3 显示 IAR 编译器声明数组所需的语法，并将数组放在绝对地址 0x20000000 上 。



```c
uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] __attribute__ ( ( section( ".my_heap" ) ) );
```

清单 2. 使用 GCC 语法声明 heap_4 使用的数组，并将数组放在名为 .my_heap 的内存段中

```csharp
uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] @ 0x20000000;
```

清单 3. 使用 IAR 语法声明 heap_4 将使用的数组，并将数组放在绝对地址 0x20000000。

### Heap_5

heap_5 用于分配和释放内存的算法与 heap_4 使用的算法相同。 与 heap_4 不同的是，heap_5 不限于从单个静态声明的数组中分配内存; heap_5 可以从多个独立的内存空间分配内存。 当运行 FreeRTOS 的系统提供的 RAM 在系统的内存映射中不显示为单个连续（无空间）块时，heap_5 非常有用。



在编写本文时，heap_5 是唯一提供的内存分配方案，必须在调用 pvPortMalloc() 之前显式初始化。 使用 vPortDefineHeapRegions() API函数初始化 Heap_5。 当使用 heap_5 时，必须先调用 vPortDefineHeapRegions()，然后才能创建任何内核对象（任务，队列，信号量等）。

### vPortDefineHeapRegions() API函数

vPortDefineHeapRegions() 用于指定每个单独的内存区域的起始地址和大小，它们共同构成 heap_5 使用的总内存。

```csharp
void vPortDefineHeapRegions(const HeapRegion_t *const pxHeapRegions );
```

清单 4. vPortDefineHeapRegions() API 函数原型

每个单独的存储区域由 HeapRegion_t 类型的结构描述。 所有可用内存区域的描述都作为 HeapRegion_t 结构数组传递给 vPortDefineHeapRegions()。



```verilog
typedef struct HeapRegion
{
    /* 将成为堆一部分的内存块的起始地址。 */
    uint8_t *pucStartAddress;

    /* 内存块的大小（字节）。 */
    size_t xSizeInBytes;
} HeapRegion_t;
```

清单 5. HeapRegion_t 结构

表 5. vPortDefineHeapRegions() 参数

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| pxHeapRegions   | 指向 HeapRegion_t 结构数组开头的指针。 数组中的每个结构都描述了使用 heap_5 时将成为堆的一部分的内存区域的起始地址和长度。数组中的 HeapRegion_t 结构必须按起始地址排序; 描述具有最低起始地址的存储区域的 HeapRegion_t 结构必须是数组中的第一个结构，并且描述具有最高起始地址的存储区域的 HeapRegion_t 结构必须是数组中的最后一个结构。数组的末尾由 HeapRegion_t 结构标记，该结构的 pucStartAddress 成员设置为 NULL。 |



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600926-a8612b64-b6cf-48eb-85cf-c66cbe5404b0.png)

图 8. 内存映射

清单 6 显示了一个 HeapRegion_t 结构数组，它们共同描述了三个 RAM 块。



```csharp
/* 定义三个RAM区域的起始地址和大小。 */
#define RAM1_START_ADDRESS( ( uint8_t * ) 0x00010000 )
#define RAM1_SIZE( 65 * 1024 )

#define RAM2_START_ADDRESS( ( uint8_t * ) 0x00020000 )
#define RAM2_SIZE( 32 * 1024 )

#define RAM3_START_ADDRESS( ( uint8_t * ) 0x00030000 )
#define RAM3_SIZE( 32 * 1024 )

/* 创建一个 HeapRegion_t 定义数组，其中包含三个 RAM 区域的每个索引，并用空地址结束数组。
HeapRegion_t 结构必须按开始地址顺序出现，包含最低开始地址的结构首先出现。*/
const HeapRegion_t xHeapRegions[] =
{
    { RAM1_START_ADDRESS, RAM1_SIZE },
    { RAM2_START_ADDRESS, RAM2_SIZE },
    { RAM3_START_ADDRESS, RAM3_SIZE },
    { NULL,               0         }  /* 标记数组的结尾。 */
};

int main( void )
{
    /* 初始化 heap_5。 */
    vPortDefineHeapRegions( xHeapRegions );

    /* 在这里添加应用代码。 */
}
```



清单 6. 一个 HeapRegion_t 结构数组，它们共同描述了 RAM 的 3 个区域



虽然清单 6 正确描述了 RAM，但它没有演示一个可用的示例，因为它将所有 RAM 分配给堆，而没有 RAM 可供其他变量使用。



构建项目时，构建过程的链接阶段会为每个变量分配一个 RAM 地址。 可供链接器使用的 RAM 通常由链接器配置文件（如链接描述文件）描述。 在图 8 B 中，假设链接器脚本包含有关 RAM1 的信息，但不包括有关 RAM2 或 RAM3 的信息。 因此，链接器在 RAM1 中放置了变量，只留下了地址为 0x0001nnnn 的 RAM1 部分可供 heap_5 使用。 0x0001nnnn 的实际值将取决于所链接的应用程序中包含的所有变量的总大小。 链接器使所有 RAM2 和所有 RAM3 都未使用，使得整个 RAM2 和整个 RAM3 可供 heap_5 使用。

如果使用清单 6 中所示的代码，则分配给地址 0x0001nnnn下面的 heap_5 的 RAM 将与用于保存变量的 RAM 重叠。 为避免这种情况，xHeapRegions[] 数组中的第一个 HeapRegion_t 结构可以使用起始地址0x0001nnnn，而不是起始地址 0x00010000。 但是，这不是推荐的解决方案，因为：



1. 起始地址可能不容易确定。
2. 链接器使用的 RAM 量可能会在将来的版本中发生变化，需要更新 HeapRegion_t结构中使用的起始地址。

1. 构建工具未知， 因此，如果链接器使用的 RAM 和 heap_5 使用的 RAM 重叠，则无法警告应用程序编写者。



清单 7 演示了一个更方便和可维护的示例。 它声明了一个名为 ucHeap 的数组。 ucHeap 是一个普通变量，因此它成为链接器分配给 RAM1 的数据的一部分。 xHeapRegions 数组中的第一个 HeapRegion_t 结构描述了 ucHeap 的起始地址和大小，因此 ucHeap 成为 heap_5 管理的内存的一部分。 可以增加ucHeap 的大小，直到链接器使用的 RAM 消耗所有 RAM1，如图 8 C 所示。



```csharp
/* 定义链接器未使用的两个RAM区域的起始地址和大小。 */
#define RAM2_START_ADDRESS( ( uint8_t * ) 0x00020000 )
#define RAM2_SIZE( 32 * 1024 )

#define RAM3_START_ADDRESS( ( uint8_t * ) 0x00030000 )
#define RAM3_SIZE( 32 * 1024 )

/* 声明一个数组，该数组将成为 heap_5 使用的堆的一部分。 该数组将由链接器放置在 RAM1 中。 */
#define RAM1_HEAP_SIZE ( 30 * 1024 )
static uint8_t ucHeap[ RAM1_HEAP_SIZE ];

/* 创建一个 HeapRegion_t 数组。 而在清单 6 中，第一个条目描述了所有 RAM1，因此 heap_5 
将使用所有 RAM1，这次第一个条目仅描述 ucHeap 数组，因此 heap_5 将仅使用包含 ucHeap 
数组的 RAM1 部分。 HeapRegion_t 结构仍必须以起始地址顺序出现，其结构包含最先出现的最低起
始地址。*/
const HeapRegion_t xHeapRegions[] =
{
    { ucHeap,             RAM1_HEAP_SIZE },
    { RAM2_START_ADDRESS, RAM2_SIZE },
    { RAM3_START_ADDRESS, RAM3_SIZE },
    { NULL,               0         }  /* 标记数组的结尾。 */
};
```



清单 7. 一个 HeapRegion_t 结构数组，描述所有 RAM2，所有 RAM3，但只是 RAM1 的一部分



清单7中演示的技术的优点包括：

1. 没有必要使用硬编码的起始地址。
2. HeapRegion_t 结构中使用的地址将由链接器自动设置，因此即使链接器使用的 RAM 量在将来的构建中发生更改，也始终是正确的。

1. 分配给 heap_5 的 RAM 不可能与链接器放入 RAM1 的数据重叠。
2. 如果 ucHeap 太大，应用程序将不会链接。



## 堆相关的实用函数

### xPortGetFreeHeapSize() API函数

xPortGetFreeHeapSize() API 函数在调用函数时返回堆中的空闲字节数。它可用于优化堆大小。 例如，如果 xPortGetFreeHeapSize() 在创建了所有内核对象后返回 2000，那么 configTOTAL_HEAP_SIZE 的值可以减少 2000。



使用 heap_3 时，xPortGetFreeHeapSize()不可用。

```dart
size_t xPortGetFreeHeapSize( void );
```

清单 8. xPortGetFreeHeapSize() API 函数原型



表 6. xPortGetFreeHeapSize() 返回值

| 参数名称/返回值 | 描述                                                     |
| --------------- | -------------------------------------------------------- |
| 返回值          | 调用 xPortGetFreeHeapSize() 时在堆中保持未分配的字节数。 |

### xPortGetMinimumEverFreeHeapSize() API函数

xPortGetMinimumEverFreeHeapSize() API 函数返回自 FreeRTOS 应用程序开始执行以来堆中曾存在的最小未分配字节数。

xPortGetMinimumEverFreeHeapSize() 返回的值表明应用程序已经接近耗尽堆空间。例如，如果 xPortGetMinimumEverFreeHeapSize() 返回200，那么，在应用程序开始执行后的某个时候，它会在耗尽堆空间的 200 字节之内。

xPortGetMinimumEverFreeHeapSize() 仅在使用 heap_4 或 heap_5 时可用。

```dart
size_t xPortGetMinimumEverFreeHeapSize( void );
```

清单 9. xPortGetMinimumEverFreeHeapSize() API 函数原型



表 7. xPortGetMinimumEverFreeHeapSize()返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| 返回值          | 自 FreeRTOS 应用程序开始执行以来堆中已存在的最小未分配字节数。 |

### malloc 失败的钩子函数

可以直接从应用程序代码调用 pvPortMalloc()。 每次创建内核对象时，它也会在 FreeRTOS 源文件中调用。内核对象的示例包括任务，队列，信号量和事件组，所有这些都将在本书后面的章节中介绍。



就像标准库 malloc() 函数一样，如果 pvPortMalloc() 由于所请求大小的块不存在而无法返回 RAM 块，那么它将返回 NULL。 如果由于应用程序编写器正在创建一个内核对象而执行 pvPortMalloc()，并且对 pvPortMalloc() 的调用返回 NULL，则内核对象不会被创建。



如果对 pvPortMalloc() 的调用返回 NULL，则可以将所有示例堆配置方案配置为调用钩子（或回调）函数。



如果在 FreeRTOSConfig.h 中将 configUSE_MALLOC_FAILED_HOOK 设置为 1，则应用程序必须提供 malloc 失败的钩子函数，该函数具有由清单 10 显示的名称和原型。该函数可以以适合应用程序的任何方式实现。

```dart
void vApplicationMallocFailedHook( void );
```

清单 10. malloc 失败的钩子函数名和原型。



# **任务管理**



## 引言与范围

### 范围

本章旨在让读者更好地理解：



- FreeRTOS 如何为应用程序中的每个任务分配处理时间。
- FreeRTOS 如何选择在任何给定时间执行哪个任务。

- 每个任务的相对优先级如何影响系统行为。 
- 任何可以存在的任务。



读者也应该很好地理解：



- 如何实现任务。
- 如何创建一个或多个任务实例。

- 如何使用任务参数。
- 如何更改已被创建任务的优先级。

- 如何删除任务。
- 如何使用任务实现定期处理（软件计时器将在后面的章节中讨论）。

- 空闲任务何时执行以及如何使用。



本章介绍的概念对于理解如何使用 FreeRTOS 以及 FreeRTOS 应用程序的行为至关重要。因此，这是书中最详细的一章。



## 任务函数

任务为 C 函数实现。 它们唯一的特殊之处是它们的原型，它必须返回 void 并接受一个 void 指针参数。 清单 11 展示了原型。

```dart
void ATaskFunction( void *pvParameters );
```

清单 11. 任务函数原型



每个任务本身都是一个小程序。 它有一个入口点，通常会在无限循环内永远运行，并且不会退出。 清单 12中显示了典型任务的结构。



FreeRTOS 任务不允许以任何方式从其实现的函数返回，即它们不能包含 return 语句，并且不能允许在函数结束之后执行。 如果不再需要任务，则应明确删除该任务。 这在清单 12 中也有说明。

单个任务函数，可被创建成任意数量的任务，每个创建的任务都是一个单独的执行实例，具有自己的堆栈以及在任务本身内定义的任何自动（堆栈）变量，都有自己的副本。



```dart
void ATaskFunction( void *pvParameters )
{
/* 变量可以像普通函数一样声明。使用这个示例函数创建的任务的每个实例都有自己的 lVariableExample 
变量副本。如果将变量声明为静态，则不会出现这种情况，在这种情况下，只有一个变量副本存在，并且该副本
将由任务的每个创建实例共享。(添加到变量名的前缀在第1.5节“数据类型和编码风格指南”中进行了描述。) */
int32_t lVariableExample = 0;

    /* 任务通常将实现为无限循环。 */
    for( ;; )
    {
        /* 实现任务功能的代码将在此处。 */
    }

    /* 如果任务实现突破上述循环，则必须在到达其实现功能结束之前删除该任务。 传递给 vTaskDelete() 
    API 函数的 NULL 参数指示要删除的任务是调用（当前）任务。 用于命名 API 函数的约定在第 0 节中
    描述，使用早于 V9.0.0 的 FreeRTOS 版本的项目必须构建一个 heap_n.c 文件。 从 
    FreeRTOS V9.0.0 开始，只有在 FreeRTOSConfig.h 中将 configSUPPORT_DYNAMIC_ALLOCATION
    设置为 1 或者未定义 configSUPPORT_DYNAMIC_ALLOCATION 时才需要 heap_n.c 文件。 有关更多
    信息，请参阅第 2 章堆内存管理。 数据类型和编码风格指南。 */
    vTaskDelete( NULL );
}
```

清单 12. 典型任务函数的结构



## 顶层任务状态

应用程序可以包含许多任务。 如果运行应用程序的处理器是单核的，那么在任何给定时间只能执行一个任务。 这意味着任务可以存在两种状态：运行和未运行。 首先考虑这种简单化的模型，但请记住，这是一种过度简化。 在本章的后面，显示 “未运行” 状态实际上包含许多子状态。



当任务处于运行状态时，则处理器正在执行该任务的代码。 当任务处于 “未运行” 状态时，任务处于休眠状态，其状态已准备好，以便在下次调度程序确定应进入 “运行” 状态时继续执行。 当一个任务恢复执行时，它会从它上次离开运行状态之前执行的指令执行。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635995518702-e2ed01de-83a9-45a8-a5df-fac1e30b3ba3.png)

图 9. 顶层任务状态和转换

从 “未运行” 状态转换为 “运行” 状态的任务被称为 “切入” 或 “换入”。 相反，从 “运行” 状态转换到 “未运行” 状态的任务据说已被 “切出” 或 “换出”。 FreeRTOS 的调度程序是唯一可以切换任务的实体。



## 创建任务

### xTaskCreate() API 函数

FreeRTOS V9.0.0 还包含 xTaskCreateStatic() 函数，它分配了在编译时静态创建任务所需的内存：任务是使用 FreeRTOS xTaskCreate() API 函数创建的。这可能是所有 API 函数中最复杂的一个，因此很不幸，它是第一个遇到的函数，但是必须首先掌握任务，因为它们是多任务系统中最基本的组件。本书附带的所有示例都使用了 xTaskCreate() 函数，因此有很多示例可供参考。



第 1.5 节 “数据类型和编程样式指南” 描述了所使用的数据类型和命名约定。

```dart
BaseType_t xTaskCreate( TaskFunction_t pvTaskCode, 
                        const char* const pcName, 
                        uint16_t usStackDepth, 
                        void *pvParameters, 
                        UBaseType_t uxPriority, 
                        TaskHandle_t *pxCreatedTask );
```

清单 13. xTaskCreate() API 函数原型

表 8. xTaskCreate() 参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| pvTaskCode      | 任务仅仅是从不退出的C函数，因此通常作为无限循环来实现。PvTaskCode参数只是指向实现任务的函数的指针(实际上，只是函数的名称)。 |
| pcName          | 任务的描述性名称。FreeRTOS 不会以任何方式使用它。它纯粹是作为调试辅助工具而包含的。通过人类可读的名称标识任务要比尝试通过其句柄标识任务简单得多。 应用程序定义的常量configMAX_TASK_NAME_LEN定义了任务名称可以包含的最大长度包括空终止符。提供长度超过此最大值的字符串将导致字符串被自动截断。 |
| usStackDepth    | 每个任务都有自己的唯一堆栈，该堆栈在创建任务时由内核分配给任务。 usStackDepth值告诉内核堆栈的大小。该值指定堆栈可以容纳的字数，而不是字节数。 例如，如果堆栈是 32 位宽并且 usStackDepth 作为 100 传入，则将分配 400 字节的堆栈空间（100 * 4 字节）。 堆栈深度乘以堆栈宽度不得超过 uint16_t 类型的变量中可包含的最大值。 空闲任务使用的堆栈大小由应用程序定义的常量 configMINIMAL_STACK_SIZE定义。在 FreeRTOS 演示应用程序中为正在使用的处理器体系结构分配给此常量的值是建议的最小任务。如果您的任务使用大量堆栈空间，则必须分配更大的值。 没有简单的方法来确定任务所需的堆栈空间。可以计算，但是大多数用户只需分配他们认为合理的值，然后使用FreeRTOS 提供的功能来确保分配的空间确实足够，并且RAM 不会被浪费。 第 12.3 节“堆栈溢出” 包含有关如何查询任务实际使用的最大堆栈空间的信息。 |
| pvParameters    | 任务函数接受指向 void（void *）的类型指针的参数。分配给 pvParameters 的值是传递给任务的值。本书中的一些示例演示了如何使用该参数。 |
| uxPriority      | 定义任务执行的优先级。可以将优先级从 0（最低优先级）分配给（configMAX_PRIORITIES - 1），这是最高优先级。 configMAX_PRIORITIES 是用户定义的常量，在3.5 节中描述。 传递上面的 uxPriority 值（configMAX_PRIORITIES - 1）将导致分配给任务的优先级被静默限制为最大合法值。 |
| pxCreatedTask   | pxCreatedTask 可用于传递正在创建的任务的句柄。然后，此句柄可用于引用 API 调用中的任务，例如，更改任务优先级或删除任务。 如果您的应用程序没有使用任务句柄，那么 pxCreatedTask 可以设置为 NULL。 |
| 返回值          | 有两种可能的返回值：pdPASS：这表明该任务已成功创建。pdFAIL：这表明该任务尚未创建，因为 FreeRTOS 可用的堆内存不足，无法分配足够的 RAM 来保存任务数据结构和堆栈。第 2 章提供了有关堆内存管理的更多信息。 |

### 示例 1. 创建任务

此示例演示了创建两个简单任务所需的步骤，然后启动执行的任务。 任务只是定期打印一个字符串，使用粗略的空循环来创建周期延迟。 这两个任务都以相同的优先级创建，除了打印出的字符串外，它们是相同的，参见清单 14 和清单 15 了解它们各自的实现。

```groovy
void vTask1( void *pvParameters )
{
const char *pcTaskName = "Task 1 is running\r\n";
volatile uint32_t ul; /* volatile 以确保 ul 没有优化。 */

    /* 对于大多数任务，都是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 延迟一段时间。 */
        for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
        {
            /* 这个循环只是一个非常粗略的延迟实现。 这里没什么可做的。 
            后面的例子将以适当的延迟替换这个粗略的循环 */
        }
    }
}
```

清单 14. 实现示例 1 中使用的第一个任务

```groovy
void vTask2( void *pvParameters )
{
const char *pcTaskName = "Task 2 is running\r\n";
volatile uint32_t ul; /* volatile 以确保 ul 不被优化掉。 */

    /* 对于大多数任务，都是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 延迟一段时间。 */
        for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
        {
            /* 这个循环只是一个非常粗略的延迟实现。 这里没什么可做的。 
            后面的例子将以适当的延迟替换这个粗略的循环 */
        }
    }
}
```

清单 15. 实现示例 1 中使用的第二个任务



main() 函数在启动调度程序之前创建任务，有关其实现请参见清单 16。

```groovy
int main( void )
{
    /* 创建两个任务之一。请注意，实际应用程序应该检查 xTaskCreate() 
    调用的返回值，以确保任务创建成功。 */
    xTaskCreate(    vTask1,  /* 指向实现任务的函数的指针。 */
                    "Task 1",/* 任务的文本名称。这只是为了方便调试。 */
                    1000,    /* 堆栈深度，小型微控制器将使用比这少得多的堆栈。 */
                    NULL,    /* 此示例不使用任务参数。 */
                    1,       /* 此任务将以优先级 1 运行。 */
                    NULL );  /* 此示例不使用任务句柄。 */

    /* 以完全相同的方式，以相同的优先级创建另一个任务。 */
    xTaskCreate(vTask2, "Task 2", 1000, NULL, 1, NULL );

    /* 启动调度程序，以便任务开始执行。 */
    vTaskStartScheduler();    

    /* 如果一切正常，那么 main() 将永远不会到达这里，因为调度程序将重新运行这些任务。
    如果 main() 确实到达这里，那么很可能没有足够的堆内存来创建空闲任务。第 2 章提供了
    关于堆内存管理的更多信息。 */
    for( ;; );
}
```

清单 16. 启动示例 1 任务



执行该示例将生成图 10 中所示的输出。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635996495870-7a09af02-4122-4001-ab11-55e8dd15e680.png)

图 10. 执行示例 1 时生成的输出

图 10 显示了两个似乎同时执行的任务; 但是，由于任务正在同一个处理器核上执行，但情况并非如此。 实际上，这两项任务都在迅速进入并退出运行状态。 这两个任务都以相同的优先级运行，因此在同一个处理器上运行共享时间。 实际执行模式如图 11 所示。



图 11 底部的箭头表示从时间 t1 开始经过的时间。 彩色线表示在每个时间点正在执行哪个任务，例如，Task1 在时间 t1 和时间 t2 之间执行。



任何时候只有一个任务可以处于运行状态。 因此，当一个任务进入运行状态（任务被切入）时，另一个任务进入非运行状态（任务被切出）。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635996495578-50f6ccc4-f552-4309-8f1c-3e3845833ba9.png)

图 11.两个示例 1 任务的实际执行模式

示例 1 在启动调度程序之前，从 main() 内创建了两个任务。 从另一个任务中创建任务也是可能的。 例如，可以从 Task1 中创建 Task2，如清单 17 所示。

```groovy
void vTask1( void *pvParameters )
{
const char *pcTaskName = "Task 1 is running\r\n";
volatile uint32_t ul; /* volatile 确保 ul 没有被优化掉。 */

    /* 如果正在执行此任务代码，则必须已经启动了调度程序。在进入无限循环之前创建另一个任务。 */
    xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, NULL );

    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 延迟一段时间。 */
        for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
        {
            /* 这个循环只是一个非常粗略的延迟实现。 这里没什么可做的。 
            后面的例子将以适当的延迟替换这个粗略的循环 */
        }
    }
}
```

清单 17. 在调度程序启动后从另一个任务中创建任务

### 示例 2. 使用任务参数

在示例 1 中创建的两个任务是字面上的，它们之间的唯一区别是它们打印出来的文本字符串。 相反，可以通过创建单个任务实现的两个实例来删除此重复。 然后可以使用任务参数将要打印出的字符串传递给每个任务。



清单 18 包含示例 2 使用的单任务函数（vTaskFunction）的代码。此单个函数替换示例 1 中使用的两个任务函数（vTask1 和 vTask2）。注意如何将任务参数强制转换为 char * 以获取字符串任务 应打印出来。

```groovy
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
volatile uint32_t ul; /* volatile 以确保 ul 不被优化掉。 */

    /* 要打印的字符串通过参数传入。将此转换为字符指针。 */
    pcTaskName = ( char * ) pvParameters;

    /* 对于大多数任务，都是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 延迟一段时间。 */
        for( ul = 0; ul < mainDELAY_LOOP_COUNT; ul++ )
        {
            /* 这个循环只是一个非常粗略的延迟实现。 这里没什么可做的。 
            后来的练习将以适当的延迟取代这个原始循环。 */
        }
    }
}
```

清单 18. 用于在示例 2 中创建两个任务的单个任务函数



即使现在只有一个任务实现（vTaskFunction），也可以创建多个已定义任务的实例。每个创建的实例将在 FreeRTOS 调度程序的控制下独立执行。



清单 19 显示了如何使用 xTaskCreate() 函数的 pvParameters 参数将文本字符串传递给任务。

```groovy
/* 定义将作为任务参数传入的字符串。它们定义为 const，不在堆栈上，以确保在执行任务时它们
仍然有效。 */
static const char *pcTextForTask1 = "Task 1 is running\r\n";
static const char *pcTextForTask2 = "Task 2 is running\r\n";

int main( void )
{
    /* 创建两个任务中的第一个。 */
    xTaskCreate(    vTaskFunction,         /* 指向实现该任务的函数的指针。 */
                    "Task 1",              /* 任务的文本名称。这只是为了方便调试。 */
                    1000,                  /* 堆栈深度 - 小型微控制器将使用比这少得多的堆栈。 */
                    (void*)pcTextForTask1, /* 使用任务参数将要打印的文本传递到任务中。 */
                    1,                     /* 这个任务将以优先级 1 运行。 */
                    NULL );                /* 本例中没有使用任务句柄。 */

    /* 用完全相同的方法创建另一个任务。注意，这一次将从同一个任务实现 (vTaskFunction) 
    创建多个任务。只有参数中传递的值不同。正在创建同一任务的两个实例。 */
    xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 1, NULL );

    /* 启动调度程序，使任务开始执行。 */
    vTaskStartScheduler();    

    /* 如果一切正常，那么 main() 将永远不会到达这里，因为调度程序现在将运行这些任务。
    如果 main() 确实到达这里，那么很可能没有足够的堆内存来创建空闲任务。第 2 章提供了关于堆
    内存管理的更多信息。 */
    for( ;; );
}
```

清单 19. 示例 2 说使用的 main() 函数



示例 2 的输出完全与图 10 中的示例 1 所示的输出完全相同。



## 任务优先级



xTaskCreate() API 函数的 uxPriority 参数为正在创建的任务分配初始优先级。 使用vTaskPrioritySet() API 函数可以在启动调度程序后，更改优先级。



可用优先级的最大数量由应用程序设置，在 FreeRTOSConfig.h 中定义的 configMAX_PRIORITIES 编译时配置常量。 低数字优先级值表示低优先级任务，优先级 0 是可能的最低优先级。 因此，可用优先级的范围是 0 到（configMAX_PRIORITIES - 1）。任意数量的任务可以共享相同的优先级，确保最大的设计灵活性。



FreeRTOS 调度程序可以使用以下两种方法之一来确定哪个任务将处于运行状态。 可以设置 configMAX_PRIORITIES 的最大值取决于使用的方法：



1. 通用方法：通用方法在 C 中实现，可以与所有 FreeRTOS 架构移植一起使用。使用通用方法时，FreeRTOS 不会限制 configMAX_PRIORITIES 可以设置的最大值。 但是，始终建议将 configMAX_PRIORITIES 值保持在最小值，因为值越高，消耗的 RAM 越多，最坏情况下的执行时间就越长。如果在 FreeRTOSConfig.h 中将 configUSE_PORT_OPTIMISED_TASK_SELECTION 设置为 0，或者如果未定义 configUSE_PORT_OPTIMISED_TASK_SELECTION，或者如果通用方法是为使用的 FreeRTOS 移植提供的唯一方法，则将使用通用方法。
2. 架构优化方法：体系结构优化方法使用少量汇编代码，并且比通用方法更快。 configMAX_PRIORITIES 设置不会影响最坏情况执行时间。如果使用体系结构优化方法，那么configMAX_PRIORITIES 不能大于 32。与通用方法一样，建议将 configMAX_PRIORITIES 保持在必要的最小值，因为它的值越高，消耗的 RAM 就越多。如果在 FreeRTOSConfig.h 中将 configUSE_PORT_OPTIMISED_TASK_SELECTION 设置为 1，则将使用体系结构优化方法。并非所有 FreeRTOS 移植都提供了架构优化方法。



FreeRTOS 调度程序将始终确保能够运行的最高优先级任务是选择进入运行状态的任务。 如果能够运行多个具有相同优先级的任务，则调度程序将依次将每个任务转换为运行状态和退出运行状态。



## 时间测量和嘀嗒中断

第 3.12 节 “调度算法” 描述了一个名为 “时间切片” 的可选功能。 在目前给出的示例中使用了时间切片，它是在输出中观察到的行为。在示例中，两个任务都以相同的优先级创建，并且两个任务始终能够运行。 因此，每个任务为 “时间片” 执行，在时间片开始时进入运行状态，并在时间片结束时退出运行状态。 在图 11 中，t1 和 t2 之间的时间等于单个时间片。



为了能够选择要运行的下一个任务，调度程序本身必须在每个时间片的末尾执行。 称为 “滴答中断” 的周期性中断用于此目的。 时间片的长度由滴答中断频率有效设置，滴答中断频率由应用程序在 FreeRTOSConfig.h 中定义的 configTICK_RATE_HZ 编译时配置常量配置。 例如，如果 configTICK_RATE_HZ 设置为 100（Hz），则时间片将为 10 毫秒。 两次滴答中断之间的时间称为 “滴答期”。 一个时间片等于一个滴答时间段。



图 11 可以展开来显示调度程序本身在执行序列中的执行情况。图 12 显示了这一点，其中顶部一行显示了调度程序何时执行，细箭头显示了从任务到滴答中断的执行顺序，然后从滴答中断返回到另一个任务。

configTICK_RATE_HZ 的最佳值取决于正在开发的应用程序，但值通常为 100。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635997524402-12ef733f-1066-4c7d-a98d-cf8ea78a2593.png)

图 12. 执行序列展开以显示滴答中断执行

FreeRTOS API 调用总是以滴答周期的倍数指定时间，这通常被简单地称为滴答。pdMS_TO_TICKS() 宏将以毫秒为单位的时间转换为以节拍为单位的时间。可用的分辨率取决于所定义的滴答频率，如果滴答频率高于 1 KHz (configTICK RATE HZ 大于 1000)，则不能使用 pdMS_TO_TICKS()。清单 20 显示了如何使用 pdMS_TO_TICKS() 将指定为 200 毫秒的时间转换为以滴答为单位指定的等效时间。

```groovy
/* pdMS_TO_TICKS() 将以毫秒为单位的时间作为其唯一参数，并计算滴答周期中的等效时间。 
此示例显示 xTimeInTicks 设置为等于 200 毫秒的滴答周期数。 */
TickType_t xTimeInTicks = pdMS_TO_TICKS( 200 );
```

清单 20. 使用pdMS TO TICKS()宏将200毫秒转换为滴答周期中的等效时间



不建议在应用程序中直接以滴答为单位指定时间，而是使用 pdMS_TO_TICKS()宏来指定以毫秒为单位的时间，这样做可以确保如果滴答频率发生变化，应用程序中指定的时间不会发生变化。



滴答计数值是调度器启动以来发生的滴答中断总数，假设滴答计数没有溢出。在指定延迟周期时，用户应用程序不必考虑溢出，因为时间一致性是由 FreeRTOS 在内部管理的。



第 3.12 节 “调度算法” 描述了影响调度程序何时选择要运行的新任务以及何时执行勾选中断的配置常量。



### 示例 3. 使用优先级进行实验

调度程序将始终确保能够运行的最高优先级任务是选择进入 “运行” 状态的任务。 在我们到目前为止的示例中，已经以相同的优先级创建了两个任务，因此他们是依次进入和退出运行状态。 此示例查看在更改示例 2 中创建的两个任务之一时的优先级。 这次，第一个任务将以优先级 1 创建，第二个任务将以优先级 2 创建。创建任务的代码如清单 21 所示。实现这两个任务的单个函数没有改变。 它仍然只是定期打印出一个字符串，使用空循环来创建延迟。

```groovy
/* 定义将作为任务参数传入的字符串。 这些是定义的 const 而不是堆栈，以确保它们
在任务执行时保持有效。 */
static const char *pcTextForTask1 = "Task 1 is running\r\n";
static const char *pcTextForTask2 = "Task 2 is running\r\n";

int main( void )
{
    /* 创建优先级为 1 的第一个任务。优先级为倒数第二个参数。 */
    xTaskCreate( vTaskFunction, "Task 1", 1000, (void*)pcTextForTask1, 1, NULL );

    /* 创建优先级为 2 的第二个任务，优先级高于优先级 1。优先级是倒数第二个参数。 */
    xTaskCreate( vTaskFunction, "Task 2", 1000, (void*)pcTextForTask2, 2, NULL );

    /* 启动调度程序，以便任务开始执行。 */
    vTaskStartScheduler();    

    /* 不会到达这里。 */
    return 0;
}
```

清单 21. 以不同的优先级创建两个任务



示例 3 产生的输出如图 13 所示。 调度程序将始终选择能够运行的最高优先级任务。 任务 2 的优先级高于任务 1，并且始终能够运行；因此，任务 2 是唯一进入运行状态的任务。 若任务 1 从未进入运行状态，它永远不会打印出它的字符串。 任务 1 被称为任务 2 的处理时间 “缺乏”。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998116184-6714d06a-2dcc-4cef-8c0e-a5d6d5745263.png)

图 13. 以不同的优先级运行两个任务

任务 2 总是能够运行，因为它永远不必等待任何事情 —— 它要么循环一个空循环，要么打印到终端。



图 14 显示了示例 3 的执行顺序。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998115879-33726541-8bb3-4b85-934b-c698858aaee9.png)

图 14. 当一个任务的优先级高于另一个任务时的执行模式



## 展开说 “非运行” 状态

到目前为止，创建的任务总是有执行处理，从来没有等待任何事情，因为他们永远不必等待任何事情，他们总是能够进入运行状态。 这种类型的 “连续处理” 任务的有用性有限，因为它们只能以最低优先级创建。 如果它们以任何其他优先级运行，它们将阻止具有较低优先级的任务一直运行。



为了使任务有用，必须重写它们以使其成为事件驱动的。 事件驱动的任务只有在触发它的事件发生后才能执行工作（处理），并且在该事件发生之前无法进入运行状态。 调度程序始终选择能够运行的最高优先级任务。 无法运行的高优先级任务意味着调度程序无法选择它们，而必须选择能够运行的优先级较低的任务。 因此，使用事件驱动任务意味着可以在不同的优先级上创建任务，而不需要最高优先级的任务占用所有较低优先级的任务的处理时间。

### 阻塞状态

正在等待事件的任务被称为处于阻塞状态，这是未运行状态的子状态。



任务可以进入阻塞状态，等待两种不同类型的事件：



1. 时间（与时间相关）事件，即该事件是延时期到期或达到绝对时间。 例如，任务可以进入阻塞状态以等待10毫秒再通过。
2. 同步事件，即该事件源自另一个任务或中断。 例如，任务可以进入阻塞状态以等待数据到达队列。 同步事件涉及广泛的事件类型。



FreeRTOS 的队列，二进制信号量，计数信号量，互斥量，递归互斥体，事件组和直接到任务通知都可用于创建同步事件。 所有这些功能都将在本书的后续章节中介绍。



任务可以在阻塞时设置带有超时的同步事件，同时有效地阻塞两种类型的事件。 例如，任务可以选择等待最多 10 毫秒的数据到达队列。 如果数据在 10 毫秒内到达，或者在没有数据到达的情况下传递 10 毫秒，任务都将离开阻塞状态。



### 挂起状态

“挂起” 也是非运行状态的子状态。 处于挂起状态的任务对调度程序时不可用的。 进入挂起状态的唯一方法是通过调用 vTaskSuspend() API函数，调用 vTaskResume() 或 xTaskResumeFromISR() API 函数时退出挂起的唯一出路。 大多数应用程序不使用挂起状态。



### 就绪状态

处于 “未运行” 状态但未被阻塞或挂起的任务被称为处于 “就绪” 状态。 它们能够运行，因此 “准备好” 运行，但当前不处于 “运行” 状态。



### 完成状态转换图

图 15 展开了先前过度简化的状态图，以包括本节中描述的所有非运行状态的子状态。 到目前为止，在示例中创建的任务没有使用阻塞或挂起状态；它们只在就绪状态和运行状态之间转换，由图 15 中的粗线突出显示。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998355516-0d90c553-da06-4c6c-80b1-b80e537ddfdd.png)

图 15. 完整的任务状态机



### 示例 4. 使用 “阻止” 状态创建延迟

到目前为止，在示例中创建的所有任务都是 “周期性的”，它们已经延迟了一段时间并打印出它们的字符串，然后再次延迟，依此类推。使用空循环非常粗略地生成延迟，该任务有效地轮询递增循环计数器，直到它达到固定值。实施例 3 清楚地证明了该方法的缺点。优先级较高的任务在执行空循环时保持运行状态，“饿死” 任何处理时间的优先级较低的任务。



任何形式的轮询还有其他几个缺点，其中最重要的是它的效率低下。在轮询期间，该任务实际上没有任何工作要做，但它仍然使用最长的处理时间，因此浪费了处理器的资源。示例 4 通过调用 vTaskDelay() API 函数替换轮询空循环来纠正此行为，其原型如清单 22 所示。新的任务定义如清单 23 所示。请注意，vTaskDelay() API 函数是仅当 FreeRTOSConfig.h 中的 INCLUDE_vTaskDelay 设置为 1 时才可用。



vTaskDelay() 将调用任务置于阻塞状态，以获得固定数量的滴答中断。任务在处于阻塞状态时不使用任何处理时间，因此任务仅在实际工作时使用处理时间。

```groovy
void vTaskDelay( TickType_txTicksToDelay );
```

清单 22. vTaskDelay() 的函数原型

表 9. vTaskDelay() 参数

| 参数名称      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| xTicksToDelay | 调用任务在转换回就绪状态之前将保持在阻塞状态的滴答中断数。例如，如果一个名为 vTaskDelay(100) 的任务在滴答计数为 10,000 时立即进入阻塞状态，并保持阻塞状态，直到滴答计数达到 10,100。可以使用宏 pdMS_TO_TICKS() 将以毫秒为单位的时间转换为以 int 为单位的时间。例如，调用 vTaskDelay(pdMS_TO_TICKS(100)) 将导致调用任务保持阻塞状态 100 毫秒。 |

```groovy
void vTaskFunction( void *pvParameters ) 
{ 
char *pcTaskName; 
const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );

    /* 要打印的字符串通过参数传入。将此转换为字符指针。 */
    pcTaskName = ( char * ) pvParameters;

    /* 对于大多数任务，都是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 延迟一段时间。 这次调用 vTaskDelay() 会将任务置于阻塞状态，直到延迟时间结束。
        参数需要 'ticks' 中指定的时间，并使用 pdMS_TO_TICKS() 宏（声明 xDelay250ms 
        常量）将 250 毫秒转换为滴答的等效时间。*/
        vTaskDelay( xDelay250ms );
    }
}
```

清单 23. 空循环延迟的示例任务的源代码已被对 vTaskDelay()的调用所取代



尽管这两项任务仍在不同的优先级下创建，但现在都将运行。 示例 4 的输出（如图 16 所示）确认了预期的行为。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975518-5fca50bb-fe3a-4557-b451-5a8f1f374495.png)

图16.执行示例4时产生的输出 

图 17 中显示的执行顺序解释了为什么两个任务都运行，即使它们是以不同的优先级创建的。 为简单起见，省略了调度程序本身的执行。



在启动调度程序时自动创建空闲任务，以确保始终有至少一个任务能够运行 (至少一个处于就绪状态的任务)。第 3.8 节，空闲任务和空闲任务钩子，更详细地描述了空闲任务。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975820-725d737a-27b4-486c-89ab-c7c758bb8c67.png)

图17. 任务使用vTaskDelay()代替NULL 循环时的执行顺序

这两个任务的实现发生了变化，没有改变它们的功能。将图 17 与图 12 进行比较，可以清楚地看到，实现此功能的方式要有效得多。



图 12 显示了当任务使用空循环来创建延迟时的执行模式——始终运行，因此使用它们之间可用处理器时间的百分之百。 图 17 显示了任务在整个延迟期间进入阻塞状态时的执行模式，因此只有在实际需要执行工作时才使用处理器时间（在这种情况下只是要打印出来的消息），并且结果只使用可用处理时间的一小部分。



在图 17 的场景中，每当任务离开阻塞状态时，它们会在重新进入阻塞状态之前执行一小部分时间。 大多数情况下，没有能够运行的应用程序任务（就绪状态下没有应用程序任务），因此，没有可以选择进入运行状态的应用程序任务。 在这种情况下，空闲任务将运行。 分配给空闲的处理时间量是系统中备用处理能力的度量。使用 RTOS 可以简单地通过允许应用程序完全由事件驱动来显着增加备用处理能力。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975351-cc37ae23-819c-4968-9d2a-44f926bfd9b0.png)

图 18.粗线表示例4中任务所执行的状态转换

图 18 中的粗线显示了示例 4 中的任务执行的转换，每个转换现在都转换到阻塞状态，然后返回到就绪状态。

### vTaskDelayUntil() API 函数

vTaskDelayUntil() 类似于 vTaskDelay()。 正如刚才演示的，vTaskDelay() 参数指定了调用vTaskDelay() 的任务与再次从阻塞状态过渡到正常状态的任务之间应该发生的滴动中断的数量。任务处于阻塞状态的时间长度由 vTaskDelay() 参数指定，但是任务离开阻塞状态的时间与调用 vTaskDelay() 的时间相关。

相反，vTaskDelayUntil() 的参数指定调用任务应该从阻塞状态移动到就绪状态的精确计时计数值。vTaskDelayUntil() 是在需要固定执行期间（您希望任务以固定频率定期执行）时应使用的 API 函数，因为调用任务被解除阻塞的时间是绝对的，而不是相对于调用函数时（与 vTaskDelay() 的情况一样）。

```groovy
void vTaskDelayUntil( TickType_t* pxPreviousWakeTime, TickType_t xTimeIncrement );
```

清单 24. vTaskDelayUntil() API 函数原型

表 10. vTaskDelayUntil() 的参数

| 参数名称           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| pxPreviousWakeTime | 此参数的命名是基于 vTaskDelayUntil() 用于实现定期执行且具有固定频率的任务。 在这种情况下，pxPreviousWakeTime 保持任务最后一次离开阻塞状态的时间（被 '唤醒'）。 此时间用作参考点，用于计算任务下次离开阻塞状态的时间。pxPreviousWakeTime 指向的变量在 vTaskDelayUntil()函数中自动更新；它通常不会被应用程序代码修改，但必须在第一次使用之前初始化为当前的滴答计数。 清单 25 演示了如何执行初始化。 |
| xTimeIncrement     | 此参数的命名也是基于 vTaskDelayUntil() 用于实现定期执行且具有固定频率的任务，频率由 xTimeIncrement 值设置。xTimeIncrement 在 'ticks' 中指定。 pdMS_TO_TICKS()宏可用于将毫秒指定的时间转换为刻度中指定的时间。 |

### 示例 5 将例子中的任务转换为使用vTaskDelayUntil()。

示例4中创建的两个任务是周期性任务，但是使用 vTaskDelay() 并不保证它们运行的频率是固定的，因为任务离开阻塞状态的时间与它们调用 vTaskDelay() 的时间相关。 将任务转换为使用vTaskDelayUntil() 而不是 vTaskDelay() 可以解决这个潜在的问题。

```groovy
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
TickType_t xLastWakeTime;

    /* 要打印的字符串通过参数传入。将此转换为字符指针。 */
    pcTaskName = ( char * ) pvParameters;

    /* 需要使用当前滴针计数初始化 xLastWakeTime 变量。注意，这是唯一一次显式地写入变量。
    在此之后，xLastWakeTime 将在 vTaskDelayUntil() 中自动更新。 */
    xLastWakeTime = xTaskGetTickCount();

    /* 对于大多数任务，都是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( pcTaskName );

        /* 这个任务应该精确地每 250 毫秒执行一次。根据 vTaskDelay() 函数，时间是以滴答为
        单位度量的，pdMS_TO_TICKS() 宏用于将毫秒转换为滴答。xLastWakeTime 在 
        vTaskDelayUntil() 中自动更新，因此任务不会显式地更新 xLastWakeTime。 */
        vTaskDelayUntil( &xLastWakeTime, pdMS_TO_TICKS( 250 ));
    }
}
```

清单 25. 使用 vTaskDelayUntil() 实现示例任务



示例 5 产生的输出与图 16 中的示例 4 所示的输出完全相同。



### 示例 6. 组合阻塞和非阻塞任务

前面的示例已经单独检查了轮询和阻塞任务的行为。 该示例通过在组合两个方案时演示执行序列来重新强制执行所述的预期系统行为，如下所述。



1. 在优先级 1 创建两个任务。除了连续打印字符串之外，它们什么也不做。这些任务从不执行任何可能导致它们进入阻塞状态的 API 函数调用，因此总是处于就绪状态或运行状态。这种性质的任务称为连续处理任务，因为它们总是有工作要做(尽管在本例中是相当琐碎的工作)。连续处理任务的源如清单 26 所示。



1. 然后在优先级 2 创建第三个任务，因此高于其他两个任务的优先级。第三个任务也只是打印出一个字符串，但这次是周期性的，所以使用 vTaskDelayUntil() API 函数在每次打印迭代之间将自己置于阻塞状态。



这个周期性任务的源代码如清单 27 所示。

```groovy
void vContinuousProcessingTask( void *pvParameters )
{
char *pcTaskName;

    /* 要打印的字符串通过参数传入。将此转换为字符指针。 */
    pcTaskName = ( char * ) pvParameters;

    /* 对于大多数任务，都务是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。这个任务只是重复地做这件事，从不阻塞或延迟。 */
        vPrintString( pcTaskName );
    }
}
```

清单 26. 示例 6 中使用的连续处理任务

```groovy
void vPeriodicTask( void *pvParameters )
{
TickType_t xLastWakeTime;
const TickType_t xDelay3ms = pdMS_TO_TICKS( 3 );

    /* 需要使用当前滴针计数初始化 xLastWakeTime 变量。注意，这是唯一一次显式地写入变量。
    在此之后，xLastWakeTime 由 vTaskDelayUntil() API 函数自动管理。 */
    xLastWakeTime = xTaskGetTickCount();

    /* 对于大多数任务，都务是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( "Periodic task is running\r\n" );

        /* 任务应该每 3 毫秒执行一次，在这个函数中可以看到 xDelay3ms 的声明。 */
        vTaskDelayUntil( &xLastWakeTime, xDelay3ms);
    }
}
```

清单 27. 示例 6 中使用的周期性任务



图 19 显示了示例 6 生成的输出，并通过图 20 中所示的执行顺序解释了观察到的行为。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636003548129-7c94f574-f7fc-4bbd-8a62-bab00cf22fa0.png)

图 19. 执行示例 6 时生成的输出



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636003547829-6ab4f336-8e9c-465b-be34-3828bb8c9d77.png)

图 20. 示例 6 的执行模式



## 空闲任务和空闲任务钩子

示例 4 中创建的任务将大部分时间用于阻塞状态。在此状态下，它们无法运行，因此调度程序也无法选择它们。



但必须始终至少有一个任务可以进入运行状态。 为确保这种情况，调用 vTaskStartScheduler() 时，调度程序会自动创建一个空闲任务。 空闲任务只是坐在一个循环中 —— 所以，就像原始第一个例子中的任务一样，它始终能够运行。



空闲任务具有可能的最低优先级（优先级为零），以确保它永远不会阻止更高优先级的应用程序任务进入运行状态 —— 尽管没有什么可以阻止应用程序设计者创建任务，从而共享空闲任务优先级，如果在FreeRTOSConfig.h 中，configIDLE_SHOULD_YIELD 编译时间配置常量可用于防止空闲任务消耗更高效地分配给应用程序任务的处理时间。 configIDLE_SHOULD_YIELD 在第 3.12 节 “调度算法” 中描述。



以最低优先级运行可确保只要优先级较高的任务进入就绪状态，空闲任务就会从运行状态转移出来。 这可以在图 17 中的时间点看到，其中空闲任务被立即换出以允许任务 2 在任务 2 离开被阻止状态的瞬间执行。 据说任务 2 已经抢占了空闲任务。 抢先自动发生，并且不知道任务被抢占。



如果应用程序使用 vTaskDelete() API 函数，那么空闲任务就不会浪费处理时间。这是因为空闲任务负责在删除任务之后清理内核资源。



### 空闲任务钩子函数

可以通过使用空闲钩子（或空闲回调）函数将应用程序特定功能直接添加到空闲任务中 —— 空闲任务循环每次迭代时由空闲任务自动调用的函数。



空闲任务钩子的常见用途包括：



- 执行低优先级，后台或连续处理功能。
- 测量备用处理能力的数量。（当所有优先级较高的应用程序任务无法执行时，空闲任务将会运行；因此，测量分配给空闲任务的处理时间量可清楚地指示多少处理时间。）

- 将处理器置于低功耗模式，在没有应用处理的情况下提供简单且自动的省电方法（虽然使用这种方法所能达到的省电比使用第 10 章低功耗支持中描述的无时钟空闲模式所能达到的省电要少）。



### 对空闲任务钩子函数实现的限制

空闲任务钩子函数必须遵循以下规则。



1. 一个空闲的任务钩子函数决不能试图阻塞或挂起。
2. 如果应用程序使用了 vTaskDelete() API函数，那么空闲任务钩子必须总是在合理的时间段内返回给调用者。这是因为空闲任务负责在删除任务之后清理内核资源。如果空闲任务永久地保留在空闲钩子函数中，则无法进行清理。



以任何方式阻塞空闲任务都可能导致无法使用任何任务进入运行状态。



空闲任务钩子函数必须具有清单 28 所示的名称和原型。

```groovy
void vApplicationIdleHook( void );
```

清单 28. 空闲任务钩子函数名称和原型

### 示例 7. 定义空闲任务钩子功能

在执行空闲任务时，使用示例 4 中的阻塞 vTaskDelay() API调用会创建大量空闲时间，因为两个应用程序任务都处于阻塞状态。 示例 7 通过添加空闲挂钩函数来利用此空闲时间，其源代码如清单 29 所示。

```groovy
/* 声明一个将由钩子函数递增的变量。 */
volatile uint32_tulIdleCycleCount = 0UL;

/* 空闲钩子函数必须被称为 vApplicationIdleHook()，不带参数，并返回 void。 */
void vApplicationIdleHook( void )
{
    /* 这个钩子函数除了递增计数器外什么都不做。 */
    ulIdleCycleCount++;
}
```

清单 29. 一个非常简单的空闲钩子函数



config_IDLE_HOOK 必须在 FreeRTOSConfig.h 设置为 1 ，以便调用空闲钩子函数。



稍微修改实现创建任务的函数以打印出 ulIdleCycleCount 值，如清单 30 所示。

```groovy
void vTaskFunction( void *pvParameters )
{
char *pcTaskName;
const TickType_t xDelay250ms = pdMS_TO_TICKS( 250 );

    /* 要打印的字符串通过参数传入。将此转换为字符指针。 */
    pcTaskName = ( char * ) pvParameters;

    /* 与大多数任务一样，此任务在无限循环中实现。 */
    for( ;; )
    {
        /* 打印出此任务的名称，并加入 ulIdleCycleCount 的次数。 */
        vPrintStringAndNumber( pcTaskName, ulIdleCycleCount);

        /* 延迟 250 毫秒。 */
        vTaskDelay( xDelay250ms);
    }
}
```

清单 30. 示例任务的源代码现在打印出 ulIdleCycleCount 值

示例 7 产生的输出如图 21 所示。 因此，在应用任务的每次迭代之间，空闲任务钩子函数被调用大约 400 万次（迭代次数取决于演示执行的硬件的速度）。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636004122657-a8018e6b-03d7-4323-9f8e-55587af3fb20.png)

图 21. 执行示例 7 时生成的输出



## 更改任务的优先级

### vTaskPrioritySet() API 函数

vTaskPrioritySet() API 函数可用于在调度程序启动后更改任何任务的优先级。请注意，当 FreeRTOSConfig.h 中的 INCLUDE_vTaskPrioritySet 设置为 1 时，vTaskPrioritySet() API 函数是可用的。

```groovy
void vTaskPrioritySet( TaskHandle_t pxTask, UBaseType_t uxNewPriority );
```

清单 31. vTaskPrioritySet() API 函数原型

表 11. vTaskPrioritySet() 参数

| 参数名称      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| pxTask        | 正在修改其优先级的任务的句柄（主题任务）—— 有关获取任务句柄的信息，请参阅xTaskCreate() API函数的 pxCreatedTask 参数。任务可以通过传递 NULL 代替有效的任务句柄来更改自己的优先级。 |
| uxNewPriority | 要设置主题任务的优先级。 这自动限制为（configMAX_PRIORITIES - 1）的最大可用优先级，其中 configMAX_PRIORITIES 是在 FreeRTOSConfig.h 头文件中设置编译时间常量。 |

### uxTaskPriorityGet() API 函数

uxTaskPriorityGet() API 函数可用于查询任务的优先级。请注意，只有在 FreeRTOSConfig.h 中将INCLUDE_uxTaskPriorityGet 设置为 1 时，uxTaskPriorityGet() API 函数才可用。

```groovy
UBaseType_t uxTaskPriorityGet( TaskHandle_t pxTask );
```

清单 32. uxTaskPriorityGet() API 函数原型

表 12. uxTaskPriorityGet()参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| pxTask          | 正在查询其优先级的任务的句柄（主题任务）— 请参阅 xTaskCreate()API 函数的pxCreatedTask参数，以获取有关获取任务句柄的信息。任务可以通过传递 NULL 代替有效的任务句柄来查询自己的优先级。 |
| 返回值          | 当前分配给正在查询的任务的优先级。                           |

### 示例 8. 改变任务优先级

调度程序将始终选择最高就绪状态任务作为进入运行状态的任务。示例 8 通过使用 vTaskPrioritySet() API 函数来更改两个任务相对于彼此的优先级来证明这一点。



示例 8 以两个不同的优先级创建两个任务。 这两个任务都没有任何 API 函数调用可能导致它进入阻塞状态，因此无法处于就绪状态或运行状态。 因此，具有最高相对优先级的任务将始终是调度程序选择处于 “正在运行” 状态的任务。



示例 8 行为如下：



1. 创建任务 1（清单 33）具有最高优先级，因此保证首先运行。 任务 1 在将任务 2（清单 34）的优先级提高到高于其自己的优先级之前打印出几个字符串。
2. 任务 2 一旦具有最高相对优先级就开始运行（进入运行状态）。 任何时候只有一个任务可以处于运行状态，因此当任务 2 处于运行状态时，任务 1 处于就绪状态。

1. 任务 2 在将自己的优先级设置回低于任务 1 的优先级之前打印出一条消息。
2. 任务 2 将其优先级降低意味着任务 1 再次成为优先级最高的任务，因此任务 1 进入运行状态，强制将任务 2 回到就绪状态。

```groovy
void vTask1( void *pvParameters )
{
UBaseType_t uxPriority;

    /* 这个任务总是在任务2之前运行，因为它是用更高的优先级创建的。
    Task 1和Task 2都不会阻塞，因此两者都将始终处于运行或就绪状态。 

    查询此任务正在运行的优先级——传入NULL意味着 “返回调用任务的权限”。 */
    uxPriority = uxTaskPriorityGet( NULL );

    for( ;; )
    {
        /* 打印出此任务的名称。 */
        vPrintString( "Task 1 is running\r\n" );

        /* 将 Task 2 的优先级设置在 Task 1 的优先级之上，将导致 Task 2 立即开始运行
        (因为此时 Task 2 的优先级将高于创建的两个任务)。注意在调用 vTaskPrioritySet() 
        时使用 task 2 的句柄( xTask2Handle )。清单 35 显示了如何获得句柄。 */
        vPrintString( "About to raise the Task 2 priority\r\n" );
        vTaskPrioritySet( xTask2Handle, ( uxPriority + 1 ) );

        /* 任务 1 只有在优先级高于任务 2 时才会运行。因此，要使此任务达到此点，
        任务 2 必须已经执行，并将其优先级重新设置为低于此任务的优先级。 */
    }
}
```

清单 33. 示例 8 中任务 1 的实现

```groovy
void vTask2( void *pvParameters )
{
UBaseType_t uxPriority;

    /* 任务 1 总是在此任务之前运行，因为任务 1 具有更高的优先级。
    任务 1 和任务 2 都不会阻塞，因此总是处于运行或就绪状态。

    查询此任务正在运行的优先级——传入 NULL 意味着“返回调用任务的优先级”。*/
    uxPriority = uxTaskPriorityGet( NULL );

    for( ;; )
    {
        /* 要使此任务达到此点，任务 1 必须已经运行并将此任务的优先级设置为高
        于其自身的优先级。 

        打印出此任务的名称。 */
        vPrintString( "Task 2 is running\r\n" );

        /* 将此任务的优先级设置回其初始值。传递 NULL 作为任务句柄意味着
        “更改调用任务的优先级”。将任务 1 的优先级设置为低于任务 1 的优先级，
        将导致任务 1 立即重新开始运行，从而抢占该任务。 */
        vPrintString( "About to lower the Task 2 priority\r\n" );
        vTaskPrioritySet( NULL, ( uxPriority - 2) );
    }
}
```

每个任务都可以查询和设置自己的优先级，而不需要使用有效的任务句柄，只需使用 NULL 即可。只有当任务希望引用任务本身之外的任务时，才需要任务句柄，例如当任务 1 更改任务 2 的优先级时。为了允许任务 1 这样做，在创建任务 2 时获取并保存任务 2 句柄，如清单 35 中的注释所突出显示的内容。

```groovy
/* 声明一个用于保存任务 2 句柄的变量。 */
TaskHandle_t xTask2Handle = NULL;

int main( void )
{
    /* 创建优先级为 2 的第一个任务。不使用任务参数并将其设置为 NULL。
    任务句柄也没有使用，因此也设置为 NULL。 */
    xTaskCreate( vTask1, "Task 1", 1000, NULL, 2, NULL );
    /* 任务以优先级 2 创建 ______________________^. */

    /* 创建优先级为 1 的第二个任务——它低于任务1的优先级。同样，没有使用任务参数，
    所以将其设置为 NULL —— 但是这次需要任务句柄，因此在最后一个参数中传递了
    xTask2Handle 的地址。 */
    xTaskCreate( vTask2, "Task 2", 1000, NULL, 1, &xTask2Handle );
    /* 任务句柄是最后一个参数 _______________________^^^^^^^^^^^^^ */

    /* 启动调度程序，使任务开始执行。 */
    vTaskStartScheduler();    

    /* 如果一切正常，那么 main() 将永远不会到达这里，因为调度程序现在将运行这些任务。
    如果 main() 确实到达这里，那么很可能没有足够的堆内存来创建空闲任务。第 2 章提供了
    关于堆内存管理的更多信息。 */
    for( ;; );
}
```

清单 35. 示例 8 的 main() 的实现



图 22 显示了示例 8 任务执行的顺序，结果输出如图 23 所示。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005149753-00f92ede-12e4-4676-9c1f-c1033c91be31.png)

图 22. 运行示例 8 时的任务执行顺序



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005149758-ae7dbbb5-5b42-498d-a9f2-657426972854.png)

图 23. 执行示例 8 时产生的输出



## 删除任务

### vTaskDelete() API 函数

任务可以使用 vTaskDelete() API 函数来删除自身或任何其他任务。 请注意，只有在 FreeRTOSConfig.h 中将 INCLUDE_vTaskDelete 设置为 1 时，vTaskDelete() API 函数才可用。

删除的任务将不再存在，且不能再进入运行模式。

释放已删除的任务所分配的内存是空闲任务的职责。因此，重要的是，使用 vTaskDelete() API 函数的应用程序不占用所有处理时间，而饿死空闲任务。



只有内核本身分配给任务的内存在删除任务时才会自动释放。任何分配的任务的实现必须明确释放的内存或其他资源。

```groovy
void vTaskDelete( TaskHandle_t pxTaskToDelete );
```

清单 36. vTaskDelete() 函数原型

表 13. vTaskDelete() 参数

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| pxTaskToDelete  | 要删除的任务的句柄（主题任务）—— 请参阅 xTaskCreate() API 函数的pxCreatedTask 参数，以获取有关获取任务句柄的信息。任务可以通过在有效的任务句柄处传递 NULL 来删除自己。 |

### 示例9. 删除任务

这是一个非常简单的示例，其行为如下。



1. 任务 1 由 main() 创建，优先级为 1。当它运行时，它以优先级 2 创建任务 2。任务 2 现在是最高优先级的任务，因此它立即开始执行。main() 的源代码如清单 37 所示，任务 1 的源代码如清单 38 所示。



1. 任务 2 除了删除自己之外，什么也不会做。它可以通过向 vTaskDelete() 传递 NULL 来删除自己，但是处于演示目的，它会使用自己的任务句柄。任务 2 的源代码如清单 39 所示。



1. 当任务 2 被删除后，任务 1 再次成为优先级最高的任务，所以它将继续执行 —— 此时它会调用 vTaskDelay() 来阻塞一小段时间。



1. 空闲任务会在任务 1 阻塞状态的时候执行，并释放之前分配给先已删除了的任务 2 的内存。



1. 当任务 1 离开阻塞状态的时候，它将再次变为最高优先级的就绪状态任务，从而抢占空闲任务。当进入运行状态后，它将再次创建任务 2， 并且如此这样继续下去。

```groovy
int main( void )
{
    /* 以优先级 1 创建第一个任务。不使用任务参数，因此将其设置为NULL。 
    任务句柄也未使用，因此同样设置为NULL。 */
    xTaskCreate( vTask1, "Task 1", 1000, NULL, 1, NULL );
    /* 创建该任务的优先级为 1 ____________________^. */

    /* 启动调度程序，以便任务开始执行。 */
    vTaskStartScheduler();    

    /* 由于调度程序已经启动，main() 永远不会到达此处。 */
    for( ;; );
}
```

清单 37. 示例 9 的 main() 的实现

```groovy
TaskHandle_t xTask2Handle = NULL;

void vTask1( void *pvParameters )
{
const TickType_txDelay100ms = pdMS_TO_TICKS( 100UL );

    for( ;; )
    {
        /* 打印这个任务的名称 */
        vPrintString( "Task 1 is running\r\n" );

        /* 创建一个更高优先级的任务 2。 同样，没有使用任务参数，所以将其设置为 NULL 
        —— 但是这次需要任务句柄，因此将 xTask2Handle 的地址作为最后一个参数传递。*/
        xTaskCreate( vTask2, "Task 2", 1000, NULL, 2, &xTask2Handle );
        /* 任务句柄是最后一个参数 _______________________^^^^^^^^^^^^^ */

        /* 任务 2 具有更高的优先级，因此任务 1 到达此处任务 2 必须已经执行并自行删除。 
        延迟 100 毫秒。 */
        vTaskDelay( xDelay100ms );
    }
}
```

清单 38. 示例 9 任务 1 的实现

```groovy
void vTask2( void *pvParameters )
{
    /* Task 2 只做删除本身。要做到这一点，它可以使用 NULL 作为参数调用 vTaskDelete()，
    但是，出于纯粹的演示目的，它在调用 vTaskDelete() 时会传递自己的任务句柄。 */
    vPrintString( "Task 2 is running and about to delete itself\r\n" );
    vTaskDelete( xTask2Handle );
}
```

清单 39. 示例 9 任务 2 的实现

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005679119-d75bf7e7-93ac-4db3-a9cb-b7f58677a559.png)

图 24. 执行示例 9 的输出结果



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005678926-6cd45eaf-06f1-4961-89ac-b416728bc6a0.png)

图 25. 示例 9 的执行顺序



## 线程本地存储

待完成。本节将在最终发布之前编写。



译者注：原文就是待完成。





## 调度算法

### 任务状态和事件的回顾

实际运行的任务（使用处理时间）处于运行状态。 在单核处理器上，在任何给定时间只能有一个任务处于运行状态。

实际上未运行的任务，既没有处于阻塞状态，也没有处于挂起状态，那就是处于就绪状态。调度程序可以选择处于就绪状态的任务作为进入运行状态的任务。调度程序总是选择优先级最高的就绪状态任务来进入运行状态。

任务可以在阻塞状态中等待事件，并在事件发生时自动移动回就绪状态。时间事件发生在特定的时间，例如，当阻塞时间过期时，通常用于实现周期性或超时行为。同步事件发生在任务或中断服务例程使用任务通知、队列、事件组或多种信号量类型之一发送信息时。它们通常用于发出异步活动的信号，例如有数据到达外设。

### 配置调度算法

调度算法是决定哪个就绪状态的任务变成运行任务的程序。

到目前为止，所有示例都使用了相同的调度算法，但可以通过配置 configUSE_PREEMPTION 和 configUSE_TIME_SLICING 常量更改算法。 两个常量都在 FreeRTOSConfig.h 中定义。

第三个配置常量 configUSE_TICKLESS_IDLE 也会影响调度算法，因为它的使用会导致滴答中断在较长时间内完全关闭。configUSE_TICKLESS_IDLE 是一个高级选项，专门用于必须最小化其功耗的应用程序。 configUSE_TICKLESS_IDLE 在第10章，低功耗支持中描述。 本节中提供的描述假设configUSE_TICKLESS_IDLE 设置为 0，如果常量未定义，则这是默认设置。

在所有可能的配置中，FreeRTOS 调度程序将确保选择共享优先级的任务以依次进入运行状态。 这种 “反过来” 政策通常被称为 “循环调度”。 循环调度算法不保证在相同优先级的任务之间平均分配时间，只有优先级相同的就绪状态任务依次进入运行状态。

### 具有时间切片的优先抢占式调度

表 14 中所示的配置将 FreeRTOS 调度程序设置为使用称为“带时间切片的固定优先级预先调度” 的调度算法，这是大多数小型 RTOS 应用程序使用的调度算法，以及所有示例中使用的算法。 到目前为止这本书。 表 15 中提供了算法名称中使用的术语的描述。

表 14. FreeRTOSConfig.h 用于配置内核以使用带时间片的优先级预先调度的设置

| 常量                   | 值   |
| ---------------------- | ---- |
| configUSE_PREEMPTION   | 1    |
| configUSE_TIME_SLICING | 1    |

表 15. 用于描述调度策略的术语的解释

术语

| 术语       | 定义                                                         |
| ---------- | ------------------------------------------------------------ |
| 固定优先级 | 描述为 “固定优先级” 的调度算法不会更改分配给正在调度的任务的优先级，但也不会阻止任务本身更改自己的优先级或其他任务的优先级。 |
| 抢占       | 如果优先级高于运行状态任务的任务进入就绪状态，则先发制人调度算法将立即“抢占”运行状态任务。 被抢占意味着不自觉地（不明显地屈服或阻塞）移出运行状态并进入就绪状态以允许不同的任务进入运行状态。 |
| 时间分片   | 时间分片用于在相同优先级的任务之间共享处理时间，即使任务确实明确地产生或进入阻塞状态。 如果存在与运行任务具有相同优先级的其他就绪状态任务，则使用时间分片描述的调度算法将选择新任务以在每个时间片结束时进入运行状态。时间片等于两次 RTOS 滴答中断之间的时间。 |

图 26 和图 27 说明了当使用具有时间分片算法的固定优先级抢占式调度时如何调度任务。 图 26 显示了当应用程序中的所有任务具有唯一优先级时，选择任务以进入“运行”状态的顺序。 图 27 显示了当应用程序中的两个任务共享优先级时，选择任务以进入“运行”状态的顺序。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163237-1d4e45b5-6947-4f62-abed-8c5843fc6685.png)

图 26. 执行模式在一个假设的应用程序中突出显示任务优先级和优先级，其中每个任务都被赋予了唯一的优先级



图 26 参考：



1. **空闲任务**：空闲任务以最低优先级运行，因此每次较高优先级任务进入就绪状态时都会被抢占 —— 例如，在时间 t3，t5 和 t9。
2. **任务 3**：任务 3 是一个事件驱动的任务，它以相对较低的优先级执行，但高于空闲优先级。 它大部分时间都处于阻塞状态，等待其感兴趣的事件，每次事件发生时从阻塞状态转换到就绪状态。 所有 FreeRTOS 任务间通信机制（任务通知，队列，信号量，事件组等）都可用于以这种方式发送事件和解除阻塞任务。事件发生在时间 t3 和 t5，也发生在 t9 和 t12 之间。 在时间 t3 和 t5 发生的事件被立即处理，因为在这些时间，任务 3 是能够运行的最高优先级任务。 在时间 t9 和 t12 之间某处发生的事件直到 t12 才被处理，因为在此之前，优先级较高的任务任务 1 和任务 2 仍在执行。 仅在时间 t12，任务 1和任务 2 都处于阻塞状态，使得任务 3 成为最高优先级就绪状态任务。

1. **任务 2**：任务 2 是周期性任务，其执行优先于任务 3 的优先级，但低于任务 1 的优先级。任务的周期间隔意味着任务 2 想要在时间 t1，t6 和 t9 执行。在时间 t6，任务 3 处于运行状态，但任务 2 具有较高的相对优先级，因此预先占用任务 3 并立即开始执行。 任务 2 完成其处理并在时间 t7 重新进入阻塞状态，此时任务 3 可以重新进入运行状态以完成其处理。 任务 3 本身在时间 t8 阻塞。
2. **任务 1**： 任务 1 也是一个事件驱动任务。 它以所有的最高优先级执行，因此可以抢占系统中的任何其他任务。 所示的唯一任务 1 事件发生在时间 t10，此时任务 1 抢占任务 2。任务 2 仅在任务 1 在时间t11 重新进入阻止状态之后才能完成其处理。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163243-98b7aa16-3853-47e8-9cad-6ede003b0e10.png)

图 27. 执行模式在一个假设的应用程序中突出显示任务优先级和时间切片，其中两个任务以相同的优先级运行



图 27 参考：

1. **空闲任务与任务 2**：空闲任务和任务 2 都是连续处理任务，两者的优先级均为 0（可能的最低优先级）。 当没有能够运行的优先级较高的任务时，调度器会将处理时间分配给优先级为 0 的任务，并通过时间分片共享分配给优先级 0 任务的时间。 每个嘀嗒中断开始一个新的时间片，在图 27 中，时间为 t1，t2，t3，t4，t5，t8，t9，t10 和 t11。空闲任务和任务 2 依次进入运行状态，这可导致两个任务在同一时间片的一部分处于运行状态，如在时间 t5 和时间 t8 之间发生。
2. **任务 1**：任务 1 的优先级高于空闲优先级。 任务 1 是一个事件驱动的任务，它将大部分时间都置于阻塞状态，等待它感兴趣的对象，每次事件发生时从阻塞状态转换到就绪状态。感兴趣的事件发生在时间 t6，因此在 t6，任务 1 成为能够运行的最高优先级任务，因此任务 1 在时间片中部分地抢占空闲任务。 事件的处理在时间 t7 完成，此时任务 1 重新进入阻止状态。



图 27 显示了由应用程序编写者创建的任务的空闲任务共享处理时间。 如果应用程序编写者创建的空闲优先级任务有工作要做，但空闲任务没有，则可能不需要将大量处理时间分配给空闲任务。configIDLE_SHOULD_YIELD 编译时配置常量可用于更改空闲任务的调度方式：



- 如果 configIDLE_SHOULD_YIELD 设置为 0，那么空闲任务将保持其整个时间片的运行状态，除非它被更高优先级的任务抢占。
- 如果 configIDLE_SHOULD_YIELD 设置为 1，那么空闲任务将在其循环的每次迭代中产生（自愿放弃其分配的时间片的剩余部分），并且在就绪状态下还有其他空闲优先级任务。



图 27 中显示的执行模式是当 configIDLE_SHOULD_YIELD 设置为 0 时将观察到的内容。图 28 中显示的执行模式是在 configIDLE_SHOULD_YIELD 设置为 1 时在同一场景中观察到的内容。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163221-747cec7a-77d0-4b45-901f-6312954d9970.png)

图 28. 同一场景的执行模式如图 27 所示，但这次将 configIDLE_SHOULD_YIELD 设置为1

图 28 还显示，当 configIDLE_SHOULD_YIELD 设置为 1 时，在空闲任务之后选择进入运行状态的任务不会对整个时间片执行，而是执行转发空闲任务所产生的时间片的剩余时间。

### 优先级抢占式调度(没有时间分片)

没有时间分片的优先级抢先式调度保持与前一节中描述的相同的任务选择和抢占算法，但不使用时间分片来共享相同优先级的任务之间的处理时间。



FreeRTOSConfig.h 设置将 FreeRTOS 调度程序配置为使用优先级抢占式调度而不进行时间切片，如表 16所示。



表 16. FreeRTOSConfig.h 设置，用于配置内核以使用优先级预先调度而无需时间切片

| 常量                   | 值   |
| ---------------------- | ---- |
| configUSE_PREEMPTION   | 1    |
| configUSE_TIME_SLICING | 0    |

如图 27 所示，如果使用时间分片，并且有多个能够运行的最高优先级的就绪状态任务，那么调度程序将选择一个新任务以在每个 RTOS 节拍中断期间进入运行状态 （标记时间片结束的滴答中断）。 如果未使用时间分片，则调度程序将仅选择新任务以在以下任一情况下进入运行状态：

- 优先级较高的任务进入就绪状态。
- 处于运行状态的任务进入阻塞或挂起状态。

当不使用时间分片时，与使用时间分片时相比，任务上下文切换更少。 因此，关闭切换时间会导致调度程序的处理开销减少。但是，关闭时间切换也会导致相同优先级的任务接收到大大不同的处理时间量，如图 29 所示。出于这个原因，运行调度程序没有时间切片被认为是一种只有经验丰富的用户才能使用的高级技术。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006430382-395d6c69-99e1-4193-84f9-1a7f47caa650.png)

图 29. 执行模式，演示当不使用时间切片时，具有相同优先级的任务如何可以获得非常不同的处理时间量

图 29 参考，假设 configIDLE_SHOULD_YIELD 设置为 0：



1. **滴答中断**：滴答中断发生在 t1、t2、t3、t4、t5、t8、t11、t12 和 t13 时刻。
2. **任务 1**：任务 1 是一个高优先级事件驱动的任务，大部分时间都处于阻塞状态，等待它感兴趣的对象发生。 每次事件发生时，任务 1 从阻塞状态转换到就绪状态（并且随后，因为它是最高优先级的就绪状态任务，进入运行状态）。图 29 显示在时间 t6 和 t7 之间处理事件的任务 1， 然后在时间 t9 和 t10 之间再次出现。

1. **空闲任务与任务 2**：空闲任务和任务 2 都是连续处理任务，并且两者都具有优先级 0（空闲优先级）。连续处理任务不进入阻塞状态。未使用时间分片，因此处于“正在运行”状态的空闲优先级任务将保持“正在运行” 状态，直到它被优先级较高的任务 1 抢占为止。在图 29 中，空闲任务在时间 t1 开始运行，并且保持在运行状态，直到它被任务 1 在时间 t6 抢占 —— 这在进入运行状态之后超过四个完整的滴答时段。任务 2 在时间 t7 开始运行，这时任务 1 重新进入阻塞状态以等待另一个事件。 任务 2 保持在运行状态，直到任务 1 在时间 t9 之前被抢占 —— 在进入运行状态之后小于一个滴答周期。在时间 t10，空闲任务重新进入运行状态，尽管已经接收到比任务 2 多四倍以上的处理时间。

### 协作式调度

本书主要介绍抢占式调度，但 FreeRTOS 也可以使用协作式调度。配置 FreeRTOS 调度程序使用协作式调度的 FreeRTOSConfig.h 设置如表 17 所示。

表 17. FreeRTOSConfig.h 设置，配置内核使用协作式调度

| 常量                   | 值     |
| ---------------------- | ------ |
| configUSE_PREEMPTION   | 0      |
| configUSE_TIME_SLICING | 任何值 |

当使用协作式调度程序时，只有在运行状态任务进入阻塞状态，或者运行状态任务通过调用 taskYIELD() 显式地产生（手动请求重新调度）时，才会发生上下文的切换。 任务永远不会被抢占，因此不能使用时间切片。



图 30 展示了协作式调度程序的行为。图 30 中的水平虚线显示任务何时处于就绪状态。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006700649-8ef52549-2f45-48ad-89c1-0111cca7958b.png)

图 30. 显示协作式调度程序行为的执行模式

图 30 参考：



1. **任务 1**：任务 1 有最高优先级，它开始于阻塞状态，等待一个信号量。在 t3 时刻，一个中断给予了信号量，引起任务 1 离开阻塞状态并进入就绪状态（第 6 章涵盖了来自中断给与的信号量的相关内容）。在 t3 时刻，任务 1 是最高优先级的任务，如果是抢占模式调度器会将任务 1 直接变成运行态任务，然而在协作式调度下，任务 1 任然是就绪态，直到 t4 ——当运行态的任务调用 taskYIELD() 时。
2. **任务 2**：任务 2 的优先级介于任务 1 与任务 3。它同样开始于阻塞状态，等待一个在 t2 时刻由任务 3 发出的消息。在 t2 时刻，任务 2 是最高优先级的就绪态任务，如果是抢占式调度，会将任务 2 直接变成运行态任务，然而在协作式调度下，任务 2 任然是就绪态，知道运行态任务调用 taskYIELD() 。运行态任务调用 taskYIELD() 在 t4 时刻，但是任务 1 是最高优先级的就绪态任务，所以任务 2 实际没有成为运行态任务，直到 t5 时刻任务 1 重新进入阻塞状态。在 t6 时刻，任务 2 重新进入阻塞状态来等待下一个消息，在这个点任务 3 再一次成为最高优先级的就绪态任务。

在一个多任务应用程序中，应用程序编写者必须注意同一个资源会被多个任务同时访问，因为同时访问可能会对该资源造成破坏。例如，考虑下面的场景，其中访问的时 UART 接口。两个任务是将字符串写入 UART， 任务 1 写的是 "abcdefghijklmnop" ，任务 2 写的是 "123456789" ：



1. 任务 1 是运行态，并且开始写字符串。当写到 "abcdefg" 时离开运行态。
2. 任务 2 进入运行态，同样开始写字符串 "123456789" 到 UART，并离开运行态。

1. 任务 1 重新进入，并将剩余的字符串写入 UART。



在这种情况下实际写入 UART 的是 "abcdefg123456789hijklmnop" 。任务 1 编写的字符串没有按照预期的顺序完整的写入 UART，而是被破坏了，因为任务 2 写入 UART 的字符串有人出现在其中。



通常，使用协作式调度比抢占式调度更容易避免由于时访问而造成的问题：



- 当使用抢占式调度时，可以在任何时刻抢占运行态的任务，包括与另一个任务共享资源的处于不一致的状态时。正如 UART 的示例所示，让资源处于不一致的状态可能会导致资源损坏。
- 当使用协作式调度时，应用程序编写者控制着任务合适切换到另一个任务。因此，应用程序编写者能够确保资源处于不一致状态时不会被切换到另一个任务。

- 在上面的 UART 示例中，应用程序编写者可以确保任务 1 在整个字符串写入 UART 之前不会离开运行状态，这样做可以消除字符串被另一个任务激活所破坏的可能性。



如图 30 所示，使用协作式调度的系统响应速度将低于抢占式调度的响应速度：



- 当使用抢占式调度时，调度程序将立即开始运行任务，该任务将成为最高优先级的就绪态任务。这在实时系统中，通常是必不可少的，因为实时系统必须在规定的时间内响应最高优先级事件。
- 当使用协作式调度时，只有正在运行的任务进入阻塞状态或者调用 taskYIELD() 时，才会切换到已成为最高优先级的就绪态任务。



# **队列管理**



## 章节介绍与范围

“队列” 提供了一个**任务到任务**，**任务到中断**和**中断到任务**的通讯机制。

### 范围

本章节的目标是告诉读者很好的理解：

- 如何创建队列。
- 一个队列是如何管理它所包含的数据。

- 如何发送数据到队列。
- 如何从队列接收数据。

- 阻塞队列意味着什么。
- 如何阻塞多个队列。

- 如何覆盖队列中的数据。
- 如何清除一个队列。

- 读取和写入一个队列对任务优先级的影响。

本章节只涵盖了任务到任务通讯。任务到中断与中断到任务通讯在第 6 章中说明。



## 队列的特征

### 数据存储

一个队列能保存有限数量的固定大小的数据单元。一个队列能保存单元的最大数量叫做 “长度”。每个队列数据单元的长度与大小是在创建队列时设置的。



队列通常是一个先入先出（FIFO）的缓冲区，即数据在队列末尾（tail）被写入，在队列前部（head）移出。图 31 展示了数据被写入和移出作为 FIFO 使用的队列。也可以写入队列的前端，并覆盖已位于队列前端的数据。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636007539867-7cde74d6-3c85-4ca7-8964-91adf229f12a.png)

图 31. 写入队列和从队列读取的示例序列

有两种方法可以实现队列的行为：



1. 通过复制实现队列：复制队列是指将发送到队列的数据一个字节一个字节地复制到队列中。
2. 通过引用实现队列：引用队列意味着队列只持有指向发送到队列的数据的指针，而不是数据本身。



FreeRTOS 是通过使用复制方法实现队列。这是考虑到复制队列比引用队列更强大，更容易使用，因为：



- 堆栈变量可以直接发送到队列，即使该变量将在声明它的函数退出后，不再存在。
- 可以将数据发送到队列，而无需先分配缓冲区来保存数据，然后将数据复制到分配的缓冲区中。

- 发送任务可以立即重用发送到队列的变量或缓冲区。
- 发送任务和接收任务是完全解耦的，应用程序设计人员不需要关心哪个任务拥有数据，或者哪个任务负责发布数据。

- 复制队列并不会阻止队列也被用于引用队列。例如，当正在排队的数据的大小使得将数据复制到队列不切实际时，可以将指向数据的指针复制到队列中。
- RTOS 完全负责分配用于存储数据的内存。

- 在受内存保护的系统中，任务可以访问的 RAM 将受到限制。在这种情况下，只有当发送和接收任务都可以访问存储数据的 RAM 时，才可以使用引用排队。按复制排队不受此限制；内核总是以完全特权运行，允许使用队列跨内存保护边界传递数据。

### 多任务访问

队列本身就是对象，任何知道它们存在的任务或 ISR 都可以访问它们。任意数量的任务可以写入同一个队列，任意数量的任务也可以从同一个队列读取。在实践中，队列有多个写入者是非常常见的，但是队列有多个读取者就不那么常见了。

### 阻塞队列读取

当任务尝试从队列中读取时，它可以选择指定 “阻塞” 时间。 如果队列已经为空，则这是任务将保持在阻塞状态以等待队列中的数据可用的时间。 当另一个任务或中断将数据放入队列时，处于阻塞状态且等待数据从队列中变为可用的任务将自动移至就绪状态。 如果指定的阻塞时间在数据可用之前到期，则任务也将自动从 “阻塞” 状态移动到 “就绪” 状态。

队列可以有多个读取者，因此单个队列可能会由多个在其上阻塞等待数据的任务。 在这种情况下，只有一个任务在数据可用时将被解除阻塞。 取消阻塞的任务始终是等待数据的最高优先级任务。 如果被阻塞的任务具有相同的优先级，那么等待数据最长的任务将被阻塞。

### 阻塞队列写入

与从队列读取数据时一样，任务也可以在向队列写入数据时指定阻塞时间。在这种情况下，如果队列已经满了，则阻塞时间是任务应该保持在阻塞状态以等待队列上可用空间的最长时间。

队列可以有多个写入者，因此对于一个完整的队列，可能有多个任务阻塞在队列上，等待完成发送操作。在这种情况下，当队列上的空间可用时，只有一个任务将被解除阻塞。未阻塞的任务总是等待空间的最高优先级任务。如果阻塞的任务具有相同的优先级，那么等待空间最长的任务将被解除阻塞。

### 阻塞多个队列

队列可被分组到集合中，允许任务进入阻塞状态来等待数据在集合的任何队列中变为可用。队列集合在第 4.6 章节 “从多个队列接收” 中展示。

## 使用队列

### xQueueCreate() API 函数

一个队列在使用前必须被显式的创建。

队列被句柄引用，句柄是类型为 QueueHandle_t 类型的变量。xQueueCreate() API 函数会创建一个队列，并给一个 QueueHandle_t 的变量来引用这个被创建的队列。

FreeRTOS V9.0.0 也包含了 xQueueCreateStatic() 函数，它创建队列是在编译时静态地分配内存。当一个队列创建时，FreeRTOS 是从 FreeRTOS 堆中分配所需 RAM。这一段 RAM 被用来保存队列数据结构和队列所含的各个单元。xQueueCreate() 在创建队列所需 RAM 不足时会返回 NULL 。第 2 章提供了 FreeRTOS 堆的更多信息。

```groovy
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength, UBaseType_t uxItemSize);
```

清单 40. xQueueCreate() API 函数原型

表 18. xQueueCreate() 参数和返回值

| 参数名        | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| uxQueueLength | 正在创建的队列一次可以容纳的最大项数。                       |
| uxItemSize    | 可以存储在队列中的每个数据项的字节大小。                     |
| 返回值        | 如果返回 NULL，则无法创建队列，因为 FreeRTOS 没有足够的堆内存来分配队列数据结构和存储区域。返回的非空值表示队列已成功创建。返回的值应该存储为已创建队列的句柄。 |

创建队列后，可以使用 xQueueReset() API 函数将队列返回到其原始的空状态。

### xQueueSendToBack() 与 xQueueSendToFront() API 函数

正如所料，xQueueSendToBack() 用于将数据发送到队列的后端（尾部），xQueueSendToFront() 用于将数据发送到队列的前端（头部）。



xQueueSend() 与 xQueueSendToBack() 等价，并且完全相同。创建队列后，可以使用 xQueueReset() API 函数将队列返回到其原始的空状态。



永远不要从中断服务例程调用 xQueueSendToFront() 或 xQueueSendToBack()。应该使用中断安全转换 xQueueSendToFrontFromISR() 和 xQueueSendToBackFromISR() 。这些将在第 6 章中描述。



```groovy
BaseType_t xQueueSendToFront( QueueHandle_t xQueue,
                              const void * pvItemToQueue,
                              TickType_t xTicksToWait );
```

清单 41. xQueueSendToFront() API 函数原型

```groovy
BaseType_t xQueueSendToBack( QueueHandle_t xQueue,
                             const void * pvItemToQueue,
                             TickType_t xTicksToWait );
```

清单 42. xQueueSendToBack() API 函数原型



表 19. xQueueSendToFront() 和 xQueueSendToSendToBack() 函数参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xQueue          | 发送(写入)数据的队列的句柄。队列句柄将从用于创建队列的 xQueueCreate() 调用中返回。 |
| pvItemToQueue   | 指向要复制到队列中的数据的指针。在创建队列时，将设置队列可以容纳的每个项目的大小，因此这多个字节将从 pvItemToQueue 复制到队列存储区域中。 |
| xTicksToWait    | 如果队列已经满了，任务应该保持阻塞状态以等待队列上可用空间的最大时间量。如果 xTicksToWait 为零且队列已满，则 xQueueSendToFront() 和 xQueueSendToBack() 都将立即返回。阻塞时间以滴答周期指定，因此它所表示的绝对时间依赖于滴答频率。宏 pdMS TO TICKS() 可用于将以毫秒为单位的时间转换为以节拍为单位的时间。如果在 FreeRTOSConfig.h 中将 INCLUDE_vTaskSuspend 设置为 1，则将 xTicksToWait 设置为 portMAX_DELAY 将导致任务无限期地等待（没有超时）。 |
| 返回值          | 有两种可能的返回值：1.pdPASS：仅当数据成功发送到队列时，才会返回 pdPASS。如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务可能被置于Blocked状态，等待空间在队列中变为可用，在函数返回之前，但数据已成功写入队列 在阻止时间到期之前。2.errQUEUE_FULL：如果由于队列已满，无法将数据写入队列，将返回errQUEUE_FULL 。如果指定了阻塞时间（xTicksToWait 不为零），则调用任务将被置于阻塞状态以等待另一个任务或中断在队列中腾出空间，但指定的阻塞时间在该状态之前到期。 |

xQueueReceive() 是用来从队列中接收（读取）一个元素。收到的元素将从队列中删除。



切勿从中断服务程序调用 xQueueReceive()。 中断安全 xQueueReceiveFromISR() API 函数在第 6 章中描述。



```verilog
BaseType_t xQueueReceive( QueueHandle_t xQueue,
                          void *const pvBuffer,
                          TickType_t xTicksToWait );
```

清单 43. xQueueReceive() API 函数原型

表 20. xQueueReceive() 函数参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xQueue          | 正在接收（读取）数据的队列句柄。将从用于创建队列的 xQueueCreate() 调用返回队列句柄。 |
| pvBuffer        | 指向要将接收到的数据复制到的内存的指针。在创建队列时设置队列保存的每个数据项的大小。 pvBuffer 指向的内存必须至少足以容纳那么多字节。 |
| xTicksToWait    | 如果队列已经为空，则任务应保持在阻塞状态以等待数据的最长时间在队列中可用。如果 xTicksToWait 为零，那么如果队列已经为空，则 xQueueReceive() 将立即返回。阻塞时间在滴答周期中指定，因此它表示的绝对时间取决于滴答频率。 宏 pdMS_TO_TICKS() 可用于将以毫秒为单位指定的时间转换为刻度中指定的时间。将 xTicksToWait 设置为 portMAX_DELAY 会导致任务无限期地等待（没有超时），前提是 FreeRTOSConfig.h 中的 INCLUDE_vTaskSuspend 设置为 1。 |
| 返回值          | 有两种可能的返回值：1.pdPASS：仅当从队列中成功读取数据时才会返回 pdPASS。 如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务可能被置于阻塞状态，等待数据在队列中可用，但是在阻塞时间到期之前已成功从队列中读取数据。2.errQUEUE_EMPTY：如果由于队列已经为空而无法从队列中读取数据，则将返回errQUEUE_EMPTY。如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务将被置于阻塞状态以等待另一个任务或中断将数据发送到队列，但阻塞时间在该时间之前到期。 |

uxQueueMessagesWaiting() 用于查询当前队列中的项目数。



切勿从中断服务程序调用 uxQueueMessagesWaiting()。 应该在其位置使用中断安全 uxQueueMessagesWaitingFromISR()。



```verilog
UBaseType_t uxQueueMessagesWaiting( QueueHandle_t xQueue );
```

清单 44. uxQueueMessagesWaiting() API 函数原型

表 21. uxQueueMessagesWaiting() 函数参数或返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xQueue          | 正在查询队列的句柄。 将从用于创建队列的 xQueueCreate() 调用返回队列句柄。 |
| 返回值          | 正在查询的队列当前持有的项目数。 如果返回零，则队列为空。    |

### 示例 10. 从队列接收时阻塞

此示例演示了正在创建的队列，从多个任务发送到队列的数据以及从队列中接收的数据。 创建队列以保存 int32_t 类型的数据项。 发送到队列的任务不指定阻塞时间，从队列接收的任务执行。



发送到队列的任务的优先级低于从队列接收的任务的优先级。 这意味着队列永远不应包含多个项目，因为只要数据被发送到队列，接收任务就会解锁，抢占发送任务，并删除数据 - 再次将队列留空。



清单 45 显示了写入队列的任务的实现。 创建此任务的两个实例，一个将值 100 连续写入队列，另一个将值 200 连续写入同一队列。 任务参数用于将这些值传递到每个任务实例中。

```verilog
static void vSenderTask( void *pvParameters )
{
int32_t lValueToSend;
BaseType_t xStatus;

    /* 创建此任务的两个实例，以便通过任务参数传递发送到队列的值 —— 这样每个实例可以使用不同
    的值。创建队列是为了保存 int32_t 类型的值，因此将参数转换为所需的类型。 */
    lValueToSend = ( int32_t ) pvParameters;

    /* 对于大多数任务，这个任务是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 将值发送到队列。

        第一个参数是数据发送到的队列。队列是在调度程序启动之前创建的，因此在此任务开始执行
        之前。

        第二个参数是要发送的数据的地址，在本例中是 lValueToSend 的地址。

        第三个参数是阻塞时间 —— 如果队列已经满了，任务应该保持在阻塞状态，等待队列上的空间
        可用。在这种情况下，未指定块时间，因为队列永远不应包含多个元素，因此永远不会满。*/
        xStatus = xQueueSendToBack( xQueue, &lValueToSend, 0 );

        if( xStatus != pdPASS )
        {
            /* 发送操作无法完成，因为队列已满 —— 这一定是一个错误，因为队列不能包含更多的
            元素 */
            vPrintString( "Could not send to the queue.\r\n" );
        }
    }
}
```

清单 45. 示例 10 中使用的发送任务的实现。



清单 46 显示了从队列接收数据的任务的实现。 接收任务指定块时间为 100 毫秒，因此将进入阻塞状态以等待数据变为可用。 当队列中的数据可用时，它将离开阻塞状态，或者在没有数据可用的情况下，它将离开 100 毫秒。 在此示例中，100 毫秒超时应该永不过期，因为有两个任务连续写入队列。

```verilog
static void vReceiverTask( void *pvParameters )
{
/* 声明将保存从队列接收的值的变量。 */
int32_t lReceivedValue;
BaseType_t xStatus;
const TickType_t xTicksToWait = pdMS_TO_TICKS( 100 );

    /* 此任务也在无限循环中定义。 */
    for( ;; )
    {
        /* 此调用应该始终发现队列为空，因为此任务将立即删除写入队列的任何数据。 */
        if( uxQueueMessagesWaiting( xQueue ) != 0 )
        {
            vPrintString( "Queue should have been empty!\r\n" );
        }

        /* 从队列中接收数据。 

        第一个参数是接收数据的队列。队列在调度程序启动之前创建，因此在此任务第一次运
        行之前创建。 

        第二个参数是将接收到的数据放置到其中的缓冲区。在这种情况下，缓冲区只是具有保存
        接收数据所需大小的变量的地址。

        最后一个参数是阻塞时间如果队列已经为空，任务将保持在阻塞状态等待数据可用的最大
        时间量。 */
        xStatus = xQueueReceive( xQueue, &lReceivedValue, xTicksToWait );

        if( xStatus == pdPASS )
        {
            /* 从队列中成功接收到数据，打印出接收到的值。 */
            vPrintStringAndNumber( "Received = ", lReceivedValue );
        }
        else
        {
            /* 即使在等待了100ms 之后，也没有从队列接收到数据。这一定是一个错误，因为
            发送任务是免费运行的，并且将不断地写入队列。*/
            vPrintString( "Could not receive from the queue.\r\n" );
        }
    }
}
```

清单 46. 示例 10 接受任务的实现



清单 47 包含 main() 函数的定义。 这只是在启动调度程序之前创建队列和三个任务。 创建队列以最多保存五个 int32_t 值，即使设置了任务的优先级，使得队列一次也不会包含多个项目。

```verilog
/* 声明一个类型为 QueueHandle_t 的变量。该变量用于将句柄存储到所有三个任务都访问的队列中。 */
QueueHandle_txQueue;int main( void )
{
    /* 创建队列最多可以容纳5个值，每个值都足够大，可以容纳 int32_t 类型的变量。 */
    xQueue= xQueueCreate( 5, sizeof( int32_t) );if( xQueue != NULL )
    {
        /* 创建将发送到队列的任务的两个实例。任务参数用于传递任务将写入队列的值，因此一个任务
        将持续向队列写入 100，而另一个任务将持续向队列写入 200。这两个任务都在优先级 1 处创
        建。 */
        xTaskCreate( vSenderTask, "Sender1", 1000, ( void * ) 100, 1, NULL );
        xTaskCreate( vSenderTask, "Sender2", 1000, ( void * ) 200, 1, NULL );

        /* 创建将从队列中读取的任务。创建任务的优先级为 2，因此高于发送方任务的优先级。 */
        xTaskCreate( vReceiverTask, "Receiver", 1000, NULL, 2, NULL );

        /* 启动调度程序，以便创建的任务开始执行。 */
        vTaskStartScheduler();
    }
    else
    {
        /* 无法创建队列。 */
    }

    /* 如果一切正常，那么 main() 将永远不会到达这里，因为调度程序现在将运行这些任务。如果
    main() 确实到达这里，那么很可能没有足够的 FreeRTOS 堆内存可用来创建空闲任务。第 2 章
    提供了关于堆内存管理的更多信息。 */
    for( ;; );
}
```

清单 47. 例 10 中 main() 的实现



发送到队列的两个任务都具有相同的优先级。 这导致两个发送任务依次将数据发送到队列。 例 10 中产生的输出如图 32 所示。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009217986-4d4b1766-a395-47f3-b5ea-874aa265b6ed.png)

图 32. 执行示例 10 产生的输出

图 33 展示了执行的顺序

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009217849-e08f6a41-477f-4e96-b904-7f536a748b7f.png)

图 33 .示例 10 执行顺序

## 从多个源接收数据

FreeRTOS 设计中常见的任务是从多个源接收数据，所以接收任务需要知道数据来自何处以确定如何处理数据。 一个简单的设计解决方案是使用单个队列来传输具有数据值和结构域中包含的数据源的结构。 该方案如图 34 所示。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009291854-192cb58f-4eef-407d-85e5-e542e10dc802.png)

图 34. 发送结构到队列的示例场景

图 34 参考：



- 一个以 Data_t 结构创建的队列。这个结构成员允许包含数据值和一个枚举类型来指示一个消息发送到队列。
- 中央的 Controller 任务用于执行主系统功能。 这必须对输入和对队列中与其通信的系统状态的更改作出反应。

- 一个 CAN 总线任务用于封装 CAN 总线接口功能。当 CAN 总线任务接收并解码消息时，它将已解码的消息发送到 Data_t 结构中的 Controller 任务。 传输结构的 eDataID 成员用于让 Controller 任务知道数据是什么 —— 在描述中它是电机速度值。 传输结构的 lDataValue 成员用于让 Controller 任务知道实际的电机速度值。
- 人机界面（HMI）任务用于封装所有 HMI 功能。机器操作员可能以多种方式输入命令和查询值，这些方式必须在 HMI 任务中检测和解释。输入新命令时，HMI 任务将命令以一个 Data_t 的结构发送到 Controller 任务。传输结构的 eDataID 成员用于让 Controller 任务知道数据是什么 —— 在描述中它是一个新的调定点的值。 传递结构的 lDataValue 成员用于让 Controller 任务知道实际调定点的值。



### 示例 11. 发送到队列和发送队列结构时的阻塞

示例 11 与示例 10 类似，但任务优先级相反，因此接收任务的优先级低于发送任务。 此外，队列用于传递结构，而不是整数。



清单 48 显示了示例 11 使用的结构的定义。

```verilog
/* 定义用于标识数据源的枚举类型。 */
typedef enum
{
    eSender1,
    eSender2
} DataSource_t;

/* 定义将在队列上传递的结构类型。 */
typedef struct
{
    uint8_t ucValue;
    DataSource_t eDataSource;
} Data_t;

/* 声明两个将在队列中传递的 Data_t 类型的变量。 */
static const Data_txStructsToSend[ 2 ] = 
{
    { 100, eSender1 }, /* 由 Sender1 使用。 */
    { 200, eSender2 }  /* 由 Sender2 使用。 */
};
```

清单 48. 要在队列上传递的结构的定义，以及由示例使用的两个变量的声明



在示例 10 中，接收任务具有最高优先级，因此队列中永远不会存在多个元素。 这是因为一旦数据被放入队列中，接收任务就会抢占发送任务。 在示例 11 中，发送任务具有更高的优先级，因此队列通常是满的。 这是因为，一旦接收任务从队列中删除了一个项目，它就会被其中一个发送任务抢占，然后立即重新填充队列。 然后，发送任务重新进入阻塞状态，等待空间再次在队列中可用。



清单 49 显示了发送任务的实现。 发送任务指定 100 毫秒的阻塞时间，因此每次队列变满时，它都会进入阻塞状态以等待由可用空间。当队列中有空间可用时，或者没有空间可用的情况下超过 100 毫秒时，它就会离开阻塞状态。在这个例子中，100 毫秒超时应该永不过期，因为接受任务通过从队列中删除元素来不断地腾出空间。

```verilog
static void vSenderTask( void *pvParameters )
{
BaseType_txStatus;
const TickType_t xTicksToWait = pdMS_TO_TICKS( 100 );

    /* 对于大多数任务，这个任务是在一个无限循环中实现的。 */
    for( ;; )
    {
        /* 发送到队列。

        第二个参数是正在发送的结构的地址。地址作为任务参数传入，因此直接使用 pvParameters。 

        第三个参数是阻塞时间 —— 如果队列已经满了，任务应该保持在阻塞状态，等待队列上的空间可用。
        之所以指定阻塞时间，是因为发送任务的优先级高于接收任务，因此预计队列将满。当两个发送任
        务都处于阻塞状态时，接收任务将从队列中删除元素。 */
        xStatus = xQueueSendToBack( xQueue, pvParameters, xTicksToWait );

        if( xStatus != pdPASS )
        {
            /* 即使等待了 100ms，发送操作也无法完成。这一定是一个错误，因为一旦两个发送任务
            都处于阻塞状态，接收任务就应该在队列中留出空间。 */
            vPrintString( "Could not send to the queue.\r\n" );
        }
    }
}
```

清单 49. 示例 11 发送任务的实现



接收任务的优先级最低，所以只有当两个发送任务都处于阻塞状态时，接收任务才会运行。发送任务仅在队列满时才进入阻塞状态，因此接收任务仅在队列满时才会执行。因此，即使没有指定阻塞时间，它也总是期望接收数据。



清单 50 显示了接收任务的实现。

```verilog
static void vReceiverTask( void *pvParameters )
{
/* 声明将保存从队列接收的值的结构。 */
Data_t xReceivedStructure;
BaseType_t xStatus;

    /* 这个任务也是在一个无限循环中定义的。 */
    for( ;; )
    {
        /* 因为它的优先级最低，所以只有当发送任务处于阻塞状态时，该任务才会运行。发送任务只
        会在队列已满时进入阻塞状态，因此该任务总是期望队列中的项数等于队列长度，本例中为 3。*/
        if( uxQueueMessagesWaiting( xQueue ) != 3 )
        {
            vPrintString( "Queue should have been full!\r\n" );
        }

        /* 从队列中接收。

        第二个参数是将接收到的数据放置到其中的缓冲区。在这种情况下，缓冲区只是具有容纳接收结
        构所需大小的变量的地址。

        最后一个参数是阻塞时间 —— 如果队列已经为空，任务将保持在阻塞状态等待数据可用的最长时
        间。在当前情况下，不需要阻塞时间，因为此任务只在队列满时运行。 */
        xStatus = xQueueReceive( xQueue, &xReceivedStructure, 0 );

        if( xStatus == pdPASS )
        {
            /* 从队列中成功接收到数据，打印出接收到的值和值的源。 */
            if( xReceivedStructure.eDataSource== eSender1 )
            {
                vPrintStringAndNumber( "From Sender 1 = ", xReceivedStructure.ucValue );
            }
            else
            {
                vPrintStringAndNumber( "From Sender 2 = ", xReceivedStructure.ucValue );
            }
        }
        else
        {
            /* 队列中没有收到任何东西。这一定是一个错误，因为该任务应该只在队列满时运行。 */
            vPrintString( "Could not receive from the queue.\r\n" );
        }
    }
}
```

清单 50. 示例 11 接收任务的定义

main() 仅比前一个示例略有变化。 创建队列以容纳三个 Data_t 结构，并且发送和接收任务的优先级相反。 main() 的实现如清单 51 所示。

```php
int main( void )
{
    /* 创建队列以容纳最多 3 个 Data_t 类型的结构。 */
    xQueue = xQueueCreate( 3, sizeof( Data_t) );

    if( xQueue != NULL )
    {
        /* 创建将写入队列的任务的两个实例。该参数用于传递任务将写入队列的结构，因此一个任务将持
        续向队列发送 xStructsToSend[0]，而另一个任务将持续发送 xStructsToSend[1]。这两个任
        务都是在优先级 2 创建的，优先级高于接收方的优先级。 */
        xTaskCreate( vSenderTask, "Sender1", 1000, &( xStructsToSend[ 0 ] ), 2, NULL);
        xTaskCreate( vSenderTask, "Sender2", 1000, &( xStructsToSend[ 1 ] ), 2, NULL);

        /* 创建将从队列中读取的任务。创建任务的优先级为 1，因此低于发送方任务的优先级。 */
        xTaskCreate( vReceiverTask, "Receiver", 1000, NULL, 1, NULL );

        /* 启动调度程序，以便创建的任务开始执行。 */
        vTaskStartScheduler();
    }
    else
    {
        /* 无法创建队列。 */
    }

    /* 如果一切正常，那么 main() 将永远不会到达这里，因为调度程序现在将运行这些任务。如果 
    main() 确实到达这里，那么很可能没有足够的堆内存来创建空闲任务。第 2 章提供了关于堆内存管
    理的更多信息。 */
    for( ;; );
}
```

清单 51. 示例 11 main() 的实现



示例 11 生成的输出如图 35 所示。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636010272533-30942d14-dede-422f-a845-deb1f85342e5.png)

图 35. 示例 11 产生的输出

图 36 显示了由于发送任务的优先级高于接收任务的优先级而导致的执行顺序。 表 22 提供了对图 36 的进一步说明，并描述了前四个消息是否来自同一任务。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636010257309-6a9aeb4f-b4f2-4080-b8b5-9f3140764004.png)

图 36. 示例 11 的执行顺序



表 22. 图 36 的关键点

| 时刻 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| t1   | 任务发送方 1 执行并向队列发送 3 个数据项。                   |
| t2   | 队列已满，因此发送方 1 进入阻塞状态，等待下一次发送完成。任务发送方 2 现在是能够运行的最高优先级任务，因此进入运行状态。 |
| t3   | 任务发送者 2 发现队列已经满了，因此进入阻塞状态，等待第一次发送完成。任务接收者现在是能够运行的最高优先级任务，因此进入运行状态。 |
| t4   | 优先级高于接收任务优先级的两个任务正在等待队列中的空间可用，从而导致任务接收者在从队列中删除一个项目后立即被抢占。 任务发送者 1 和发送者 2 具有相同的优先级，因此调度程序选择等待时间最长的任务作为将进入运行状态的任务 —— 在这种情况下是任务发送者 1。 |
| t5   | 任务发送者 1 将另一个数据项发送到队列。 队列中只有一个空间，因此任务发送者 1 进入阻塞状态以等待下一次发送完成。 任务接收器再次是能够运行的最高优先级任务，因此进入运行状态。任务发送者 1 现在已向队列发送了四个项目，任务发送者 2 仍在等待将其第一个项目发送到队列。 |
| t6   | 优先级高于接收任务优先级的两个任务正在等待队列中的空间可用，因此任务接收者一旦从队列中删除了一个项目就会被抢占。 此时发送者 2 等待的时间比发送者 1 长，因此发送者 2 进入运行状态。 |
| t7   | 任务发送者 2 将数据项发送到队列。 队列中只有一个空格，因此发件人 2 进入阻止状态以等待下一次发送完成。 发送者 1 和发送者 2 都在等待队列中的空间可用，因此任务接收者是唯一可以进入运行状态的任务。 |

### 排队指针

如果存储在队列中的数据大小很大，则最好使用队列将指针传输到数据，而不是将数据本身逐字节地复制到队列中。 传输指针在处理时间和创建队列所需的 RAM 量方面都更有效。 但是，在队列指针时，必须特别注意确保：



1. 指向的 RAM 的所有者是明确定义的。通过指针在任务之间共享内存时，必须确保两个任务不会同时修改内存内容，或采取任何其他可能导致内存内容无效或不一致的操作。 理想情况下，只允许发送任务访问存储器，直到指向存储器的指针已经排队，并且在从队列接收到指针之后，只允许接收任务访问存储器。
2. 指向的 RAM 仍然有效。如果指向的内存是动态分配的，或者是从预先分配的缓冲区池中获取的，则完全一个任务应该负责释放内存。 任务完成后，任何任务都不应尝试访问内存。永远不应该使用指针来访问已在任务堆栈上分配的数据。 堆栈帧更改后，数据无效。



举例来说，清单 52，清单 53 和清单 54 演示了如何使用队列从一个任务向另一个任务发送指向缓冲区的指针：



- 清单 52 创建了一个最多可以容纳 5 个指针的队列。
- 清单 53 分配缓冲区，将字符串写入缓冲区，然后将指向缓冲区的指针发送到队列。

- 清单 54 从队列中接收指向缓冲区的指针，然后将包含在缓冲区中的字符串打印出来。

```php
/* 声明 QueueHandle_t 类型的变量以保存正在创建的队列的句柄。 */
QueueHandle_t xPointerQueue;

/* 创建一个最多可容纳 5 个指针的队列，在本例中为字符指针。 */
xPointerQueue = xQueueCreate( 5, sizeof( char * ) );
```

清单 52. 创建一个包含指针的队列

```groovy
/* 获取缓冲区的任务，向缓冲区写入一个字符串，然后将缓冲区的地址发送到清单 52 中创建的队列。 */
void vStringSendingTask( void *pvParameters )
{
char *pcStringToSend;
const size_t xMaxStringLength = 50;
BaseType_t xStringNumber = 0;

    for( ;; )
    {
        /* 获取至少为 xMaxStringLength 字符大的缓冲区。prvGetBuffer() 的实现没有显示，
        它可能从预先分配的缓冲区池中获取缓冲区，或者只是动态地分配缓冲区。 */
        pcStringToSend = ( char * ) prvGetBuffer( xMaxStringLength );

        /* 将字符串写入缓冲区。 */
        snprintf( pcStringToSend, xMaxStringLength, "String number %d\r\n", xStringNumber );

        /* 增加计数器，使字符串在此任务的每次迭代中都不同。 */
        xStringNumber++;

        /* 将缓冲区的地址发送到清单 52 中创建的队列。缓冲区的地址存储在 pcStringToSend 变量中。*/
        xQueueSend( xPointerQueue,     /* 队列的句柄。 */
                    &pcStringToSend,   /* 指向缓冲区的指针的地址。 */
                    portMAX_DELAY );
    }
}
```

清单 53. 使用队列发送指向缓冲区的指针

```groovy
/* 从清单 52 中创建的队列中接收缓冲区地址并写入清单 53 中的任务。缓冲区包含一个字符串，
该字符串被打印出来。 */
void vStringReceivingTask( void *pvParameters )
{
char *pcReceivedString;

    for( ;; )
    {
        /* 接收缓冲区的地址。 */
        xQueueReceive( xPointerQueue,     /* 队列的句柄。 */
                       &pcReceivedString, /* 将缓冲区地址存储在 pcReceivedString 中。 */
                       portMAX_DELAY );   

        /* 缓冲区保存一个字符串，将其打印出来。 */
        vPrintString( pcReceivedString );

        /* 不再需要缓冲区 —— 释放它以便可以释放或重新使用它。 */
        prvReleaseBuffer( pcReceivedString );
    }
}
```

清单 54. 使用队列接收指向缓冲区的指针

### 使用队列发送不同类型和长度的数据

前面几节已经证明了两个强大的设计模式：发送结构到一个队列，发送指针到一个队列。 组合这两个技术就可以允许一个任务使用一个队列接收来自任何数据源的任何数据类型。 FreeRTOS+TCP TCP/IP 协议栈的实现提供了如何这样实现的实际例子。

在自己的任务中运行的 TCP/IP 协议栈必须处理许多来自不同源的事件。 不同的事件类型与不同类型和长度的数据相关联。 在 TCP/IP 任务之外发生的所有事件都由 IPStackEvent_t 类型的结构描述，并发送到队列上的 TCP/IP 任务。 IPStackEvent_t 结构如清单 55 所示，IPStackEvent_t 结构的 pvData 成员是一个指针，可用于直接保存值或指向缓冲区。

```verilog
/* 在 TCP/IP 堆栈中用于识别事件的枚举类型的子集。 */
typedef enum
{
    eNetworkDownEvent = 0, /* 网络接口已丢失，或者需要（重新）连接。 */
    eNetworkRxEvent,       /* 从网络接收到一个数据包。 */
    eTCPAcceptEvent,       /* 调用 FreeRTOS_accept() 接受或等待新客户端。 */

    /* 其他事件类型会显示在此处，但不会显示在此列表中。 */

} eIPEvent_t;

/* 描述事件并在队列中发送到TCP/IP任务的结构。 */
typedef struct IP_TASK_COMMANDS
{
    /* 标识事件的枚举类型。请参见上面的 eIPEvent_t 定义。 */
    eIPEvent_t eEventType;

    /* 可以保存值或指向缓冲区的通用指针。 */
    void *pvData;

} IPStackEvent_t;
```

清单 55. 用于将事件发送到 FreeRTOS+TCP上的 TCP/IP 协议栈任务结构



TCP/IP 事件及其相关数据的示例包括:



- eNetworkRxEvent：数据的分组已经从网络接收到。从网络接收的数据使用类型为 IPStackEvent_t 的结构发送到 TCP/IP 任务，该结构的 eEventType 成员设置为 eNetworkRxEvent，该结构的 pvData 成员用于指向包含接收数据的缓冲区。清单 56 显示了一个伪代码示例。
- eTCPAcceptEvent：套接字是接受或等待来自客户端的连接。接受事件是使用 IPStackEvent_t 类型的结构从调用 FreeRTOS_accept() 的任务发送到 TCP/IP 任务的。该结构的 eEventType 成员设置为 eTCPAcceptEvent，该结构的 pvData 成员设置为接受连接的套接字的句柄。清单 57 显示了一个伪代码示例。

- eNetworkDownEvent：网络需要连接或重新连接。网络中断事件是使用 IPStackEvent_t 类型的结构从网络接口发送到 TCP/IP 任务的。该结构的 eEventType 成员设置为 eNetworkDownEvent。网络关闭事件不与任何数据相关联，因此不使用该结构的 pvData 成员。清单 58 显示了一个伪代码示例。

```verilog
void vSendRxDataToTheTCPTask( NetworkBufferDescriptor_t *pxRxedData )
{
IPStackEvent_t xEventStruct;

    /* 完成 IPStackEvent_t 结构。接收到的数据存储在 pxRxedData。 */
    xEventStruct.eEventType = eNetworkRxEvent;
    xEventStruct.pvData = ( void * ) pxRxedData;

    /* 发送 IPStackEvent_t 结构到 TCP/IP 协议栈。 */
    xSendEventStructToIPTask( &xEventStruct );

}
```

清单 56. 伪代码，显示如何使用 IPStackEvent_t 结构将从网络接收的数据发送到 TCP/IP 任务

```verilog
void vSendAcceptRequestToTheTCPTask( Socket_t xSocket )
{
IPStackEvent_t xEventStruct;

    /* 完成 IPStackEvent_t 结构。 */
    xEventStruct.eEventType = eTCPAcceptEvent;
    xEventStruct.pvData = ( void * ) xSocket;

    /* 发送 IPStackEvent_t 结构以 TCP/IP 任务。*/
    xSendEventStructToIPTask( &xEventStruct );
}
```

清单 57. 伪代码，显示如何使用 IPStackEvent_t 结构发送接受到 TCP/IP 任务的连接的套接字句柄

```verilog
void vSendNetworkDownEventToTheTCPTask(Socket_t xSocket)
{
IPStackEvent_t xEventStruct;

    /* 完成 IPStackEvent_t 结构。 */
    xEventStruct.eEventType = eNetworkDownEvent;
    xEventStruct.pvData = NULL; /* 未使用，但设置为 NULL 保证完整性。 */

    /* 发送 IPStackEvent_t 类型的结构体到 TCP/IP 任务。*/
    xSendEventStructToIPTask( &xEventStruct );
}
```

清单 58. 伪代码，显示如何使用 IPStackEvent_t 结构向 TCP 发送网络关闭事件



清单 59 中显示了在 TCP/IP 任务中接收和处理这些事件的代码。可以看出，从队列接收的 IPStackEvent_t结构的 eEventType 成员用于确定如何解释 pvData 成员。

```php
IPStackEvent_t xReceivedEvent;

    /* 阻止网络事件队列，直到接收到事件，或者 xNextIPSleep 滴答通过而没有接收到事件。
    如果对 xQueueReceive() 的调用返回是因为超时，而不是因为接收到事件，则将 eEventType 
    设置为 eNoEvent。 */
    xReceivedEvent.eEventType = eNoEvent;
    xQueueReceive( xNetworkEventQueue, &xReceivedEvent, xNextIPSleep );

    /* 收到了哪个事件(如果有)？ */
    switch( xReceivedEvent.eEventType )
    {
        case eNetworkDownEvent :
            /* 尝试(重新)建立连接。此事件与任何数据都没有关联。 */
            prvProcessNetworkDownEvent();
            break;
        case eNetworkRxEvent:
            /* 网络接口收到了一个新数据包。指向接收到的数据的指针存储在接收到的 
            IPStackEvent_t 结构的 pvData 成员中。处理接收到的数据。*/
            prvHandleEthernetPacket( ( NetworkBufferDescriptor_t * )( xReceivedEvent.pvData ) );
            break;
        case eTCPAcceptEvent:
            /* 调用了 FreeRTOS_accept() API函数。接受连接的套接字句柄存储在
            接收到的 IPStackEvent_t 结构的 pvData 成员中。 */
            xSocket = ( FreeRTOS_Socket_t * ) ( xReceivedEvent.pvData );
            xTCPCheckNewClient( pxSocket );
            break;

        /* 其他事件类型以相同的方式处理，但在此不显示。 */
    }
```

清单 59. 显示如何接收和处理 IPStackEvent_t 结构的伪代码



## 从多个队列接收

### 队列集

应用程序设计通常需要单个任务来接收不同大小的数据、不同含义的数据以及来自不同来源的数据。上一节演示了如何使用接收结构的单个队列以简洁高效的方式实现这一点。然而，有时应用程序的设计者正在使用限制其设计选择的约束，这就需要对某些数据源使用单独的队列。例如，集成到设计中的第三方代码可能假设存在专用队列。在这种情况下，可以使用 “队列集”。



队列设置允许所有任务从多个队列接收数据，而无需任务依次轮询每个队列以确定哪个队列(如果有)包含数据。



与使用接收结构的单个队列实现相同功能的设计相比，使用队列集从多个源接收数据的设计不那么整洁，效率也较低。因此，建议仅在设计约束绝对必要时才使用队列集。



以下部分描述了如何使用由以下人员设置的队列：



1. 创建队列集。
2. 向集合中添加队列。信号量也可以添加到队列集中。信号量将在本书后面描述。

1. 从队列集中读取数据，以确定队列集中哪些队列包含数据。当作为队列集合成员的队列接收数据时，接收队列的句柄被发送到队列集合，当任务调用从队列集合读取的函数时返回。因此，如果从队列集中返回队列句柄，那么句柄引用的队列就知道包含数据，然后任务就可以直接从队列中读取。



注意：如果队列是队列集的成员，那么不要从队列中读取数据，除非队列的句柄已经首先从队列集中读取。



队列集功能是通过在 FreeRTOSConfig.h 中将 configUSE_QUEUE_SETS 编译时配置常数设置为 1 来启用的。

### xQueueCreateSet() API函数

必须先显式创建队列集，然后才能使用它。



队列集由句柄引用，句柄是 QueueSetHandle_t 类型的变量。xQueueCreateSet() API 函数创建一个队列集，并返回一个引用它创建的队列集的 QueueSetHandle_t。

```verilog
QueueSetHandle_t xQueueCreateSet(const UBaseType_t uxEventQueueLength);
```

清单 60. xQueueCreateSet() 函数原型



表 23. xQueueCreateSet() 参数与返回值

| 参数名             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| uxEventQueueLength | 当队列集合中的一个队列接收数据时，接收队列的句柄被发送到队列集合。 uxEventQueueLength 定义了正在创建的队列集在任何时候可以容纳的队列句柄的最大数量。队列句柄仅在队列集中的队列接收数据时发送给队列集。如果队列已满，则无法接收数据，因此如果队列集中的所有队列都已满，则无法向队列集发送队列句柄。因此，队列集中一次必须容纳的最大项目数是该组中每个队列长度的总和。例如，如果集合中有三个空队列，每个队列的长度为五，那么集合中的队列总共可以在集合中的所有队列都满之前接收十五个项目(三个队列乘以五个项目)。在该示例中，uxEventQueueLength 必须设置为 15，以保证队列集可以接收发送给它的每个项目。信号量也可以添加到队列集中。二进制和计数信号量将在本书后面介绍。为了计算必要的uxEventQueueLength，二进制信号量的长度为 1，计数信号量的长度由信号量的最大计数值给出。作为另一个例子，如果队列集将包含长度为 3 的队列和长度为 1 的二进制信号量，uxEventQueueLength 必须设置为 4 (3+1)。 |
| 返回值             | 如果返回空值，则无法创建队列集，因为空闲操作系统没有足够的堆内存来分配队列集数据结构和存储区域。返回的非空值表示队列集已成功创建。返回值应该存储为创建的队列集的句柄。 |

xQueueAddToSet() 将队列或信号量添加到队列集中。信号量将在本书后面描述。

```verilog
BaseType_t xQueueAddToSet( QueueSetMemberHandle_t xQueueOrSemaphore, 
                           QueueSetHandle_t xQueueSet );
```

清单 61. xQueueAddToSet() API 函数原型



表 24. xQueueAddToSet() 参数与返回值

| 参数名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| xQueueOrSemaphore | 正在添加到队列集中的队列或信号量的句柄。队列句柄和信号量句柄都可以转换为 QueueSetMemberHandle_t 类型。 |
| xQueueSet         | 要添加队列或信号量的队列集的句柄。                           |
| 返回值            | 有两种可能的返回值:1.pdPASS: 只有当队列或信号量成功添加到队列集中时，才会返回 pdPASS。2.pdFAIL: 如果队列或信号量无法添加到队列集中，将返回 pdFAIL。队列和二进制信号量只有在为空时才能添加到集合中。计数信号量只能在其计数为零时添加到集合中。队列和信号量一次只能是一个集合的成员。 |

xQueueSelectFromSet() 从队列集中读取队列句柄。



当作为集合成员的队列或信号量接收数据时，接收队列或信号量的句柄被发送到队列集合，并在任务调用 xQueueSelectFromSet() 时返回。如果从对 xQueueSelectFromSet() 的调用中返回句柄，则句柄引用的队列或信号量已知包含数据，然后调用任务必须直接从队列或信号量中读取。



注意: 除非队列或信号量的句柄是从对 xQueueSelectFromSet() 的调用中首先返回的，否则不要从属于集合成员的队列或信号量中读取数据。每次调用 xQueueSelectFromSet() 返回队列句柄或信号量句柄时，只从队列或信号量中读取一个项目。

```verilog
QueueSetMemberHandle_t xQueueSelectFromSet( QueueSetHandle_t xQueueSet,
                                            const TickType_t xTicksToWait );
```

清单 62. xQueueSelectFromSet() API 函数原型



表 25. xQueueSelectFromSet() 参数与返回值

| 参数名       | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| xQueueSet    | 队列集的句柄，从中接收(读取)队列句柄或信号量句柄。对用于创建队列集的 xQueueCreateSet() 的调用将返回队列集句柄。 |
| xTicksToWait | 如果队列集中的所有队列和信号量都为空，则调用任务应保持在阻塞状态以等待从队列集中接收队列或信号量句柄的最长时间。如果 xTicksToWait 为零，那么如果集合中的所有队列和信号量都为空，xQueueSelectFromSet() 将立即返回。阻塞时间以刻度周期指定，因此它表示的绝对时间取决于刻度频率。宏 pdMS_TO_TICKS() 可用于将以毫秒为单位指定的时间转换为以刻度为单位指定的时间。将 xTicksToWait 设置为 portMAXDELAY 将导致任务无限期等待(无超时)，前提是在 FreeRTOSConfig.h 中将 INCLUDE_vTaskSuspend 设置为 1。 |
| 返回值       | 非空的返回值将是已知包含数据的队列或信号量的句柄。如果指定了阻塞时间(xTicksToWait 不为零)，那么调用任务可能被置于阻塞状态，以等待数据从集合中的队列或信号量变得可用，但是在阻塞时间到期之前，从队列集合中成功读取了句柄。句柄以 QueueSetMemberHandle_t 类型返回，可以转换为 QueueHandle_t 类型或 SemaphoreHandle_t 类型。如果返回值为空，则无法从队列集中读取句柄。如果指定了阻塞时间(xTicksToWait 不为零)，则调用任务将被置于阻塞状态，以等待另一个任务或中断向集合中的一个或多个信号量发送数据，但阻塞时间在此之前已经过期。 |

### 示例12. 使用一个队列集

本示例创建两个发送任务和一个接收任务。发送任务通过两个独立的队列向接收任务发送数据，每个任务一个队列。这两个队列被添加到队列集中，接收任务从队列集中读取以确定这两个队列中的哪一个包含数据。



任务、队列和队列集都是在 main() 中创建的，请参见清单 63 了解其实现。

```verilog
/* 声明两个队列句柄类型的变量。两个队列都被添加到同一个队列集中。 */
static QueueHandle_t xQueue1 = NULL, xQueue2 = NULL;

/* 声明 QueueSetHandle_t 类型的变量。这是将两个队列添加到的队列集。 */
static QueueSetHandle_t xQueueSet = NULL;

int main( void )
{
    /* 创建两个队列，这两个队列都发送字符指针。接收任务的优先级高于发送任务的优先级，
    因此队列中任何时候都不会有一个以上的项目*/
    xQueue1 = xQueueCreate( 1, sizeof( char * ) );
    xQueue2 = xQueueCreate( 1, sizeof( char * ) );

    /* 创建队列集。两个队列将被添加到该组中，每个队列可以包含 1 个项目，因此该队列组一次
    最多只能容纳 2 个队列(每个队列 2 个队列乘以 1 个项目)。 */
    xQueueSet = xQueueCreateSet( 1 * 2 );

    /* 将两个队列添加到集合中。 */
    xQueueAddToSet( xQueue1, xQueueSet );
    xQueueAddToSet( xQueue2, xQueueSet );

    /* 创建发送到队列的任务。 */
    xTaskCreate( vSenderTask1, "Sender1", 1000, NULL, 1, NULL );
    xTaskCreate( vSenderTask2, "Sender2", 1000, NULL, 1, NULL );

    /* 创建从队列集中读取的任务，以确定两个队列中哪个包含数据。 */
    xTaskCreate( vReceiverTask, "Receiver", 1000, NULL, 2, NULL );

    /* 启动调度程序，以便创建的任务开始执行。 */
    vTaskStartScheduler();

    /* 正常情况下，vTaskStartScheduler() 不应该返回，因此下面几行永远不会执行。 */
    for( ;; );
    return 0;
}
```

清单 63. 示例 12 main() 实现



第一个发送任务使用 xQueue1 每 100 毫秒向接收任务发送一个字符指针。第二个发送任务使用 xQueue2每 200 毫秒向接收任务发送一个字符指针。字符指针被设置为指向标识发送任务的字符串。清单 64 显示了两个发送任务的实现。

```verilog
void vSenderTask1( void *pvParameters )
{
const TickType_t xBlockTime = pdMS_TO_TICKS( 100 );
const char * const pcMessage = "Message from vSenderTask1\r\n";

    /* 根据大多数任务，这个任务是在无限循环中实现的。 */
    for( ;; )
    {
        /* 阻塞 100ms. */
        vTaskDelay( xBlockTime );

        /* 将此任务的字符串发送到 xQueue1。没有必要使用阻塞时间，即使队列只能容纳一个
        项目。这是因为从队列中读取的任务的优先级高于该任务的优先级；一旦该任务写入队列，
        它将被从队列中读取的任务占用，因此当对 xQueueSend() 的调用返回时，队列已经再次
        为空。阻止时间设置为 0。 */
        xQueueSend( xQueue1, &pcMessage, 0 );
    }
}
/*-----------------------------------------------------------*/

void vSenderTask2( void *pvParameters )
{
const TickType_t xBlockTime = pdMS_TO_TICKS( 200 );
const char * const pcMessage = "Message from vSenderTask2\r\n";

    /* 根据大多数任务，这个任务是在无限循环中实现的。 */
    for( ;; )
    {
        /* 阻塞 200ms. */
        vTaskDelay( xBlockTime );

        /* 将此任务的字符串发送到 xQueue2。没有必要使用阻塞时间，即使队列只能容纳一个
        项目。这是因为从队列中读取的任务的优先级高于该任务的优先级；一旦该任务写入队列，
        它将被从队列中读取的任务占用，因此当对 xQueueSend() 的调用返回时，队列已经再次
        为空。阻止时间设置为 0。 */
        xQueueSend( xQueue2, &pcMessage, 0 );
    }
}
```

清单 64. 示例 12 中使用的发送任务

发送任务写入的队列是同一队列集的成员。每次任务发送到其中一个队列时，队列的句柄都会被发送到队列集中。接收任务调用 xQueueSelectFromSet() 从队列集中读取队列句柄。接收任务从集合中接收到队列句柄后，它知道接收句柄引用的队列包含数据，因此直接从队列中读取数据。它从队列中读取的数据是指向字符串的指针，接收任务将打印出该字符串。



如果对 xQueueSelectFromSet() 的调用超时，则它将返回空值。在示例 12 中，xQueueSelectFromSet() 是以不确定的块时间调用的，因此永远不会超时，并且只能返回有效的队列句柄。因此，在使用返回值之前，接收任务不需要检查 xQueueSelectFromSet() 是否返回空值。



xQueueSelectFromSet() 只有在句柄引用的队列包含数据时才会返回队列句柄，因此从队列中读取时没有必要使用块时间。



清单 65 显示了接收任务的实现。

```verilog
void vReceiverTask( void *pvParameters )
{
QueueHandle_t xQueueThatContainsData;
char *pcReceivedString;

    /* 根据大多数任务，这个任务是在无限循环中实现的。 */
    for( ;; )
    {
        /* 阻止队列集中的一个队列包含数据。将从 xQueueSelectFromSet() 返回的 
        QueueSetMemberHandle_t 值转换为 QueueHandle_t，因为已知该集的所有成员都是队列
        (队列集不包含任何信号量)。*/
        xQueueThatContainsData = ( QueueHandle_t ) xQueueSelectFromSet( xQueueSet,
                                                                        portMAX_DELAY );

        /* 读取队列集时使用了不确定的阻塞时间，因此除非队列集中的一个队列包含数据，否则 
        xQueueSelectFromSet() 不会返回，并且 xQueueThatContainsData 不能为空。从队
        列中读取。没有必要指定阻塞时间，因为已知队列包含数据。阻止时间设置为 0。 */
        xQueueReceive( xQueueThatContainsData, &pcReceivedString, 0 );

        /* 打印从队列接收到的字符串。 */
        vPrintString( pcReceivedString );
    }
}
```

清单 65. 示例 12 使用的接受任务



图 37 显示了示例 12 产生的输出。可以看出，接收任务从两个发送任务接收字符串。vSenderTask1() 使用的阻塞时间是 vSenderTask2() 使用的块时间的一半，导致 vSenderTask1() 发送的字符串打印频率是 vSenderTask2() 发送的字符串的两倍。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636012137223-8409cebc-07e8-4a46-ab17-ccbebd5462e6.png)

图 37. 执行示例 12 时产生的输出

### 更实际的队列集用例

例 12 展示了一个非常简单的例子；队列集只包含队列，它包含的两个队列都用于发送字符指针。在真实的应用程序中，队列集可能包含队列和信号量，并且队列可能不都包含相同的数据类型。在这种情况下，在使用返回值之前，有必要测试 xQueueSelectFromSet() 返回的值。清单 66 演示了当集合具有以下成员时，如何使用从 xQueueSelectFromSet() 返回的值:



1. 二进制信号量。
2. 从中读取字符指针的队列。

1. 从中读取 uint32_t 值的队列。



清单 66 假设队列和信号量已经被创建并添加到队列集中。

```groovy
/* 从中接收字符指针的队列句柄。 */
QueueHandle_t xCharPointerQueue;

/* 接收 uint32_t 值的队列句柄。 */
QueueHandle_t xUint32tQueue;

/* 二进制信号量的句柄。 */
SemaphoreHandle_t xBinarySemaphore;

/* 两个队列和二进制信号量所属的队列集。 */
QueueSetHandle_t xQueueSet;

void vAMoreRealisticReceiverTask( void *pvParameters )
{
QueueSetMemberHandle_t xHandle;
char *pcReceivedString;
uint32_t ulRecievedValue;
const TickType_t xDelay100ms = pdMS_TO_TICKS( 100 );

    for( ;; )
    {
        /* 在队列集中阻塞最长100毫秒，以等待队列集中的一个成员包含数据。*/
         xHandle = xQueueSelectFromSet( xQueueSet, xDelay100ms);

         /* 测试从 xQueueSelectFromSet() 返回的值。如果返回值为空，则对 
         xQueueSelectFromSet() 的调用超时。如果返回值不为空，则返回值将是集合成员之一的句柄。
         QueueSetMemberHandle_t 值可以转换为 QueueHandle_t 或 SemaphoreHandle_t。是否需
         要显式转换取决于编译器。 */

         if( xHandle == NULL )
         {
             /* 对 xQueueSelectFromSet() 的调用超时。 */
         }
         else if( xHandle == ( QueueSetMemberHandle_t ) xCharPointerQueue )
         {
             /* 对 xQueueSelectFromSet() 的调用返回了接收字符指针的队列句柄。从队列中读取。已
             知队列包含数据，因此使用阻塞时间 0。*/
             xQueueReceive(xCharPointerQueue, &pcReceivedString, 0 );

             /* 这里可以处理接收到的字符指针... */
         }
         else if( xHandle == ( QueueSetMemberHandle_t ) xUint32tQueue )
         {
             /* 对 xQueueSelectFromSet() 的调用返回了接收 uint32_t 类型的队列句柄。从队列中
             读取。已知队列包含数据，因此使用 0 的阻塞时间。 */
             xQueueReceive(xUint32tQueue, &ulRecievedValue, 0 );

             /* 接收到的值可以在这里处理... */
         }
         else if( xHandle == ( QueueSetMemberHandle_t ) xBinarySemaphore )
         {
             /* 对 xQueueSelectFromSet() 的调用返回了二进制信号量的句柄。现在拿旗语。信号量已知
             可用，因此使用 0 的阻塞时间。*/
             xSemaphoreTake(xBinarySemaphore, 0 );

             /* 获取信号量时需要的任何处理都可以在这里执行... */
         }
     }
 }
```

清单 66. 使用包含队列和信号量的队列集



## 使用队列创建邮箱

嵌入式社区内部对术语没有共识，“邮箱”在不同的实时操作系统中将有不同的含义。在这本书里，术语邮箱是指一个长度为1的队列。队列之所以被描述为邮箱，是因为它在应用程序中的使用方式，而不是因为它与队列的功能不同:



- 队列用于将数据从一个任务发送到另一个任务，或者从中断服务例程发送到一个任务。发送方在队列中放置一个条目，接收方从队列中移除该条目。数据通过队列从发送方传递到接收方。
- 邮箱用于保存任何任务或任何中断服务例程都可以读取的数据。数据不会通过邮箱，而是保留在邮箱中，直到被覆盖。发件人会覆盖邮箱中的值。接收器从邮箱中读取该值，但不从邮箱中删除该值。



本章描述了允许队列用作邮箱的两个队列应用编程接口函数。



清单 67 显示了一个被创建用作邮箱的队列。

```c
/* 邮箱可以容纳固定大小的数据项。创建邮箱(队列)时设置数据项的大小。在本例中，创建邮箱是为了
保存示例结构。Example_t 包含一个时间戳，允许邮箱中保存的数据记录邮箱上次更新的时间。本例中使用的
时间戳仅用于演示目的——邮箱可以保存应用程序作者想要的任何数据，并且数据不需要包含时间戳。 */
typedef struct xExampleStructure
{
    TickType_t xTimeStamp;
    uint32_t ulValue;
} Example_t;

/* 邮箱是一个队列，因此它的句柄存储在 QueueHandle_t 类型的变量中。*/
QueueHandle_t xMailbox;

void vAFunction( void )
{
    /* 创建将用作邮箱的队列。队列的长度为1，允许它与xQueueOverwrite()应用编程接口函数一起
    使用，如下所述。 */
    xMailbox = xQueueCreate( 1, sizeof( Example_t ) );
}
```

清单 67. 正在创建用作邮箱的队列

### xQueueOverwrite() API 函数

像 xQueueSendToBack() 应用编程接口函数一样，xQueueOverwrite() 应用编程接口函数向队列发送数据。与 xQueueSendToBack() 不同，如果队列已经满了，那么 xQueueOverwrite() 将覆盖队列中已经存在的数据。

xQueueOverwrite() 只能用于长度为1的队列。这种限制避免了函数的实现在队列已满时任意决定要覆盖队列中的哪个项目。



注意: 永远不要从中断服务例程调用 xQueueOverwrite()。应该使用中断安全版本xQueueOverwriteFromISR() 来代替它。

```verilog
BaseType_t xQueueOverwrite( QueueHandle_t xQueue, const void * pvItemToQueue );
```

清单 68. xQueueOverwrite() API 函数原型



表 26. xQueueOverwrite() 参数和返回值

| 参数名/返回值 | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| xQueue        | 数据被发送(写入)到的队列的句柄。队列句柄将从用于创建队列的 xQueueCreate() 调用中返回。 |
| pvItemToQueue | 指向要复制到队列中的数据的指针。队列可以容纳的每个项目的大小是在创建队列时设置的，因此这些字节将从 pvItemToQueue 复制到队列存储区域。 |
| 返回值        | xQueueOverwrite() 将写入队列，即使队列已满，因此 pdPASS 是唯一可能的返回值。 |

```verilog
void vUpdateMailbox( uint32_t ulNewValue )
{
/* 清单 67 中定义了 Example_t */
Example_t xData;

    /* 新数据写入到Example_t结构。*/
    xData.ulValue = ulNewValue;

    /* 使用RTOS滴答计数作为Example_t结构中存储的时间戳。 */
    xData.xTimeStamp = xTaskGetTickCount();

    /* 发送结构邮箱 - 覆盖已在信箱中的任何数据。 */
    xQueueOverwrite( xMailbox, &xData );
}
```

清单 69. 使用 xQueueOverwrite() API 函数

### xQueuePeek() API 函数

xQueuePeek() 用于从队列中接收(读取)项目，而不从队列中移除项目。xQueuePeek() 从队列头接收数据，而不修改存储在队列中的数据或数据在队列中的存储顺序。



注意: 永远不要从中断服务例程调用 xQueuePeek()。应该使用中断安全版本 xQueuePeekFromISR() 来代替它。

xQueuePeek() 具有与 xQueueReceive() 相同的函数参数和返回值。

```verilog
BaseType_txQueuePeek( QueueHandle_t xQueue,
                      void *const pvBuffer,
                      TickType_t xTicksToWait );
```

清单 70. xQueuePeek() API 函数原型



清单 71 显示了 xQueuePeek() 用于接收发布到清单 69 中邮箱(队列)的项目



```verilog
BaseType_t vReadMailbox( Example_t *pxData )
{
TickType_t xPreviousTimeStamp;
BaseType_t xDataUpdated;

    /* 此函数使用从邮箱收到的最新值更新 Example_t 结构。 记录 *pxData 中已经包含的时间戳记，
    然后再用新数据覆盖它。 */
    xPreviousTimeStamp = pxData->xTimeStamp;

    /* 使用邮箱中包含的数据更新 pxData 指向的 Example_t 结构。 如果在此处使用
    xQueueReceive()，则邮箱将保留为空，然后其他任何任务都无法读取数据。 使用
    xQueuePeek() 而不是 xQueueReceive() 可以确保数据保留在邮箱中。已指定阻止时间，
    因此如果邮箱为空，则调用任务将被置于 阻塞状态以等待邮箱包含数据。 由于使用了无限的
    阻塞时间，因此不必检查从 xQueuePeek() 返回的值，因为 xQueuePeek() 仅在有数据时
    才返回。 */
    xQueuePeek( xMailbox, pxData, portMAX_DELAY );

    /* 返回 pdTRUE 如果从邮箱中读取的值已更新，因为该功能被称为最后。 否则返回 pdFALSE。 */
    if( pxData->xTimeStamp > xPreviousTimeStamp )
    {
        xDataUpdated = pdTRUE;
    }
    else
    {
        xDataUpdated = pdFALSE;
    }

    return xDataUpdated;
}
```

清单71. 使用 xQueuePeek() API 函数



# **软件定时器管理**

## 章节介绍和范围

软件计时器用于调度功能在将来的设定时间执行，或以固定频率定期执行。由软件定时器执行的函数称为软件定时器的回调函数。

软件计时器由FreeRTOS内核实现，并受FreeRTOS内核的控制。它们不需要硬件支持，也与硬件计时器或硬件计数器无关。

请注意，根据FreeRTOS使用创新设计以确保最高效率的理念，软件计时器不会使用任何处理时间，除非软件计时器回调函数实际正在执行。

软件计时器功能是可选的。要包括软件计时器功能，请执行以下操作:

1. 将FreeRTOS源文件FreeRTOS/Source/timers.c构建为项目的一部分 
2. 在FreeRTOSConfig.h中将configUSE_TIMERS设置为1。

### 范围

本章旨在让读者更好地了解以下内容：



- 软件定时器的特性与任务特性的比较。
- RTOS后台任务。

- 计时器命令队列。
- 单次软件定时器和周期性软件定时器之间的区别。

- 如何创建、启动、重置和更改软件计时器的周期。



## 软件定时器回调函数

软件计时器回调函数被实现为C函数。它们唯一的特别之处是它们的原型，它必须返回void，并将软件计时器的句柄作为其唯一的参数。清单72演示了回调函数原型。

```verilog
void ATimerCallback( TimerHandle_t xTimer );
```

清单72.软件计时器回调函数原型



软件计时器回调函数自始至终执行，并以正常方式退出。它们应该保持简短，并且不能进入阻塞状态。



注意：正如将看到的，软件计时器回调函数在启动FreeRTOS调度程序时自动创建的任务的上下文中执行。 因此，软件计时器回调函数决不能调用会导致调用任务进入阻塞状态的FreeRTOS API函数，这一点至关重要。 可以调用xQueueReceive()之类的函数，但前提是该函数的xTicksToWait参数(指定函数的阻塞时间)设置为0。 调用vTaskDelay()之类的函数是不对的，因为调用vTaskDelay()会始终将调用任务置于阻塞状态。

## 软件计时器的属性和状态

### 软件计时器的周期

软件计时器的‘周期’是软件计时器启动和软件计时器的回调函数执行之间的时间

### 单次计时器和自动重新加载计时器

有两种类型的软件计时器：



1. 单次计时器 一旦启动，一次性定时器将只执行其回调函数一次。一次性计时器可以手动重新启动，但不会自行重新启动。
2. 自动重新加载计时器 一旦启动，自动重新加载计时器将在每次到期时重新启动，从而定期执行其回调函数。



图38显示了单次定时器和自动重新加载定时器之间的行为差异。虚线垂直线标记计时中断发生的时间。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705888-f6634080-ebab-4eda-aaca-bf13cc26a399.png)

图 38一次性软件计时器和自动重新加载软件计时器之间的行为差异

参考图38：

- 计时器1 定时器1是具有6个滴答周期的一次性定时器。它在时间t1启动，因此它的回调函数在6个刻度之后，即时间t7执行。由于定时器1是一次性定时器，其回调函数不会再次执行。
- 计时器2 定时器2是具有5个滴答周期的自动重新加载定时器。它在时间t1启动，因此它的回调函数在时间t1之后每5个节拍执行一次。在图38中，这是时间t6、t11和t16。

### 软件计时器状态

软件计时器可以处于以下两种状态之一：

- 休眠：存在休眠的软件计时器，可以由其句柄引用，但不在运行，因此其回调函数将不会执行
- 运行：正在运行的软件定时器，将在自该软件定时器进入运行状态，或自该软件定时器上次被重置以来经过与其周期相等的时间之后执行其回调功能。

图39和图40分别显示了自动重新加载定时器和单次定时器在休眠和运行状态之间可能的转换。这两个图的关键区别在于定时器到期后进入的状态；自动重新加载定时器执行其回调函数，然后重新进入运行状态，一次性定时器执行其回调函数，然后进入休眠状态。

xTimerDelete()接口函数的作用是：删除计时器。可以随时删除计时器。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705902-99800f84-ae67-440b-90c9-3310cac40bce.png)

图39自动重新加载软件计时器状态和转换



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705551-9f9c00ea-ec01-4ccb-be7d-55253a1a817f.png)

图40一次性软件定时器状态和转换



## 软件定时器的上下文

### RTOS守护(计时器服务)任务

所有软件计时器回调函数都在同一RTOS守护进程(或“计时器服务”)任务的上下文中执行[1]。

*[1]. 该任务过去被称为“计时器服务任务”，因为最初它只用于执行软件计时器回调函数。现在同一任务也用于其他目的，因此它被称为“RTOS守护程序任务”的更一般的名称。*

守护程序任务，是在启动调度程序时，自动创建的标准FreeRTOS任务。其优先级和堆栈大小分别由configTIMER_TASK_PRIORITY和configTIMER_TASK_STACK_DEPTH编译时间配置常量设置。这两个常量都在FreeRTOSConfig.h中定义。

软件计时器回调函数不得调用会导致调用任务进入阻塞状态的FreeRTOS API函数，否则将导致守护程序任务进入阻塞状态。

### 计时器命令队列

软件计时器API函数将命令从调用任务发送到称为“计时器命令队列”的队列上的守护程序任务。这如图41所示。命令的例子包括“启动定时器”、“停止定时器”和“重置定时器”。

计时器命令队列是在启动调度程序时自动创建的标准FreeRTOS队列。定时器命令队列的长度由FreeRTOSConfig.h中的configTIMER_QUEUE_LENGTH编译时间配置常量设置。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974388-3c1f11a9-dd13-4d5e-b740-fd915ce0b6a0.png)

图41 软件定时器API函数使用定时器命令队列与RTOS守护程序任务通信

### 守护进程任务调度

守护程序任务与任何其他FreeRTOS任务一样进行调度；当守护程序任务是能够运行的最高优先级任务时，它只会处理命令或执行计时器回调函数。图42和图43演示了configTIMER_TASK_PRIORITY设置如何影响执行模式



图42显示了当守护程序任务的优先级低于调用 xTimerStart() API函数的任务的优先级时的执行模式



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974334-7e306f7b-1f83-4107-85be-dee306722f04.png)

图42 调用xTimerStart()的任务的优先级高于守护程序任务的优先级时的执行模式

参照图42，其中任务1的优先级高于守护程序任务的优先级，并且守护程序任务的优先级高于空闲任务的优先级：



1. t1时刻 ：任务1处于RUNNING状态，守护程序任务处于BLOCKED状态。 守护程序任务将脱离阻塞状态，如果一个命令被发送到计时器命令队列，在这种情况下，它将处理命令。或者如果软件计时器超时，在这种情况下，它将执行软件计时器的回调函数。
2. t2时刻：任务1调用xTimerStart()。 xTimerStart()向计时器命令队列发送命令，使守护程序任务离开阻塞状态。 任务1的优先级高于守护程序任务的优先级，因此守护程序任务不会抢占任务1。 任务1仍处于Running状态，守护程序任务已离开BLOCKED状态，进入READY状态。

1. t3时刻：任务1完成 xTimerStart() API函数的执行。 任务1从函数开始到函数结束执行xTimerStart()，而不离开运行状态。
2. t4时刻：任务1调用导致其进入阻塞状态的API函数。守护程序任务现在是处于就绪状态的最高优先级任务，因此调度程序选择守护程序任务作为进入运行状态的任务。然后，守护程序任务开始处理任务1发送到计时器命令队列的命令。 *注意：正在启动的软件计时器将到期的时间，是从向计时器命令队列发送“启动计时器”命令开始计算的，而不是从守护程序任务从计时器命令队列接收到“启动计时器”命令的时间计算的。*

1. t5时刻：守护程序任务已完成对任务1发送给它的命令的处理，并尝试从计时器命令队列接收更多数据。计时器命令队列为空，因此守护程序任务重新进入阻塞状态。如果将命令发送到计时器命令队列，或者如果软件计时器超时，则守护程序任务将再次离开阻塞状态。空闲任务现在是处于就绪状态的最高优先级任务，因此调度程序选择空闲任务作为要进入运行状态的任务。

图43显示了类似于图42所示的场景，但是这一次守护程序任务的优先级高于调用xTimerStart()的任务的优先级。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974569-c3bbbfc2-83ff-4d93-9db7-9feb390934af.png)

图43 当调用xTimerStart()的任务的优先级低于守护程序任务的优先级时的执行模式



参照图43，其中守护任务的优先级高于任务1的优先级，任务1的优先级高于空闲任务的优先级： 1. t1时刻 和之前一样，任务1正在运行态，守护任务在阻塞态。



1. t2时刻 任务1调用xTimerStart()。 xTimerStart() 向计时器命令队列发送命令，使守护程序任务离开阻塞状态。守护程序任务的优先级高于任务1的优先级，因此调度器选择守护程序任务作为进入运行状态的任务。 任务1在完成执行 xTimerStart() 函数之前被守护程序任务抢占，现在处于就绪状态。守护程序任务开始处理任务1发送到定时器命令队列的命令。
2. t3时刻 守护程序任务已完成对任务1发送给它的命令的处理，并尝试从计时器命令队列接收更多数据。 计时器命令队列为空，因此守护程序任务重新进入阻塞状态。 任务1现在是处于就绪状态的最高优先级任务，因此调度程序选择任务1作为要进入运行状态的任务。

1. t4时刻 任务1在完成执行 xTimerStart() 函数之前被守护程序任务抢占，并且只有在重新进入运行状态后才退出(从)xTimerStart()。
2. t5时刻 任务1调用导致其进入阻塞状态的API函数。空闲任务现在是处于就绪状态的最高优先级任务，因此调度程序选择空闲任务作为要进入运行状态的任务。



在图42所示的场景中，任务1向计时器命令队列发送命令与守护进程任务接收和处理命令之间经过了一段时间。在图43所示的场景中，在Task1从发送命令的函数返回之前，守护进程任务已经接收并处理了Task1发送给它的命令。



发送到计时器命令队列的命令包含时间戳。时间戳用于说明从应用程序任务发送的命令到守护程序任务正在处理的同一命令之间经过的任何时间。例如，如果发送“启动计时器”命令来启动周期为10个滴答的计时器，则时间戳用于确保计时器是在命令发送后10个滴答超时，而不是在命令被守护进程处理之后10个滴答超时。



## 创建和开始一个软件定时器

### xTimerCreate()API函数

FreeRTOS V9.0.0还包括xTimerCreateStatic()函数，该函数分配在编译时静态创建计时器所需的内存：软件计时器必须先显式创建，然后才能使用。



软件计时器由TimerHandle_t类型的变量引用。xTimerCreate()用于创建软件计时器，并返回TimerHandle_t以引用其创建的软件计时器。软件计时器在休眠状态下创建。



可以在调度器运行之前创建软件计时器，也可以在调度器启动后从任务创建软件计时器。第0节介绍了使用的数据类型和命名约定。

```verilog
TimerHandle_t xTimerCreate( const char * const pcTimerName, 
                                TickType_t xTimerPeriodInTicks, 
                                UBaseType_t uxAutoReload, 
                                void * pvTimerID, 
                                TimerCallbackFunction_t pxCallbackFunction );
```

清单73. xTimerCreate()API函数原型



表 27. xTimerCreate()参数和返回值

| 参数/返回值         | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| pcTimerName         | 计时器的描述性名称。FreeRTOS不会以任何方式使用它。它纯粹是作为调试辅助工具而包含的。使用人类可读的名称标识计时器比尝试通过其句柄标识要简单得多。 |
| xTimerPeriodInTicks | 以刻度为单位指定的计时器周期。pdMS_TO_TICKS()宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| uxAutoReload        | 将uxAutoReload设置为pdTRUE以创建自动重新加载计时器。将uxAutoReload设置为pdFALSE以创建一次性计时器 |
| pvTimerID           | 每个软件定时器都有一个ID值。ID是一个空指针，应用程序编写器可以将其用于任何目的。当多个软件计时器使用相同的回调函数时，ID特别有用，因为它可用于提供特定于计时器的存储。本章中的一个示例演示了计时器ID的使用。 pvTimerID设置正在创建的任务的ID的初始值。 |
| pxCallbackFunction  | 软件计时器回调函数仅仅是符合清单72中所示原型的C函数。pxCallbackFunction参数是指向要用作正在创建的软件计时器的回调函数的函数指针(实际上就是函数名)。 |
| 返回值              | 如果返回NULL，则无法创建软件计时器，因为FreeRTOS没有足够的堆内存来分配必要的数据结构。返回的非NULL值表示软件计时器已成功创建。返回值是创建的计时器的句柄。 第2章提供了有关堆内存管理的更多信息。 |

### xTimerStart()API函数

xTimerStart()用于启动处于休眠状态的软件定时器，或重置(重新启动)处于运行状态的软件定时器。xTimerStop()用于停止处于运行状态的软件计时器。停止软件计时器与将计时器转换到休眠状态相同。

可以在调度程序启动之前调用xTimerStart()，但是当这样做时，软件计时器直到调度程序启动时才会实际启动。

注意：切勿从中断服务例程调用xTimerStart()。应该使用中断安全版本xTimerStartFromISR()来代替它。

```verilog
BaseType_t xTimerStart( TimerHandle_t xTimer, TickType_t xTicksToWait );
```

清单74. xTimerStart() API函数原型



表 28. xTimerStart() 参数和返回值

| 参数/返回值  | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| xTimer       | 正在启动或重置的软件计时器的句柄。句柄将从用于创建软件计时器的 xTimerCreate() 调用中返回。 |
| xTicksToWait | xTimerStart() 使用计时器命令队列向守护程序任务发送“启动计时器”命令。xTicksToWait 指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。如果xTicksToWait为零且计时器命令队列已满，则xTimerStart()将立即返回。 阻塞时间以滴答周期指定，因此它表示的绝对时间取决于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位的时间转换为以滴答为单位的时间。 如果FreeRTOSConfig.h中的include_vTaskSuspend设置为1，则。将xTicksToWait设置为portMAX_DELAY将导致调用任务无限期地保持在阻塞状态(没有超时)，以等待定时器命令队列中的空间变得可用。 如果在调度程序启动之前调用xTimerStart()，则会忽略xTicksToWait的值，并且xTimerStart()的行为就像xTicksToWait已设置为零一样。 |
| 返回值       | 有两个可能的返回值 1. pdPASS 只有当“启动定时器”命令成功发送到定时器命令队列时，才会返回pdPASS。 如果守护程序任务的优先级高于调用xTimerStart()的任务的优先级，那么调度程序将确保在xTimerStart()返回之前处理启动命令。这是因为一旦计时器命令队列中有数据，守护进程任务就会抢占调用xTimerStart()的任务。 如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在块时间到期之前，数据已成功写入计时器命令队列。2.pdFALSE 如果由于队列已满而无法将“启动定时器”命令写入定时器命令队列，则将返回pdFALSE。 如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在计时器命令队列中腾出空间，但指定的阻塞时间在此之前已过期。 |

### 示例13. 创建单发和自动重载定时器

此示例创建并启动一个一次性计时器和一个自动重新加载计时器-如清单75所示。

```c
/* 分配给单次和自动重新加载计时器的周期分别为3.333秒和半秒。 */ 
#define mainONE_SHOT_TIMER_PERIOD pdMS_TO_TICKS( 3333 )
#define mainAUTO_RELOAD_TIMER_PERIOD pdMS_TO_TICKS( 500 ) 
int main( void )
 {
 TimerHandle_t xAutoReloadTimer, xOneShotTimer;
 BaseType_t xTimer1Started, xTimer2Started;
    /* 创建一次计时器，将创建的计时器的句柄存储在xOneShotTimer中。*/  
    xOneShotTimer = xTimerCreate( 
    /* 软件计时器的文本名称-未由FreeRTOS使用。*/
    "OneShot",
    /*软件计时器的周期(以滴答为单位)。*/
    mainONE_SHOT_TIMER_PERIOD,
    /* 将uxAutoRealod设置为pdFALSE将创建一次性软件计时器。*/
    pdFALSE,
    /* 此示例不使用计时器ID。*/
    0,
    /* 要由正在创建的软件计时器使用的回调函数。*/
    prvOneShotTimerCallback );

    /* 创建自动重新加载计时器，将创建的计时器的句柄存储在xAutoReloadTimer中。*/ 
    xAutoReloadTimer = xTimerCreate( 
    /* 软件计时器的文本名称-未由FreeRTOS使用。 */
    "AutoReload",
    /* 软件计时器的周期(以滴答为单位)。*/
    mainAUTO_RELOAD_TIMER_PERIOD,
    /* 将uxAutoRealod设置为pdTRUE将创建自动重新加载计时器。*/
    pdTRUE, 
    /* 此示例不使用计时器ID。 */
    0,
    /* 要由正在创建的软件计时器使用的回调函数。*/
    prvAutoReloadTimerCallback );

    /* 检查软件计时器是否已创建。*/
    if( ( xOneShotTimer != NULL ) && (xAutoReloadTimer != NULL ) ) 
    {
    /* 阻塞时间设为0(无块时间)启动软件计时器。调度程序尚未启动，因此此处指定的任何块时间都将被忽略。
 */
    xTimer1Started = xTimerStart( xOneShotTimer, 0 );
    xTimer2Started = xTimerStart( xAutoReloadTimer, 0);
    /* xTimerStart()的实现使用计时器命令队列，如果计时器命令队列已满，xTimerStart()将失败。计时器服务任务在调度程序启动之前不会创建，因此发送到命令队列的所有命令都将保留在队列中，直到调度程序启动之后。检查传递的两个xTimerStart()调用。*/
    if( ( xTimer1Started == pdPASS ) && (xTimer2Started == pdPASS ) ) 
    { /* Start the scheduler. */
    vTaskStartScheduler(); 
    }
  } 
    /*一如既往，这条线不应该达到。 */ 
    for( ;; );
}
```

清单75. 创建并启动示例13中使用的计时器



计时器的回调函数在每次被调用时只打印一条消息。清单76中显示了一次性计时器回调函数的实现。自动重新加载计时器回调函数的实现如清单77所示。

```verilog
static void prvOneShotTimerCallback( TimerHandle_t xTimer )
{
TickType_t xTimeNow;
    /* 获取当前的滴答计数。 */
    xTimeNow = xTaskGetTickCount();

    /* 输出一个字符串以显示执行回调的时间*/
    vPrintStringAndNumber( "One-shot timer callback executing", xTimeNow );

    /* 文件范围变量。 */
    ulCallCount++;
 }
```

清单76. 示例13中的一次性定时器使用的回调函数

```verilog
static void prvAutoReloadTimerCallback( TimerHandle_t xTimer )
{
TickType_t xTimeNow;
    /* 获取当前的滴答计数。 */
    xTimeNow = uxTaskGetTickCount();

     /* 输出一个字符串以显示执行回调的时间*/
   vPrintStringAndNumber( "Auto-reload timer callback executing", xTimeNow );

  ulCallCount++;
 }
```

清单77. 示例13中的自动重新加载计时器使用的回调函数



执行此示例将生成如图44所示的输出。图44显示了自动重新加载计时器的回调函数以500个滴答的固定周期执行(清单75中的mainAUTO_RELOAD_TIMER_PERIOD设置为500)，当滴答计数为3333时，一次性计时器的回调函数只执行一次(清单75中的MainOne_Shot_Timer_Period设置为3333)。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636016723478-770313b8-648a-490e-8a2d-10d63175c4eb.png)

图44 执行示例13时产生的输出



## 定时器ID

每个软件计时器都有一个ID，它是应用程序编写器可以出于任何目的使用的标记值。ID存储在空指针(void*)中，因此可以直接存储整数值、指向任何其他对象或用作函数指针。

创建软件计时器时会为ID分配初始值-之后可以使用vTimerSetTimerID() API函数更新ID，并使用pvTimerGetTimerID() API函数进行查询。

与其他软件计时器API函数不同，vTimerSetTimerID()和pvTimerGetTimerID()直接访问软件计时器-它们不向计时器命令队列发送命令。

### vTimerSetTimerID()API函数

```verilog
void vTimerSetTimerID( const TimerHandle_t xTimer, void *pvNewID );
```

清单78. vTimerSetTimerID() API函数原型



表 29. vTimerSetTimerID() 参数

| 参数/返回值 | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| xTimer      | 使用新ID值更新的软件计时器的句柄。 句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| pvNewID     | 将设置软件计时器ID的值。                                     |

### pvTimerGetTimerID()API函数

```verilog
void *pvTimerGetTimerID( TimerHandle_t xTimer );
```

清单79 pvTimerGetTimerID() API函数原型



表 30. pvTimerGetTimerID()参数和返回值

| 参数/返回值 | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| xTimer      | 正在查询的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| 返回值      | 正在查询的软件计时器的ID。                                   |

### 示例14.使用回调函数参数和软件定时器ID

可以将相同的回调函数分配给多个软件计时器。完成后，回调函数参数用于确定哪个软件计时器过期。



示例13使用了两个单独的回调函数；一个回调函数由OneShot计时器使用，另一个回调函数由自动重新加载计时器使用。示例14创建与示例13创建的功能类似的功能，但将单个回调函数分配给两个软件计时器。



示例14使用的main()函数与示例13使用的main()函数几乎相同，唯一的区别是创建软件计时器的位置。清单80显示了这种差异，其中prvTimerCallback()用作两个计时器的回调函数。

```verilog
/* 创建一次计时器软件计时器，将句柄存储在xOneShotTimer中。*/
xOneShotTimer = xTimerCreate( "OneShot",
                                          mainONE_SHOT_TIMER_PERIOD,
                                          pdFALSE,
                                          /* 计时器ID初始化为0*/
                                          0,
                                         /* 两个计时器都使用prvTimerCallback()。*/
                                         prvTimerCallback );


/* 创建自动重新加载软件计时器，将句柄存储在xAutoReloadTimer中*/
xAutoReloadTimer = xTimerCreate( "AutoReload",
                                               mainAUTO_RELOAD_TIMER_PERIOD,
                                               pdTRUE,
                                               /* 计时器的ID初始化为0。 */
                                               0,
                                               /* 两个计时器都使用prvTimerCallback()。*/
                                               prvTimerCallback );
```

清单80. 创建示例14中使用的计时器



prvTimerCallback()将在任一计时器超时时执行。prvTimerCallback()的实现使用函数的参数来确定调用它是因为一次性计时器过期，还是因为自动重新加载计时器过期。



prvTimerCallback()还演示了如何将软件计时器ID用作特定于计时器的存储；每个软件计时器在其自己的ID中保存其过期次数的计数，并且自动重新加载计时器在第五次执行时使用该计数停止自身。



prvTimerCallback()的实现如清单79所示

```verilog
static void prvTimerCallback( TimerHandle_t xTimer )
{
TickType_t xTimeNow;
uint32_t ulExecutionCount;

        /* 此软件计时器过期次数的计数存储在计时器的ID中。获取ID，将其递增，然后将其另存为新的ID            值。该ID是一个空指针，因此被强制转换为uint32_t。*/
        ulExecutionCount = ( uint32_t ) pvTimerGetTimerID( xTimer );
        ulExecutionCount++;
        vTimerSetTimerID( xTimer, ( void * ) ulExecutionCount );

        /*获取当前的计时次数。*/
        xTimeNow = xTaskGetTickCount();

        /*创建定时器时，单次定时器的句柄存储在xOneShotTimer中。将传入此函数的句柄与                         xOneShotTimer进行比较，以确定是一次性计时器还是自动重新加载计时器过期，然后输出一个          字符串以显示执行回调的时间。*/
        if( xTimer == xOneShotTimer )
        {
            vPrintStringAndNumber( "One-shot timer callback executing", xTimeNow );
        }
        else
        {
            /*xTimer不等于xOneShotTimer，所以一定是自动重新加载计时器过期，调用了回掉函数。*/
            vPrintStringAndNumber( "Auto-reload timer callback executing", xTimeNow );

            if( ulExecutionCount == 5 )
            {
                /*自动重新加载计时器执行5次后停止。此回调函数在RTOS守护程序任务的上下文中执行，                     因此不能调用任何可能将守护程序任务置于阻塞状态的函数。因此，使用块时间0。*/

                xTimerStop( xTimer, 0 );
             }
        }
 }
```

清单81. 示例14中使用的计时器回调函数



示例14产生的输出如图45所示。可以看到，自动重新加载计时器只执行五次。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636018371894-d9394688-5f72-4013-a729-fe5b572f9b11.png)

图45执行示例14时产生的输出



## 更改定时器的周期

每个官方FreeRTOS端口都提供了一个或多个示例项目。大多数示例项目都是在运行中不断自检，LED用于提供项目状态的可视反馈；如果自检总是通过，则LED缓慢闪烁，如果自检失败，则LED快速闪烁。

一些示例项目在任务中执行自检，并使用vTaskDelay()函数控制LED的切换速率。其他示例项目在软件计时器回调函数中执行自检，并使用计时器的周期来控制LED的切换速率。

### xTimerChangePeriod()API函数

使用xTimerChangePeriod()函数更改软件计时器的周期。



如果xTimerChangePeriod()用于更改已在运行的计时器的周期，则该计时器将使用新的周期值重新计算其到期时间。重新计算的过期时间是相对于调用xTimerChangePeriod()的时间，而不是相对于最初启动计时器的时间。



如果使用xTimerChangePeriod()来更改处于休眠状态的计时器(未运行的计时器)的周期，则计时器将计算到期时间，并转换到运行状态(计时器将开始运行)。



注意：切勿从中断服务例程调用xTimerChangePeriod()。应该使用中断安全版本xTimerChangePerodFromISR()来代替它。

```verilog
BaseType_t xTimerChangePeriod( TimerHandle_t xTimer,  
                               TickType_t xNewTimerPeriodInTicks, 
                               TickType_t xTicksToWait );
```

清单82. xTimerChangePeriod() API函数原型



表 31 xTimerChangePeriod()参数和返回值

| 参数/返回值         | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| xTimer              | 需要更新的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| xTimerPeriodInTicks | 软件计时器的新周期，以刻度为单位指定。pdMS_TO_TICKS() c宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| xTicksToWait        | xTimerChangePeriod()使用计时器命令队列向守护程序任务发送“change Period”命令。xTicksToWait指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。如果xTicksToWait为零且计时器命令队列已满，则xTimerChangePeriod()将立即返回。 |
| 返回值              | 有两个可能的返回值 1.pdPASS 只有当数据成功发送到计时器命令队列时，才会返回pdPASS。如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在阻塞时间到期之前，数据已成功写入计时器命令队列。2.pdFALSE如果由于队列已满而无法将‘Change Period’命令写入计时器命令队列，则将返回pdFALSE。如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在队列中腾出空间，但指定的阻塞时间在此之前已过期。 |

清单83 展示了包含自检的FreeRTOS例程是怎么在软件定时器的回调函数中使用 xTimerChangePeriod()在自检失败时提高LED闪烁速度的。执行自检的软件定时器被称为“检查定时器”。

```verilog
/* 检查计时器的创建周期为3000毫秒，导致LED每3秒切换一次。如果自检功能检测到意外状态，则检查计时器的周期将更改为仅200毫秒，从而导致更快的切换速率。*/ 
const TickType_t xHealthyTimerPeriod = pdMS_TO_TICKS( 3000 ); 
const TickType_t xErrorTimerPeriod = pdMS_TO_TICKS( 200 ); 

/* 检查计时器使用的回调函数。 */ 
static void prvCheckTimerCallbackFunction( TimerHandle_t xTimer ) 
{ 
static BaseType_t xErrorDetected = pdFALSE; 

    if( xErrorDetected == pdFALSE ) 
    { 
        /* 尚未检测到任何错误。再次运行自检功能。该函数要求示例创建的每个任务报告其自己的状态，并检查所有任务是否实际上仍在运行(因此能够正确报告其状态)。*/ 
        if( CheckTasksAreRunningWithoutError() == pdFAIL ) 
        { 
            /*一个或多个任务报告意外状态，可能发生了错误。
            减少检查计时器的周期以提高此回调函数的执行速率，这样做还可以提高LED的切换速率。
            此回调函数在RTOS守护进程任务的上下文中执行，因此使用阻塞时间0来确保守护进程任务永             远不会进入阻塞状态。*/ 
            xTimerChangePeriod( xTimer,            /* 正在更新的计时器。*/ 
                                xErrorTimerPeriod, /* 计时器的新周期*/ 
                                0 );               /* 发送此命令时不要阻塞。 */ 
        } 

        /* 锁定已检测到错误。 */ 
        xErrorDetected = pdTRUE; 
    } 

    /* 切换LED。LED切换的速率取决于调用此函数的频率，该频率由检查定时器的周期确定。如果CheckTasksAreRunningWithoutError()曾经返回pdFAIL，则计时器的周期将从3000ms减少到200ms。*/ 
    ToggleLED(); 
}
```

清单83.使用xTimerChangePeriod()



## 重置一个定时器

重置软件计时器意味着重新启动计时器；计时器的超时时间相对于计时器重置的时间被重新计算，而不是根据计时器最初启动的时间。图46演示了这一点，它显示了一个以6为周期的计时器，在最终到期并执行他的回调函数之前，重启两次的过程。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636019663078-040714ed-3f00-4f97-b1c8-1b9b9a3fd5cf.png)

图46. 启动和重置周期为6个滴答的软件计时器

参考图46：

- 定时器1在时间t1启动。它的周期为6，因此它执行回调函数的时间最初计算为T7，即启动后的6个滴答。
- 定时器1在到达时间T7之前，也就是在它到期并执行其回调函数之前被重置。定时器1在时间t5被重置，因此它将执行其回调函数的时间被重新计算为t11，即它被重置后的6个滴答。

- 定时器1在时间t11之前再次重置，因此在其到期并执行其回调函数之前再次复位。定时器1在时间t9被重置，因此它将执行其回调函数的时间被重新计算为t15，这是它上次被重置后的6个滴答。
- 定时器1不会再次复位，因此它在时间t15到期，并且相应地执行其回调函数。

### xTimerReset() API函数

使用xTimerReset()API函数重置计时器。

xTimerReset()还可用于启动处于休眠状态的计时器。

注意：切勿从中断服务例程调用xTimerReset()。应该使用中断安全版本xTimerResetFromISR()来代替它。

```verilog
BaseType_t xTimerReset( TimerHandle_t xTimer, TickType_t xTicksToWait );
```

清单84. xTimerReset() API函数原型



表 32. xTimerReset()参数和返回值

| 参数/返回值         | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| xTimer              | 需要更新的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| xTimerPeriodInTicks | 软件计时器的新周期，以刻度为单位指定。pdms_to_ticks()宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| xTicksToWait        | xTimerChangePeriod()使用计时器命令队列向守护程序任务发送“重置”命令。xTicksToWait指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。如果xTicksToWait为零且计时器命令队列已满，则xTimerReset()将立即返回。如果FreeRTOSConfig.h中的include_vTaskSuspend设置为1，则将xTicksToWait设置为portMAX_DELAY将导致调用任务无限期地保持阻塞状态(没有超时)，以等待计时器命令队列中的空间变得可用。 |
| 返回值              | 有两种可能的返回值：pdPASS 只有当数据成功发送到计时器命令队列时，才会返回pdPASS。 如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在阻塞时间到期之前，数据已成功写入计时器命令队列。pdFALSE 如果由于队列已满而无法将‘RESET’命令写入定时器命令队列，则将返回pdFALSE。 如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在队列中腾出空间，但指定的阻塞时间在此之前已过期。 |

### 示例15.重置软件计时器

此示例模拟手机上的背光行为。背光：



- 按下某个键时打开。
- 如果在特定时间段内按下更多键，则保持打开状态。

- 如果在特定时间段内没有按下键，则自动关闭。



使用一次性软件计时器来实现此行为：



- 按下按键时打开[模拟]背光，在软件计时器的回调函数中关闭[模拟]背光。
- 每次按键按下，软件定时器重置。

- 因此，需要按下按键防止背光熄灭的时间等于软件定时器的周期；如果在计时器到期之前没有通过按键重置软件计时器，则执行计时器的回调功能，并且关闭背光。



xSimulatedBacklightOn变量保存背光状态。xSimulatedBacklightOn设置为pdTRUE表示背光打开，设置为pdFALSE表示背光关闭。



软件计时器回调函数如清单85所示。

```verilog
static void prvBacklightTimerCallback( TimerHandle_t xTimer )
{ 
    TickType_t xTimeNow = xTaskGetTickCount();

         /* 背光计时器超时，关闭背光。*/
        xSimulatedBacklightOn = pdFALSE;

         /*打印背光关闭的时间。*/
         vPrintStringAndNumber( "Timer expired, turning backlight OFF at time\t\t", xTimeNow); 
}
```

清单85. 示例15中使用的一次性计时器的回调函数



示例15创建一个任务来轮询键盘[1]。清单86显示了该任务，但是出于下一段中描述的原因，清单86并不是最佳设计的代表。



*[1].打印到Windows控制台和从Windows控制台读取密钥都会导致执行Windows系统调用。Windows系统调用，包括使用Windows控制台、磁盘或TCP/IP堆栈，可能会对FreeRTOS Windows端口的行为产生不利影响，通常应该避免。*



使用FreeRTOS允许您的应用程序是事件驱动的。事件驱动设计非常高效地使用处理时间，因为处理时间仅在事件已发生时使用，并且处理时间不会浪费在轮询尚未发生的事件上。实施例15不能由事件驱动，因为在使用FreeRTOS Windows端口时处理键盘中断是不切实际的，因此必须使用效率低得多的轮询技术。如果清单86是一个中断服务例程，那么将使用xTimerResetFromISR()代替xTimerReset()。

```verilog
static void vKeyHitTask( void *pvParameters ) 
{ 
const TickType_t xShortDelay = pdMS_TO_TICKS( 50 );
TickType_t xTimeNow;

    vPrintString( "Press a key to turn the backlight on.\r\n" ); 

    /* 理想情况下，应用程序应该是事件驱动的，并使用中断来处理按键。在使用FreeRTOS Windows端口时使用键盘中断是不切实际的，因此此任务用于轮询按键。*/
    for( ;; )
    {
        /* 按键了吗？*/ 
        if( _kbhit() != 0 ) 
        {
            /* 已按下一个键。记录时间。 */ 
            xTimeNow = xTaskGetTickCount();

            if( xSimulatedBacklightOn == pdFALSE )
            {
                /* 背光关闭了，所以打开它并打印打开的时间。 */ 
                xSimulatedBacklightOn = pdTRUE; vPrintStringAndNumber(
                                            "Key pressed, turning backlight ON at time\t\t", xTimeNow );
            }
            else
            { 
                /* 背光已经打开，因此打印一条消息，说明计时器即将重置以及重置的时间。*/
                vPrintStringAndNumber(
                                    "Key pressed, resetting software timer at time\t\t", xTimeNow ); 
             } 

             /* 重置软件计时器。如果之前关闭了背光，则此调用将启动计时器。如果背光先前处于打开状态，则此调用将重新启动计时器。真实的应用程序可以读取中断中的按键。如果此函数是中断服务例程，则必须使用xTimerResetFromISR()而不是xTimerReset()。*/
             xTimerReset( xBacklightTimer, xShortDelay ); 

             /* 读取并丢弃按下的键-这在这个简单的示例中不是必需的。*/ 
             ( void ) _getch(); 
           }
      }
}
```

清单86. 示例15中用于重置软件计时器的任务



执行示例15时产生的输出如图47所示。参考图47：



- 第一次按键发生在滴答计数为812的时候。当时打开了背光，启动了一次计时器。
- 当滴答计数为1813、3114、4015和5016时，出现了进一步的按键。所有这些按键都会导致计时器在计时器到期之前被重置。

- 计时器在滴答计数为10016时超时。当时背光是关着的。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636020021564-3b9b3a22-9894-4fdd-a45e-7891522494d2.png)

图47执行示例15时产生的输出

在图47中可以看到，计时器有5000个滴答的周期；在上次按下一个键之后恰好5000个滴答地关闭了背光，所以在最后一次重置计时器之后有5000个滴答。



# **中断管理**



## 章节介绍和范围

### 事件

嵌入式实时系统必须对源自环境的事件做出响应。例如，到达以太网外设的数据包（事件）可能需要传递到 TCP/IP 堆栈进行处理（操作）。非琐碎的系统必须服务于来自多个源的事件，所有这些源都有不同的处理开销和响应时间要求。在每种情况下，都必须对最佳事件处理实施策略作出判断：



1. 如何检测事件？通常使用中断，但也可以轮询输入。
2. 当使用中断时，中断服务程序（ISR）内部应执行多少处理，外部应执行多少处理？通常希望每个 ISR尽可能短。

1. 如何将事件与主（非 ISR）代码通信，以及如何构造此代码以最佳地适应潜在异步事件的处理？



FreeRTOS 不会对应用程序设计者强加任何特定的事件处理策略，但确实提供了允许以简单且可维护的方式实施所选策略的功能。



重要的是区分任务的优先级和中断的优先级：



- 任务是与运行 FreeRTOS 的硬件无关的软件功能。 应用程序编写者在软件中分配任务的优先级，软件算法（调度程序）决定哪个任务将处于运行状态。
- 尽管是用软件编写的，但中断服务程序是硬件特性，因为硬件控制将运行哪个中断服务程序以及何时运行。 任务只有在没有 ISR 运行时才会运行，所以最低优先级的中断会中断最高优先级的任务，任务没有办法抢占 ISR。



所有运行FreeRTOS的体系结构都能够处理中断，但有关中断条目和中断优先级分配的细节因体系结构而异。

### 范围

本章旨在让读者更好地理解：



- 哪些 FreeRTOS API 函数可以在中断服务程序中使用。
- 将中断处理推迟到任务的方法。

- 如何创建和使用二进制信号量和计数信号量。
- 二进制和计数信号量之间的区别。

- 如何使用队列将数据传入和传出中断服务程序。
- 一些 FreeRTOS 移植时可用的中断嵌套模型。





## 从 ISR 使用 FreeRTOS API

### 中断安全 API

通常需要使用来自中断服务函数 (ISR) 的 FreeRTOS API 函数提供的功能，但许多 FreeRTOS API 函数执行在 ISR 内无效的操作——其中最值得注意的是放置调用 API函数进入Blocked状态； 如果从 ISR 调用 API 函数，则它不是从任务调用的，因此没有调用任务可以置于阻塞状态。 FreeRTOS 通过提供一些 API 函数的两个版本来解决这个问题； 一种用于任务的版本，一种用于 ISR 的版本。 旨在从 ISR 使用的函数在其名称后加上“FromISR”。



注意：切勿从 ISR 调用名称中不含“FromISR”的 FreeRTOS API 函数。

### 使用单独的中断安全 API 的好处

具有用于中断的单独 API 可以使任务代码更高效，ISR 代码更有效，并且中断输入更简单。 要了解为什么，考虑另一种解决方案，它将提供一个API函数的单一版本，可以从一个任务和一个ISR调用。如果可以从任务和ISR调用相同版本的API函数，则：



- API 函数需要额外的逻辑来确定它们是从任务还是从 ISR 调用的。 额外的逻辑会在函数中引入新的路径，使函数变得更长、更复杂、更难测试。
- 当从任务调用函数时，某些 API 函数参数将过时，而当从 ISR 调用函数时，其他函数参数将过时。

- 每个 FreeRTOS 移植都需要提供一种机制来确定执行上下文（任务或 ISR）。
- 难以确定执行上下文（任务或 ISR）的架构将需要额外的、浪费的、使用更复杂的非标准中断入口代码，允许软件提供执行上下文。

### 使用单独的中断安全 API 的缺点

拥有两个版本的某些 API 函数可以提高任务和 ISR 的效率，但会带来新的问题；有时需要从任务和 ISR 调用不属于 FreeRTOS API 的函数，但需要使用 FreeRTOS API。



这通常只有在集成第三方代码时才会出现问题，因为这是软件设计不受应用程序编写者控制的唯一时候。 如果这确实成为一个问题，那么可以使用以下技术之一来克服该问题：



1. 将中断处理推迟到任务，因此 API 函数只能从任务的上下文中调用。
2. 如果您使用的是支持中断嵌套的 FreeRTOS 移植，则使用以 “FromISR” 结尾的 API 函数版本，因为该版本可以从任务和 ISR 中调用（反之则不然，不以 “FromISR” 结尾的 API 函数 不能从 ISR 调用“FromISR”）。

1. 第三方代码通常包括一个 RTOS 抽象层，可以实现该抽象层来测试调用函数的上下文（任务或中断），然后调用适合该上下文的 API 函数。

### xHigherPriorityTaskWoken 参数

本节介绍了 xHigherPriorityTaskWoken 参数的概念。 如果您尚未完全理解本节，请不要担心，因为以下部分提供实际示例。



如果上下文切换是由中断执行的，那么中断退出时运行的任务可能与进入中断时运行的任务不同——中断将中断一个任务，但返回到另一个任务。



一些 FreeRTOS API 函数可以将任务从阻塞状态移动到就绪状态。 这已经在诸如 xQueueSendToBack() 之类的函数中看到了，如果有一个任务在阻塞状态等待数据在主题队列上可用，它将解除对任务的阻塞。



如果被 FreeRTOS API 函数解除阻塞的任务的优先级高于处于运行状态的任务的优先级，那么根据 FreeRTOS 调度策略，应该切换到更高优先级的任务。 实际切换到更高优先级任务的时间取决于调用 API 函数的上下文：



- 如果从任务中调用 API 函数



如果在 FreeRTOSConfig.h 中将 configUSE_PREEMPTION 设置为 1，那么在 API 函数中会自动切换到更高优先级的任务——所以在 API 函数退出之前。 这已经在图 43 中看到，其中写入定时器命令队列导致在写入命令队列的函数退出之前切换到 RTOS 守护程序任务。



- 如果从中断调用 API 函数



中断内不会自动切换到更高优先级的任务。 相反，设置了一个变量来通知应用程序编写者应该执行上下文切换。 中断安全 API 函数（以 “FromISR” 结尾的函数）有一个名为 pxHigherPriorityTaskWoken 的指针参数，用于此目的。



如果应该执行上下文切换，则中断安全 API 函数会将 *pxHigherPriorityTaskWoken 设置为 pdTRUE。 为了能够检测到这种情况，pxHigherPriorityTaskWoken 指向的变量在第一次使用之前必须初始化为 pdFALSE。



如果应用程序编写者选择不从 ISR 请求上下文切换，则更高优先级的任务将保持就绪状态，直到调度程序下次运行——在最坏的情况下将在下一次滴答中断期间。



FreeRTOS API 函数只能将 *pxHighPriorityTaskWoken 设置为 pdTRUE。 如果一个 ISR 调用了多个 FreeRTOS API 函数，那么可以在每个 API 函数调用中传递相同的变量作为 pxHigherPriorityTaskWoken 参数，并且该变量只需要在第一次使用之前初始化为 pdFALSE。



在 API 函数的中断安全版本中，上下文切换不会自动发生有几个原因：



1. 避免不必要的上下文切换：在任务需要执行任何处理之前，中断可能会执行多次。 例如，考虑一个任务处理一个由中断驱动的 UART 接收到的字符串的场景； 每次接收到一个字符时，UART ISR 都切换到任务是一种浪费，因为任务只有在接收到完整字符串后才能执行。



1. 控制执行序列：中断可能偶尔发生，而且时间不可预测。 专业的 FreeRTOS 用户可能希望暂时避免在其应用程序的特定点不可预测地切换到不同的任务——尽管这也可以使用 FreeRTOS 调度程序锁定机制来实现。
2. 可移植性：它是可以跨所有 FreeRTOS 移植使用的最简单的机制。



1. 效率：面向较小处理器架构的端口仅允许在 ISR 的最后请求上下文切换，而消除该限制将需要额外且更复杂的代码。 它还允许在同一 ISR 内多次调用 FreeRTOS API 函数，而不会在同一 ISR 内生成多个上下文切换请求。



1. 在 RTOS 滴答定时中断执行：正如本书后面将看到的，可以将应用程序代码添加到 RTOS 滴答中断中。 尝试在滴答中断内进行上下文切换的结果取决于正在使用的 FreeRTOS 移植。 充其量，它会导致对调度程序的不必要调用。



使用 pxHigherPriorityTaskWoken 参数是可选的。 如果不需要，则将 pxHigherPriorityTaskWoken 设置为 NULL。

### portYIELD_FROM_ISR() 和 portEND_SWITCHING_ISR() 宏

本节介绍用于从 ISR 请求上下文切换的宏。 如果您尚未完全理解本节，请不要担心，因为以下部分提供实际示例。



taskYIELD() 是一个宏，可以在任务中调用以请求上下文切换。 portYIELD_FROM_ISR() 和 portEND_SWITCHING_ISR() 都是 taskYIELD() 的中断安全版本。 portYIELD_FROM_ISR() 和 portEND_SWITCHING_ISR() 都以同样的方式使用，并且做同样的事情。 某些 FreeRTOS 移植仅提供两个宏之一。 较新的 FreeRTOS 移植提供这两种宏。本书中的示例使用 portYIELD_FROM_ISR()。

1

```verilog
portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );
```



清单 87. portEND_SWITCHING_ISR() 宏 

```verilog
portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
```



清单 88. portYIELD_FROM_ISR() 宏



从中断安全 API 函数传出的 xHigherPriorityTaskWoken 参数可以直接用作调用 portYIELD_FROM_ISR()中的参数。



如果 portYIELD_FROM_ISR()xHigherPriorityTaskWoken 参数是 pdFALSE（0），则不请求上下文切换，并且宏不起作用。 如果 portYIELD_FROM_ISR()xHigherPriorityTaskWoken 参数不是 pdFALSE，则请求上下文切换，并且处于 Running 状态的任务可能会更改。 中断将始终返回到处于运行状态的任务，即使在中断执行期间处于运行状态的任务发生了变化。



大多数 FreeRTOS 移植允许在 ISR 内的任何位置调用 portYIELD_FROM_ISR()。 一些 FreeRTOS 移植（主要用于较小架构的端口）仅允许在 ISR 的最后调用 portYIELD_FROM_ISR()。



## 延迟中断处理

保持 ISR 尽可能短通常被认为是最佳实践。 原因包括：



- 即使任务已被分配非常高的优先级，如果硬件不提供中断，则它们只会运行。
- ISR 可以中断（添加“抖动”）任务的开始时间和执行时间。

- 根据运行 FreeRTOS 的架构，在执行 ISR 时可能无法接受任何新中断，或至少是新中断的子集。
- 应用程序编写者需要考虑并防范任务和 ISR 同时访问变量、外设和内存缓冲区等资源的后果。

- 一些 FreeRTOS 移植允许中断嵌套，但中断嵌套会增加复杂性并降低可预测性。中断越短，嵌套的可能性就越小。
- 

中断服务程序必须记录中断的原因，并清除中断。 中断所需的任何其他处理通常可以在任务中执行，从而允许中断服务程序尽可能快地退出。这称为“延迟中断处理”，因为中断所需的处理从“延迟”开始。 任务的 ISR。



将中断处理推迟到任务还允许应用程序编写者优先处理相对于应用程序中其他任务的处理，并使用所有 FreeRTOS API 函数。



如果中断处理的任务的优先级高于任何其他任务的优先级，则处理将立即执行，就像处理已在 ISR 本身中执行一样。 这种情况如图 48 所示，其中任务 1 是一个正常的应用程序任务，任务 2 是中断处理被推迟到的任务。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636075883656-39b14edf-eb77-4a3d-b477-c442fc65c0e8.png)

图 48. 高优先级任务中的中断处理



在图 48 中，中断处理从时间 t2 开始，实际上在时间 t4 结束，但只有时间 t2 和 t3 之间的时间段用于 ISR。 如果未使用延迟中断处理，则时间 t2 和 t4 之间的整个时间段都将花费在 ISR 中。



关于何时最好执行 ISR 中的中断所需的所有处理以及何时最好将部分处理推迟到任务，没有绝对的规则。 在以下情况下将处理推迟到任务最有用：



- 中断所需的处理并非微不足道。 例如，如果中断只是存储模数转换的结果，那么几乎可以肯定这最好在 ISR 内部执行，但如果转换结果还必须通过软件过滤器，那么它可能是最好在任务中执行过滤器。
- 中断处理可以方便地执行 ISR 内部无法执行的操作，例如写入控制台或分配内存。

- 中断处理不是确定性的——这意味着事先不知道处理需要多长时间。



以下各节描述并演示了本章到目前为止介绍的概念，包括可用于实现延迟中断处理的 FreeRTOS 功能。



## 用于同步的Binary Semaphore 

中断安全版本的Binary Semaphore API可以被用来在每次特定的中断发生时解除对任务的封锁。有效地使任务与中断同步。这使得大部分的中断事件处理在同步任务中实现。只有非常快和短的部分直接留在ISR中。如上一节所述，二进制信号是被用来 “推迟”中断处理的到一个任务1。



使用直接到任务的通知将一个任务从中断中解禁，比使用Binary Semaphore更有效率。直接到任务的通知在第9章 "任务通知 "中才会涉及。



正如之前在图48中所展示的，如果中断处理是特别关键的时间，那么可以设置延迟处理任务的优先级，以确保该任务总是优先于系统中的其他任务。然后，ISR可以被实现，包括对 portYIELD_FROM_ISR() ，确保ISR直接返回到中断处理的任务中。这样做的效果是确保整个事件处理的在时间上连续执行（没有中断），这就像所有的事件都是在ISR本身。图49重复了图48中的情景，但文字被更新为描述如何使用semaphore来控制延迟处理任务的执行。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636079382522-076e62b6-ff99-42b3-9216-0ffa86e99374.png)

图49. 使用二进制信号来实现延迟的中断处理



延迟处理任务使用一个阻塞的 "take" 调用到 semaphore ，作为一种手段进入阻塞状态以等待事件的发生。当事件发生时，ISR在同一信号上使用 "give" 操作来解除对任务的封锁，这样所需的事件处理可以继续进行。



“接受信号”和“给予信号”是具有不同含义的概念根据它们的使用情况，有不同的含义。在这种中断同步的情况下，binary Semaphore在概念上可以被认为是一个长度为1的队列。该队列可以在任何时候都最多包含一个项，所以总是要么是空的，要么是满的（hence, binary）。通过调用 xSemaphoreTake() ，中断处理被推迟到的任务有效地试图用一个阻塞时间从队列中读取，如果队列是空的，则导致任务进入阻塞状态。当事件发生时，ISR使用 xSemaphoreGiveFromISR() 函数将一个令牌（semaphore）放入队列，使队列变满。这将导致任务退出阻塞状态并移除令牌，使队列再次变空。当任务完成其处理后，它再次尝试从队列中读出并发现队列是空的，于是重新进入阻塞状态以等待下一个事件。这个序列在图50中展示。



图50显示了中断“给出”信号，尽管它并没有首先“拿走”它，以及任务“拿”走了信号，但没有把它送回去。这就是为什么这种情况被描述为在概念上类似于向队列中写和从队列中读。它经常引起混淆因为它并不遵循与其他信号使用场景相同的规则，在其他场景中，一个任务使用semaphore的任务必须一直把它送回来--比如第7章中资源管理描述的场景。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636081992050-7c154d1a-1644-4f15-978d-aefaf283a81c.png)

图50. 使用binary semaphore使一个任务与中断同步



### xSemaphoreCreateBinary() API函数

FreeRTOS V9.0.0还包括 xSemaphoreCreateBinaryStatic() 函数，该函数在编译时分配内存。所需的内存，以便在编译时静态地创建一个二进制信号：对所有各种类型的FreeRTOS semaphore的句柄都存储在一个SemaphoreHandle_t类型的变量中。



在使用semaphore之前，必须先创建它。要创建一个binary semaphore，请使用 xSemaphoreCreateBinary() API函数1。



一些Semaphore API函数实际上是宏，而不是函数。为了简单起见，在本书中被称为函数。

```verilog
SemaphoreHandle_t xSemaphoreCreateBinary( void );
```

清单89. xSemaphoreCreateBinary() API函数原型



表 33. xSemaphoreCreateBinary() 的返回值

| 参数名称 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 返回值   | 如果返回的是NULL，那么这个信号就不能被创建，因为没有足够的堆内存可供FreeRTOS分配给semaphore数据结构。如果返回的值不是NULL，则表明该信号已被成功创建。返回的值应该作为创建的semaphore的句柄。 |

### xSemaphoreTake() API函数

“取用”semaphore 是指 "获得 "或 "接收 "该semaphore 。只有在semaphore是可用的情况下才可以采取。



所有不同类型的FreeRTOS信号，除了递归互斥，都可以使用 xSemaphoreTake() 函数。



xSemaphoreTake() 不能从一个中断服务例程中使用。

```verilog
BaseType_t xSemaphoreTake( SemaphoreHandle_t xSemaphore, TickType_t xTicksToWait );
```

清单90. xSemaphoreTake() 的API函数原型



表 34.xSemaphoreTake() 参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xSemaphore      | 被“带走”的semaphore。一个semaphore是由一个 SemaphoreHandle_t 类型的变量来引用的。它必须在使用前明确地创建。 |
| xTicksToWait    | 任务在阻塞状态下保持的最大时间状态下等待semaphore，如果semaphore还没有可用的话。如果xTicksToWait是0，那么xSemaphoreTake()将立即返回，如果 则将立即返回，因为semaphore是不可用的。区块时间是以tick周期为单位的，所以它的绝对时间代表的绝对时间取决于tick频率。宏指令pdMS_TO_TICKS()可以用来将一个以毫秒为单位的时间转换成以刻度为单位的时间。将xTicksToWait设置为portMAX_DELAY，如果INCLUDE_vTaskSuspend 在 FreeRTOSConfig.h 中被设置为 1，将导致任务无限期地等待（没有超时）。 |
| 返回值          | 有两个可能的返回值：1.pdPASSpdPASS仅在调用 xSemaphoreTake() 成功获得信号时返回。如果指定了一个块状时间（xTicksToWait不是0），那么就有可能调用的任务被放置到阻塞状态，以等待semaphore，如果它不是立即可用的，但是在阻塞时间结束前，semaphore变得可用。2.pdFALSE该信号灯不可用。如果指定了一个阻塞时间（xTicksToWait不是0），那么调用任务将被置入阻塞状态，以等待semaphore变成可用的状态，但是在这之前阻断时间已过期。 |

### xSemaphoreGiveFromISR() API函数

二进制和计数Binary and counting semaphores1可以使用 xSemaphoreGiveFromISR() 函数 “给出”。

本书的后面一节将介绍Counting semaphores。



xSemaphoreGiveFromISR() 是 xSemaphoreGive() 的中断安全版本，所以具有pxHigherPriorityTaskWoken参数，这在本章开始时已经描述过。

```verilog
BaseType_t xSemaphoreGiveFromISR( SemaphoreHandle_t xSemaphore, 
 BaseType_t *pxHigherPriorityTaskWoken );
```

清单91. xSemaphoreGiveFromISR() API函数原型



表 35. xSemaphoreGiveFromISR() 参数和返回值

| 参数名称/返回值           | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| xSemaphore                | 被 "给予 "的semaphore。一个semaphore是由一个类型为SemaphoreHandle_t的变量来引用，并且在使用前必须明确地被创建 使用。 |
| pxHigherPriorityTaskWoken | 有可能在一个单一的semaphore上会有一个或多个 任务阻塞在其上，等待semaphore成为 可用的。调用 xSemaphoreGiveFromISR() 可以使 可以使semaphore可用，从而导致等待semaphore的任务离开阻塞状态。如果调用 xSemaphoreGiveFromISR() 导致一个任务离开 阻塞状态，而解除阻塞的任务的优先级高于 比当前执行的任务（被中断的任务）的优先级高。那么，在内部，xSemaphoreGiveFromISR() 将*pxHigherPriorityTaskWoken 设置为pdTRUE。如果 xSemaphoreGiveFromISR() 把这个值设置为pdTRUE。那么通常情况下，在退出中断之前应该进行上下文切换。这将确保该中断直接返回到最高优先级的就绪状态任务。 |
| 返回值                    | 有两个可能的返回值：1. pdPASSpdPASS只有在调用 xSemaphoreGiveFromISR() 是成功的。2、pdFAIL如果一个semaphore已经可用，则不能被给予。并且 xSemaphoreGiveFromISR() 将返回pdFAIL。 |

### 示例16. 使用binary semaphore使一个任务与中断同步

这个例子使用一个binary semaphore来解除一个任务与中断服务程序之间的阻断——有效地使该任务与中断同步。



一个简单的周期性任务被用来每500毫秒产生一个软件中断。一个软件中断是为了方便而使用的，因为在一些目标环境中，连接到一个真正的中断的复杂性，所以使用软件中断是为了方便。清单92显示了周期性任务的实现。请注意，该任务在中断产生前后都会打印出一个字符串。这使得执行的顺序可以在例子执行时产生的输出中被观察到。

```verilog
/* 本例中使用的软件中断的编号。显示的代码来自Windows项目，其中数字0到2是由FreeRTOS Windows端口本身使用的，所以3是应用程序可用的第一个数字。*/
#define mainINTERRUPT_NUMBER 3
static void vPeriodicTask( void *pvParameters )
{
    const TickType_t xDelay500ms = pdMS_TO_TICKS(500UL);
 		/* 和大多数任务一样，这个任务是在一个无限循环中实现的。*/
 		for( ;; )
 		{
 				/* 阻止直到再次产生软件中断的时间。*/
 			 vTaskDelay( xDelay500ms );
 			/* 产生中断，在中断产生前后打印一条信息。这样执行的顺序就可以从输出中看出来。
 用于生成软件中断的语法取决于所使用的FreeRTOS的端口。下面使用的语法只能用于FreeRTOS Windows端口，在该端口中，这种中断只是模拟的。*/
 			 vPrintString( "Periodic task - About to generate an interrupt.\r\n" );
 			 vPortGenerateSimulatedInterrupt( mainINTERRUPT_NUMBER );
 			 vPrintString( "Periodic task - Interrupt generated.\r\n\r\n\r\n" );
 		} 
}
```

清单92. 实现例16中定期生成软件中断的任务 



清单93显示了中断处理被推迟的任务的实现——该任务通过使用一个binaty semaphore同步。同样，在任务的每次迭代中都会打印出一个字符串，所以任务和中断的执行顺序可以从例子执行时产生的输出中看出来。

```verilog
{
    /* 和大多数任务一样，这个任务是在一个无限循环中实现的。 */
    for (;;)
    {
        /* 使用semaphore来等待事件的发生。这个semaphore是在调度器启动之前创建的，所以在这个任务第一次运行之前的时候。这个任务会无限期地阻塞，这意味着这个函数调用只会在成功获得semaphore返回——所以不需要检查xSemaphoreTake()返回的值。*/
        xSemaphoreTake(xBinarySemaphore, portMAX_DELAY);
      /* 要到达这里，这个事件必须已经发生。处理该事件（在本例中，只是打印出一条信息）。 */
        vPrintString("Handler task - Processing event.\r\n");
    }
}
```

清单93. 中断处理所涉及的任务的实现 （与中断同步的任务）。



清单94显示了这个ISR，除了 "给 "信号以解除对中断处理的任务的封锁外，没有什么作用。



注意xHigherPriorityTaskWoken变量是如何被使用的。在调用xSemaphoreGiveFromISR() 之前，它被设置为pdFALSE，然后在调用portYIELD_FROM_ISR() 时作为参数使用。如果xHigherPriorityTaskWoken等于pdTRUE，在 portYIELD_FROM_ISR() 宏中会要求进行上下文切换。



与大多数FreeRTOS运行的架构不同，FreeRTOS的Windows端口需要一个ISR来返回一个值。与Windows端口一起提供的 portYIELD_FROM_ISR() 宏的实现包括返回语句，所以清单94没有明确显示返回值。

```verilog
static uint32_t ulExampleInterruptHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken;
    /* xHigherPriorityTaskWoken参数必须被初始化为pdFALSE，因为如果需要进行上下文切换，它将在中断安全API函数中被设置为pdTRUE。 */
    xHigherPriorityTaskWoken = pdFALSE;
    /* “给予”semaphore以解除对任务的封锁任务，将xHigherPriorityTaskWoken的地址作为中断安全API函数的参数pxHigherPriorityTaskWoken传入。 */
    xSemaphoreGiveFromISR(xBinarySemaphore, &xHigherPriorityTaskWoken);
  /* 将xHigherPriorityTaskWoken的值传给portYIELD_FROM_ISR()。如果xHigherPriorityTaskWoken在xSemaphoreGiveFromISR()中被设置为pdTRUE，那么调用portYIELD_FROM_ISR()将请求进行上下文切换。如果xHigherPriorityTaskWoken仍然是pdFALSE，那么调用   portYIELD_FROM_ISR()将没有任何影响。与大多数 FreeRTOS 端口不同， Windows 端口要求 ISR 返回一个值——返回语句在 Windows 版本的 portYIELD_FROM_ISR() 中。*/
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单94. 示例16中使用的软件中断的ISR



main()函数创建了binary semaphore，创建了任务，安装了中断处理程序。处理并启动调度程序。该函数的实现见清单95。



为安装中断处理程序而调用的函数的语法是针对FreeRTOS Windows端口的，对于其他FreeRTOS端口可能有所不同。请参考FreeRTOS.org网站上的特定端口文档页面，以及FreeRTOS下载中提供的例子，以找到你正在使用的端口所需的语法。

```verilog
int main(void)
{
    /* 在使用semaphore之前，必须明确地创建它。在这个例子中，我们创建了一个binary semaphore。 */
    xBinarySemaphore = xSemaphoreCreateBinary();
    /* 检查semaphore是否成功创建。 */
    if (xBinarySemaphore != NULL)
    {
        /* 创建 "处理程序 "任务，这是一个被推迟处理的中断任务,一个将与中断同步的任务。处理程序任务是以高优先级创建的，以确保它在中断退出后立即运行。在这种情况下，我们选择了3的优先级。 */
        xTaskCreate(vHandlerTask, "Handler", 1000, NULL, 3, NULL);
        /* 创建将定期产生一个软件中断的任务。这个任务的优先级低于处理任务，以确保每次处理任务退出阻塞状态时，它将被抢占。 */
        xTaskCreate(vPeriodicTask, "Periodic", 1000, NULL, 1, NULL);
        /* 安装软件中断的处理程序。这样做所需的语法取决于正在使用的FreeRTOS端口。这里显示的语法只能用于FreeRTOS的windows端口，在那里这种中断只是模拟的。 */
        vPortSetInterruptHandler(mainINTERRUPT_NUMBER, ulExampleInterruptHandler);
        /* 启动调度器，使创建的任务开始执行。 */
        vTaskStartScheduler();
    }
    /* 正常情况下，绝对不能达到下面这一行。 */
    for (;;);
}
```

清单95. 示例16中 main() 的实现



例16产生的输出如图51所示。正如预期的那样，vHandlerTask() 在中断产生后立即进入了运行状态，所以任务的输出与周期性任务产生的输出相分离。图52中提供了进一步的解释。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636093177939-15bcfd4b-fdd7-45ea-8215-02b2af35bafc.png)

图51. 执行示例16时产生的输出

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636093274205-f5851fb4-dffb-41f7-a29c-a2ac0c5f7907.png)

图52. 执行例16时的执行顺序

### 完善示例16中使用的任务的实现

示例16使用一个binary semaphore来同步一个任务和一个中断。其执行顺序如下：



1. 中断发生了。



2. ISR执行并“给出”semaphore 以解除对任务的封锁。



3. 任务在ISR之后立即执行，并“拿走”了semaphore。



4. 任务处理了该事件，然后试图再次“拿走”semaphore——进入阻塞状态，因为semaphore还不可用（另一个中断还没有发生）。



只有当中断发生的频率相对较低时，示例16中的任务结构才是充分的。为了理解这个原因，我们可以考虑一下，如果在任务完成对第一个中断的处理之前，又发生了第二个、第三个中断，会发生什么：



- 当第二个ISR执行时，semaphore将是空的，所以ISR将给出semaphore，且任务在完成对第一个事件的处理后将立即处理第二个事件。这种情况在图53中显示。



- 当第三个ISR执行时，semaphore已经可用，防止ISR再次给出semaphore，所以任务不会知道第三个事件已经发生。这种情况在图54中显示。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636094583388-c25d8f39-1a95-4fb2-a2ef-5a9fba60794a.png)

图53. 在任务完成处理第一个事件之前发生一个中断时的情景



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636094676453-cb3a2122-221e-4fbc-9fb5-bf133a9d19f8.png)

图54 在任务完成处理第一个事件之前发生两个中断时的情景



示例16中使用的延迟中断处理任务，如清单93所示，其结构是在每次调用 xSemaphoreTake() 时只处理一个事件。这对示例16来说是足够的，因为产生事件的中断是由软件触发的，而且发生的时间是可预测的。在实际应用中，中断是由硬件产生的，并且发生在不可预测的时间。因此，为了尽量减少错过中断的机会，延迟中断处理任务的结构必须使它在每次调用 xSemaphoreTake() 之间处理所有已经存在的事件。1

清单96证明了这一点，它显示了一个UART的延迟中断处理任务的结构。在清单96中，假设UART在每次收到字符时都会产生一个接收中断，并且UART将收到的字符放入一个硬件的 FIFO（一个硬件缓冲区）。

另外，也可以使用计数信号或直接到任务的通知来计数事件。计数信号将在下一节中描述。直接到在第9章,任务的通知中描述。直接到任务的通知是首选方法，因为它们在运行时间和RAM的使用上都是最有效的。

示例16中使用的延迟中断处理任务还有一个弱点；它在调用 xSemaphoreTake() 时没有使用超时。相反，该任务将portMAX_DELAY作为 xSemaphoreTake() 的xTicksToWait参数，这导致该任务无限期地等待（没有超时），等待 semaphore 的出现。无限期超时经常被用在示例代码中，因为它们的使用简化了示例的结构，从而使示例更容易理解。然而，在实际应用中，无限期超时通常是不好的做法，因为它们使人们很难从错误中恢复。作一个例子，考虑这样的情景：一个任务正在等待一个中断来给出 semaphore ，但是硬件中的错误状态阻止了中断的产生：



- 如果任务在没有超时的情况下等待，它将不知道错误状态，并将永远等待。
- 如果任务在等待时有超时，那么 xSemaphoreTake() 将在超时结束时返回pdFAIL，然后任务可以在下次执行时检测并清除错误。这种情况在清单96中也有演示。

```verilog
static void vUARTReceiveHandlerTask(void *pvParameters)
{
    /* xMaxExpectedBlockTime持有两个中断之间的最大预期时间。 */
    const TickType_t xMaxExpectedBlockTime = pdMS_TO_TICKS(500);
    /* 和大多数任务一样，这个任务是在一个无限循环中实现的。 */
    for (;;)
    {
        /* 该 semaphore 是由UART的接收(Rx)中断"给予"的。等待下一个中断的时间最多为xMaxExpectedBlockTime ticks。 */
        if (xSemaphoreTake(xBinarySemaphore, xMaxExpectedBlockTime) == pdPASS)
        {
            /* 获得了semaphore。在再次调用xSemaphoreTake()之前，处理所有待处理的Rx事件。每个Rx事件都会在UART的接收FIFO中放置一个字符，UART_RxCount()被认为是返回FIFO中的字符数。*/
            while (UART_RxCount() > 0)
            {
                /* UART_ProcessNextRxEvent()被假定为处理一个Rx字符，使FIFO中的字符数减少1。 */
                UART_ProcessNextRxEvent();
            }
            /* 没有更多的Rx事件等待处理（FIFO中没有更多的字符），所以回环并调用xSemaphoreTake()以等待下一个中断。从代码的这一点到调用xSemaphoreTake()之间发生的任何中断都会被锁在semaphore中，所以不会丢失。 */
        }
        else
        {
            /* 没有在预期的时间内收到一个事件。检查并在必要时清除UART中可能阻止UART产生更多中断的任何错误条件。 */
            UART_ClearErrors();
        }
    }
}
```

清单96. 推荐的延迟中断处理任务的结构，以UART接收处理程序为例

------

## Counting  Semaphores



正如binary semaphores 可以被认为是长度为1的队列一样，counting semaphores 可以被认为是长度为1以上的队列。任务对存储在队列中的数据不感兴趣，只是对队列中的项目数量感兴趣。 configUSE_COUNTING_SEMAPHORES必须在FreeRTOSConfig.h中设置为1，才能使用counting semaphores 。



每当一个 counting semaphore 被 "给予 "时，其队列中的另一个空间就被使用。队列中的项目数量是 semaphore 的“计数”值。



1. 计数事件1

在这种情况下，事件处理程序会在每次事件发生时 “给”一个semaphore——导致信号的计数值在每次“给出 ”时被递增。 一个任务在每次处理一个事件时都会“拿走”一个semaphore——导致semaphore 的数值被递减。

在每次 "取 "的时候都会递减。计数值是已经发生的事件数与已经发生的事件数之间的差值。这个机制是图55中所示。



用于统计事件的计数信号器在创建时，其初始计数值为0。

使用直接到任务的通知来计数事件比使用counting semaphores更有效。直接到任务的通知在第9章才会涉及。

2. 资源管理

在这种情况下，计数值表示可用资源的数量。为了获得资源的控制权，一个任务必须首先获得一个 semaphore ——减少 semaphore 的计数值。当计数值达到0时，就没有可用的资源了。当一个任务完成了对资源的控制，它就会“给”回 semaphore ——增加 semaphore 的计数值。



用于管理资源的 Counting semaphores 在创建时，其初始计数值等于可用资源的数量。第7章介绍了使用semaphores 来管理资源。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636097763206-7d7472d6-fae3-4560-a851-4917b7a68aca.png)

图55. 使用一个counting semaphore来 "计数 "事件

### xSemaphoreCreateCounting() API函数

FreeRTOS V9.0.0还包括 xSemaphoreCreateCountingStatic() 函数，该函数在编译时分配静态地创建一个counting semaphore的内存：所有不同类型的FreeRTOS信号的句柄都存储在一个SemaphoreHandle_t类型的变量中。



在使用semaphore之前，必须先创建它。要创建一个counting semaphore，请使用 xSemaphoreCreateCounting() API函数。

```verilog
SemaphoreHandle_t xSemaphoreCreateCounting( UBaseType_t uxMaxCount,
 UBaseType_t uxInitialCount );
```

清单97. xSemaphoreCreateCounting() API函数原型 



表 36. xSemaphoreCreateCounting()参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| uxMaxCount      | semaphore的最大值。继续用队列来比喻，uxMaxCount值实际上就是队列的长度。当semaphore被用来计算或锁住事件时，uxMaxCount是可以锁住的最大事件数。当semaphore被用来管理对资源集合的访问时，uxMaxCount应该被设置为可用资源的总数。 |
| uxInitialCount  | semaphore被创建后的初始计数值。当semaphore被用来计算或锁定事件时，uxInitialCount应该被设置为0——因为当semaphore被创建时，可能还没有事件发生。当semaphore被用来管理对一组资源的访问时，uxInitialCount应该被设置为等于uxMaxCount，因为当semaphore被创建时，所有的资源都是可用的。 |
| 返回值          | 如果返回NULL，说明不能创建semaphore，因为没有足够的堆内存可供FreeRTOS分配semaphore的数据结构。第2章提供了更多关于堆内存管理的信息。返回的非NULL值表示已经成功创建了semaphore。返回的值应该作为创建的semaphore的句柄来存储。 |

### 示例17. 使用计数信号来使一个任务与中断同步

示例17改进了示例16的实现，用binary semaphore代替了counting semaphore。main()被修改为包含一个对 xSemaphoreCreateCounting()，以取代对 xSemaphoreCreateBinary() 的调用。新的API 调用显示在清单98中。

```verilog
/* 在使用semaphore之前，必须明确地创建它。在这个例子中，一个counting semaphore被创建。创建的semaphore的最大计数值为10，初始计数值为0。 */
xCountingSemaphore = xSemaphoreCreateCounting(10, 0);
```

清单98. 示例17中用于创建counting semaphore的 xSemaphoreCreateCounting() 的调用



为了模拟高频率发生的多个事件，中断服务例程被改变为在每个中断中“给予”semaphore 不止一次。每个事件都被锁在 semaphore 的计数值中。修改后的中断服务例程显示在清单99中。

```verilog
static uint32_t ulExampleInterruptHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken;
    /* xHigherPriorityTaskWoken参数必须被初始化为pdFALSE，因为如果需要进行上下文切换，它将在中断安全API函数中被设置为pdTRUE。 */
    xHigherPriorityTaskWoken = pdFALSE;
    /* 多次 “给”semaphore 。第一次将解除对推迟的 中断处理任务，接下来的 "给予 "是为了证明  semaphore锁住事件，以允许被推迟的中断处理任务依次处理它们，而不会丢失事件。这模拟了处理器收到的多个中断，尽管在这种情况下，事件是在一个中断发生中模拟的。 */
    xSemaphoreGiveFromISR(xCountingSemaphore, &xHigherPriorityTaskWoken);
    xSemaphoreGiveFromISR(xCountingSemaphore, &xHigherPriorityTaskWoken);
    xSemaphoreGiveFromISR(xCountingSemaphore, &xHigherPriorityTaskWoken);
    /* 将xHigherPriorityTaskWoken的值传给portYIELD_FROM_ISR()。如果
 xHigherPriorityTaskWoken在xSemaphoreGiveFromISR()中被设置为pdTRUE，那么调用portYIELD_FROM_ISR()将请求进行上下文切换。如果xHigherPriorityTaskWoken仍然是pdFALSE，那么调用portYIELD_FROM_ISR()将没有任何影响。与大多数 FreeRTOS 端口不同， Windows 端口要求 ISR 返回一个值——返回语句在 Windows 版本的 portYIELD_FROM_ISR() 中。 */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单99. 示例17所使用的中断服务例程的实现



所有其他的函数与示例16中使用的函数相比都没有修改。



示例17执行时产生的输出如图56所示。可以看出，在每次产生中断时，被推迟处理的任务会处理所有三个[模拟]事件。这些事件被锁在信号的计数值中，允许任务依次处理它们。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636101378827-75ccc71d-0a24-4dd0-b078-f9e41f7bdabf.png)

图56. 执行示例17时产生的输出



## 推迟工作到RTOS守护进程任务

到目前为止，所介绍的延迟中断处理的例子都要求应用程序编写者为每个使用延迟处理技术的中断创建一个任务。也可以使用 xTimerPendFunctionCallFromISR() API函数将中断处理推迟到RTOS守护任务中——无需为每个中断创建一个单独的任务。将中断处理推迟到守护任务被称为“集中的推迟中断处理”。

在第五章中指出，守护任务最初被称为定时器服务任务，因为它最初只用来执行软件定时器回调函数。因此，xTimerPendFunctionCall() 是在 timers.c 中实现的，根据将函数的名称与实现该函数的文件名放在一起的惯例，该函数的名称前缀为"Timer"。

第5章描述了与软件定时器有关的FreeRTOS API函数如何在定时器命令队列中向守护任务发送命令。xTimerPendFunctionCall() 和 xTimerPendFunctionCallFromISR() API函数使用相同的定时器命令队列，向守护任务发送 "执行函数 "命令。发送给守护任务的函数随后在守护任务的上下文中被执行。



集中式延时中断处理的优点包括：

- 降低资源使用量

它消除了为每个延迟中断创建一个单独任务的需要。

- 简化的用户模式

​       递延中断处理函数是一个标准的C函数。



集中式延时中断处理的缺点包括：

- 灵活性较低

不可能单独设置每个递延中断处理任务的优先级。每个递延中断处理功能以守护任务的优先级执行。如第五章所述，守护任务的优先级是由 configTIMER_TASK_PRIORITY 在 FreeRTOSConfig.h 中的编译时间配置常数来设置。

- 减少决定论

xTimerPendFunctionCallFromISR() 将一个命令发送到定时器命令队列的后面。

在xTimerPendFunctionCallFromISR() 发送给队列的"execute function"命令之前，已经在定时器命令队列中的命令将被守护任务处理。



不同的中断有不同的时间限制，所以在同一个应用程序中使用两种推迟中断处理的方法是很常见的。



### xTimerPendFunctionCallFromISR() API函数

xTimerPendFunctionCallFromISR() 是 xTimerPendFunctionCall() 的中断安全版本。

这两个API函数都允许由应用程序编写者提供的函数由RTOS守护任务执行，因此也是在RTOS守护任务的上下文中执行。要执行的函数和函数的输入参数值都被发送到定时器命令队列中的守护任务。因此，该函数实际执行的时间取决于守护任务相对于应用程序中其他任务的优先级。

```verilog
BaseType_t xTimerPendFunctionCallFromISR( PendedFunction_t xFunctionToPend,
 void *pvParameter1,
 uint32_t ulParameter2,
 BaseType_t *pxHigherPriorityTaskWoken );
```

清单100. xTimerPendFunctionCallFromISR() API函数原型

```verilog
void vPendableFunction( void *pvParameter1, uint32_t ulParameter2 );
```

清单101. xTimerPendFunctionCallFromISR() 的xFunctionToPend参数中传递的函数必须符合的原型。



表 37. xTimerPendFunctionCallFromISR() 参数和返回值

| 参数名称/返回值           | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| pvParameter1              | 将被传递到由守护任务执行的函数中的值，作为该函数的pvParameter1参数。该参数有一个void *类型，允许它用来传递任何数据类型。例如，整数类型可以直接转换为void *，或者，void *可以用来指向一个结构。 |
| ulParameter2              | 将被传递到由守护任务执行的函数中的值，作为该函数的ulParameter2参数。 |
| pxHigherPriorityTaskWoken | xTimerPendFunctionCallFromISR() 写到定时器命令队列。如果RTOS守护任务处于阻塞状态以等待定时器命令队列上的数据，那么写到定时器命令队列将导致守护任务离开阻塞状态。如果守护任务的优先级高于当前执行的任务（被中断的任务）的优先级，那么在内部，xTimerPendFunctionCallFromISR() 将把*pxHigherPriorityTaskWoken设为pdTRUE。如果 xTimerPendFunctionCallFromISR() 将此值设置为pdTRUE，那么在退出中断之前必须进行上下文切换。这将确保中断直接返回到守护任务，因为守护任务将是最高优先级的就绪状态任务。 |
| 返回值                    | 有两个可能的返回值：1. pdPASS如果"execute function"命令被写入定时器命令队列，将返回pdPASS。2. pdFAIL如果"execute function"命令不能被写入定时器命令队列，因为定时器命令队列已经满了，则将返回pdFAIL。第5章描述了如何设置定时器命令队列的长度。 |

### 示例18. 集中的延迟中断处理

示例18提供了与示例16类似的功能，但没有使用semaphore，也没有专门创建一个任务来执行中断所需要的处理。相反，处理是由RTOS守护任务执行的。



示例18所使用的中断服务例程如清单102所示。它调用 xTimerPendFunctionCallFromISR()，将一个指向vDeferredHandlingFunction() 函数的指针传递给守护任务。延迟的中断处理由vDeferredHandlingFunction() 函数执行。



中断服务程序每次执行时都会增加一个名为 ulParameterValue 的变量。ulParameterValue 在调用xTimerPendFunctionCallFromISR() 时被用作 ulParameter2 的值，因此在守护任务执行vDeferredHandlingFunction() 时也将被用作 ulParameter2 的值。该函数的另一个参数pvParameter1，在这个例子中没有使用。

```verilog
static uint32_t ulExampleInterruptHandler(void)
{
    static uint32_t ulParameterValue = 0;
    BaseType_t xHigherPriorityTaskWoken;
    /* xHigherPriorityTaskWoken参数必须被初始化为pdFALSE，因为如果需要进行上下文切换，它将在中断安全API函数中被设置为pdTRUE。 */
    xHigherPriorityTaskWoken = pdFALSE;
    /* 向守护任务发送一个指向中断的延迟处理函数的指针。递延处理函数的pvParameter1参数不使用，所以直接设置为NULL。递延处理函数的ulParameter2参数用于传递一个数字，这个数字在每次执行这个中断处理程序时都会递增1。 */
    xTimerPendFunctionCallFromISR(vDeferredHandlingFunction, /* 要执行的功能。 */
                                  NULL,                      /* 未使用。 */
                                  ulParameterValue,          /* 递增值。 */
                                  &xHigherPriorityTaskWoken);
    ulParameterValue++;
    /* 将xHigherPriorityTaskWoken的值传给portYIELD_FROM_ISR()。如果 xHigherPriorityTaskWoken 在 xTimerPendFunctionCallFromISR() 中被设置为 pdTRUE， 那么调用 portYELD_FROM_ISR() 将请求进行上下文切换。如果 xHigherPriorityTaskWoken 仍然是 pdFALSE， 那么调用 portYIELD_FROM_ISR() 将没有任何影响。与大多数 FreeRTOS 端口不同，Windows 端口要求 ISR 返回一个值——返回语句在 Windows 版本的 portYIELD_FROM_ISR() 中。 */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单102. 示例18中使用的软件中断处理程序



vDeferredHandlingFunction() 的实现在清单103中显示。它打印出一个固定的字符串，以及其ulParameter2的参数值。



vDeferredHandlingFunction() 必须具有清单101中的原型，尽管在这个例子中，只有一个参数被实际使用。

```verilog
static void vDeferredHandlingFunction(void *pvParameter1, uint32_t ulParameter2)
{
    /* 处理事件————在本例中只是打印出一条信息和ulParameter2的值。 pvParameter1在本例中没有使用。 */
    vPrintStringAndNumber("Handler function - Processing event ", ulParameter2);
}
```

清单103. 执行示例18中中断所需的处理的函数



示例18使用的main() 函数在清单104中显示。它比示例16使用的 main() 函数更简单，因为它没有创建一个semaphore 或一个任务来执行延迟中断处理。



vPeriodicTask() 是周期性产生软件中断的任务。它被创建时的优先级低于守护任务的优先级，以确保一旦守护任务离开阻塞状态，它就会被守护任务抢占。

```verilog
int main(void)
{
    /* 产生软件中断的任务是以低于守护任务的优先级创建的。守护任务的优先级由FreeRTOSConfig.h中的configTIMER_TASK_PRIORITY编译时配置常数设置。 */
    const UBaseType_t ulPeriodicTaskPriority = configTIMER_TASK_PRIORITY - 1;
    /* 创建将周期性产生软件中断的任务。 */
    xTaskCreate(vPeriodicTask, "Periodic", 1000, NULL, ulPeriodicTaskPriority, NULL);
    /* 安装软件中断的处理程序。这样做所需的语法取决于正在使用的FreeRTOS端口。这里显示的语法只能用于FreeRTOS的windows端口，在那里这种中断只是模拟的。 */
    vPortSetInterruptHandler(mainINTERRUPT_NUMBER, ulExampleInterruptHandler);
    /* 启动调度器，使创建的任务开始执行。 */
    vTaskStartScheduler();

    /* 正常情况下，绝对不能达到下面这行。 */
    for (;;)
        ;
}
```

清单104. 示例18中 main() 的实现



示例18产生的输出如图57所示。守护任务的优先级高于产生软件中断的任务的优先级，所以vDeferredHandlingFunction() 在中断产生后立即被守护任务执行。这导致 vDeferredHandlingFunction() 输出的消息出现在周期性任务输出的两个消息之间，就像用 semaphore 来解锁专门的延迟中断处理任务时那样。图58中提供了进一步的解释。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636105289736-5c5c060b-774c-4943-a6af-84c5675d2b25.png)

图57. 执行示例18时产生的输出

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636105350694-0dc0ebe9-b795-4087-a35e-20942bf90516.png)

图58 执行示例18时的执行顺序



## 在中断服务程序中使用队列

二进制和计数信号被用来交流事件。队列被用来交流事件，并传输数据。



xQueueSendToFrontFromISR() 是 xQueueSendToFront() 的版本，可以在中断服务例程中安全使用；xQueueSendToBackFromISR() 是 xQueueSendToBack() 的版本，可以在中断服务例程中安全使用；而xQueueReceiveFromISR() 是 xQueueReceive() 的版本，可以在中断服务例程中安全使用。



### xQueueSendToFrontFromISR() 和xQueueSendToBackFromISR() API 函数

```verilog
BaseType_t xQueueSendToFrontFromISR(QueueHandle_t xQueue,
                                    void *pvItemToQueue
                                        BaseType_t *pxHigherPriorityTaskWoken
                                   );
```

清单105. xQueueSendToFrontFromISR() API函数原型

```verilog
BaseType_t xQueueSendToBackFromISR(QueueHandle_t xQueue,
                                   void *pvItemToQueue
                                       BaseType_t *pxHigherPriorityTaskWoken
                                  );
```

清单106. xQueueSendToBackFromISR() API函数原型



xQueueSendFromISR() 和 xQueueSendToBackFromISR() 在功能上等同。



表 38. xQueueSendToFrontFromISR() 和 xQueueSendToBackFromISR() 参数和返回值

| 参数名称/返回值           | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| xQueue                    | 发送（写入）数据的队列的句柄。该队列句柄将从用于创建队列的xQueueCreate()调用中返回。 |
| pvItemToQueue             | 一个指向将被复制到队列中的数据的指针。队列所能容纳的每个项目的大小是在创建队列时设置的，所以这个字节将从 pvItemToQueue 复制到队列存储区。 |
| pxHigherPriorityTaskWoken | 一个队列有可能会有一个或多个任务被阻塞在上面，等待数据的出现。调用 xQueueSendToFrontFromISR() 或 xQueueSendToBackFromISR() 可以使数据可用，从而导致这样一个任务离开阻塞状态。如果调用API函数导致一个任务离开阻塞状态，并且解除阻塞的任务的优先级高于当前执行的任务（被中断的任务），那么在内部，API函数将把*pxHigherPriorityTaskWoken 设置为pdTRUE。如果 xQueueSendToFrontFromISR() 或xQueueSendToBackFromISR() 将此值设置为pdTRUE，那么在中断退出之前应该进行上下文切换。这将确保中断直接返回到最高优先级的就绪状态任务。 |
| 返回值                    | 有两个可能的返回值:1. pdPASSpdPASS仅在数据被成功发送到队列时返回。2. errQUEUE_FULLerrQUEUE_FULL如果数据不能被发送到队列，则返回，因为队列已经满了。 |

### 从ISR中使用队列时的考虑因素

队列提供了一种简单方便的方式将数据从中断传递给任务，但如果数据到达的频率很高，使用队列的效率就不高。



FreeRTOS下载中的许多演示程序包括一个简单的UART驱动程序，它使用一个队列将字符从UART的接收ISR中传递出来。在这些演示程序中，使用队列有两个原因：演示队列在ISR中的使用，以及故意加载系统以测试FreeRTOS端口。以这种方式使用队列的ISR绝对不是为了代表一种有效的设计，除非数据到达的速度很慢，否则建议生产代码不要复制这种技术。更有效的技术，那些适合于生产代码，包括：



- 使用直接内存访问（DMA）硬件来接收和缓冲字符。这种方法实际上没有软件开销。然后可以使用直接到任务的通知1来解除对任务的封锁，只有在检测到传输中断后才会处理缓冲区。
- 将收到的每个字符复制到线程安全的RAM缓冲区2。同样，在收到一个完整的消息后，或在检测到传输中断后，可以使用一个直接到任务的通知来解除对处理缓冲区的任务的封锁。

- 在ISR中直接处理收到的字符，然后使用队列将数据处理的结果（而不是原始数据）发送给一个任务。这在之前由图34演示过。



1 直接到任务的通知提供了从ISR中解除任务阻塞的最有效方法。直接到任务的通知将在第9章 "任务通知 "中介绍。

2 作为FreeRTOS+TCP(http://www.FreeRTOS.org/tcp)的一部分提供的 "流缓冲 "可用于此目的。

### 示例19. 在一个队列中从一个中断中发送和接收信息

这个例子演示了 xQueueSendToBackFromISR() 和 xQueueReceiveFromISR() 在同一个中断中使用。和以前一样，为了方便，中断是由软件产生的。



创建一个周期性任务，每200毫秒向一个队列发送五个数字。只有在所有五个数字都被发送后，它才会产生一个软件中断。该任务的实现在清单107中显示。

```verilog
static void vIntegerGenerator(void *pvParameters)
{
    TickType_t xLastExecutionTime;
    uint32_t ulValueToSend = 0;
    int i;
    /* 初始化调用vTaskDelayUntil()所使用的变量。 */
    xLastExecutionTime = xTaskGetTickCount();
    for (;;)
    {
        /* 这是一个定期任务。阻止直到它再次运行的时间。该任务将每200ms执行一次。 */
        vTaskDelayUntil(&xLastExecutionTime, pdMS_TO_TICKS(200));
        /* 向队列发送五个数字，每个数字比前一个数字高一个。这些数字由中断服务例程从队列中读取。
中断服务例程总是清空队列，所以这个任务保证能够写入所有五个值，而不需要指定一个块的时间。 */
        for (i = 0; i < 5; i++)
        {
            xQueueSendToBack(xIntegerQueue, &ulValueToSend, 0);
            ulValueToSend++;
        }
        /* 产生中断，以便中断服务程序可以从队列中读取数值。用于产生软件中断的语法取决于正在使用的FreeRTOS端口。下面使用的语法只能用于FreeRTOS的Windows端口，在该端口中，这种中断只是模拟的。*/
        vPrintString("Generator task - About to generate an interrupt.\r\n");
        vPortGenerateSimulatedInterrupt(mainINTERRUPT_NUMBER);
        vPrintString("Generator task - Interrupt generated.\r\n\r\n\r\n");
    }
}
```

清单 107. 示例19中写入队列的任务的实现



中断服务例程反复调用 xQueueReceiveFromISR() ，直到所有由周期性任务写入队列的值都被读出，队列留空。每个收到的值的最后两位被用作一个字符串数组的索引。然后通过调用 xQueueSendFromISR() ，将相应索引位置的字符串指针发送到不同的队列。中断服务例程的实现见清单108。

```verilog
static uint32_t ulExampleInterruptHandler(void)
{
    BaseType_t xHigherPriorityTaskWoken;
    uint32_t ulReceivedNumber;
    /* 字符串被声明为静态常量，以确保它们不被分配到中断服务程序的堆栈中，因此即使在中断服务程序不执行时也存在。 */
    static const char *pcStrings[] =
        {
            "String 0\r\n",
            "String 1\r\n",
            "String 2\r\n",
            "String 3\r\n"};
    /* 一如既往，xHigherPriorityTaskWoken被初始化为pdFALSE，以便能够检测到它在中断安全API函数中被设置为pdTRUE。请注意，由于中断安全API函数只能将xHigherPriorityTaskWoken设置为pdTRUE，所以在调用xQueueReceiveFromISR()和调用xQueueSendToBackFromISR()时使用同一个xHigherPriorityTaskWoken变量是安全的。 */
    xHigherPriorityTaskWoken = pdFALSE;
    /* 从队列中读取，直到队列为空。 */
    while (xQueueReceiveFromISR(xIntegerQueue,
                                &ulReceivedNumber,
                                &xHigherPriorityTaskWoken) != errQUEUE_EMPTY)
    {
        /* 将接收到的值截断到最后两位（值0到3，包括在内），然后用截断的值作为pcStrings[]数组的索引，选择一个字符串（char *）发送到另一个队列上。 */
        ulReceivedNumber &= 0x03;
        xQueueSendToBackFromISR(xStringQueue,
                                &pcStrings[ulReceivedNumber],
                                &xHigherPriorityTaskWoken);
    }
    /* 如果从xIntegerQueue接收导致任务离开阻塞状态，并且如果离开阻塞状态的任务的优先级高于运行状态的任务的优先级，那么xHigherPriorityTaskWoken将在xQueueReceiveFromISR()中被设置为pdTRUE。
 如果向xStringQueue发送导致任务离开阻塞状态，并且如果离开阻塞状态的任务的优先级高于运行状态的任务的优先级，那么xHigherPriorityTaskWoken将在xQueueSendToBackFromISR()中被设置为pdTRUE。xHigherPriorityTaskWoken被用作portYIELD_FROM_ISR()的参数。如果xHigherPriorityTaskWoken等于pdTRUE，那么调用portYIELD_FROM_ISR()将请求进行上下文切换。如果xHigherPriorityTaskWoken仍然是pdFALSE，那么调用portYIELD_FROM_ISR()将没有任何作用。
Windows端口使用的portYIELD_FROM_ISR()的实现包括一个返回语句，这就是为什么这个函数没有明确地返回一个值。 */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单108. 例19所使用的中断服务例程的实现



从中断服务例程中接收字符指针的任务在队列中阻塞，直到有消息到达，在接收到每个字符串时打印出来。它的实现在清单109中显示。

```verilog
static void vStringPrinter( void *pvParameters )
{
char *pcString;
 for( ;; )
 {
 /* Block on the queue to wait for data to arrive. */
 xQueueReceive( xStringQueue, &pcString, portMAX_DELAY );
 /* Print out the string received. */
 vPrintString( pcString );
 } }
```

清单109. 打印出从示例19中的中断服务例程收到的字符串的任务



像往常一样，main()在启动调度程序之前创建所需的队列和任务。它的实现在清单110中显示。

```verilog
int main( void )
{
 /* 在使用一个队列之前，必须首先创建它。创建本例所使用的两个队列。一个队列可以容纳uint32_t类型的变量，另一个队列可以容纳char*类型的变量。两个队列最多可以容纳10个项目。真正的应用程序应该检查返回值以确保队列被成功创建。 */
 xIntegerQueue = xQueueCreate( 10, sizeof( uint32_t ) );
 xStringQueue = xQueueCreate( 10, sizeof( char * ) );
 /* 创建一个任务，使用队列将整数传递给中断服务例程。该任务的优先级为1。 */
 xTaskCreate( vIntegerGenerator, "IntGen", 1000, NULL, 1, NULL );
 /* 创建一个任务，打印出从中断服务例程发送给它的字符串。这个任务是以较高的优先级2创建的。 */
 xTaskCreate( vStringPrinter, "String", 1000, NULL, 2, NULL );
 /* 安装软件中断的处理程序。这样做所需的语法取决于正在使用的FreeRTOS端口。这里显示的语法只能用于FreeRTOS的Windows端口，在那里这种中断只是模拟的。 */
 vPortSetInterruptHandler( mainINTERRUPT_NUMBER, ulExampleInterruptHandler );
 /* 启动调度程序，使创建的任务开始执行。 */
 vTaskStartScheduler();
 
 /* 如果一切顺利，那么main()将永远不会到达这里，因为调度器现在正在运行任务。如果main()确实到达这里，那么很可能是没有足够的堆内存可供创建空闲任务。第二章提供了更多关于堆内存管理的信息。 */
 for( ;; );
}
```

清单110. 示例19的main()函数



执行示例19时产生的输出如图59所示。可以看出，中断接收了所有五个整数，并产生了五个字符串作为响应。图60中给出了更多的解释。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636337624983-fa5577b7-cf32-4898-927b-584773549def.png)

图59. 执行示例19时产生的输出

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636337712214-d0dc3f3d-9031-4fba-bf94-1675c821d4f7.png)

图60. 例19产生的执行顺序



## 中断嵌套

在任务优先级和中断优先级之间出现混淆是很常见的。本节讨论的是中断优先级，即中断服务例程（ISR）执行时相对于对方的优先级。分配给一个任务的优先级与分配给一个中断的优先级没有任何关系。硬件决定ISR的执行时间，而软件决定任务的执行时间。为响应硬件中断而执行的ISR将中断一个任务，但一个任务不能抢先执行ISR。



支持中断嵌套的端口需要在 FreeRTOSConfig.h 中定义表39中详述的一个或两个常量。

configMAX_API_CALL_INTERRUPT_PRIORITY 都定义了同一个属性。旧的 FreeRTOS 端口使用 configMAX_SYSCALL_INTERRUPT_PRIORITY ，而新的 FreeRTOS 端口使用 configMAX_API_CALL_INTERRUPT_PRIORITY 。



表 39. 控制中断嵌套的常量



| 恒定                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| configMAX_SYSCALL_INTERRUPT_PRIORITY or configMAX_API_CALL_INTERRUPT_PRIORITY | 设置最高的中断优先级，从中可以调用中断安全的FreeRTOS API函数。 |
| configKERNEL_INTERRUPT_PRIORITY                              | 设置tick中断所使用的中断优先级，并且必须始终设置为可能的最低中断优先级。如果使用的FreeRTOS端口没有同时使用configMAX_SYSCALL_INTERRUPT_PRIORITY 常量，那么任何使用中断安全的FreeRTOS API函数的中断也必须以 configKERNEL_INTERRUPT_PRIORITY 定义的优先级执行。 |

每个中断源都有一个数值优先级，和一个逻辑优先级：



- 数值优先级

数字优先级只是分配给中断优先级的数字。例如，如果一个中断被分配的优先级为7，那么它的数字优先级就是7。 同样，如果一个中断被分配的优先级为200，那么它的数字优先级就是200。

- 逻辑优先级

一个中断的逻辑优先级描述了该中断对其他中断的优先级。



如果两个不同优先级的中断同时发生，那么处理器将执行两个中断中逻辑优先级较高的那个中断的ISR，然后再执行两个中断中逻辑优先级较低那个中断的ISR。



一个中断可以中断（嵌套）任何具有较低逻辑优先级的中断，但一个中断不能中断（嵌套）任何具有相同或更高逻辑优先级的中断。



中断的数字优先级和逻辑优先级之间的关系取决于处理器结构。在某些处理器上，分配给一个中断的数字优先级越高，该中断的逻辑优先级就越高，而在其他处理器架构上，分配给一个中断的数字优先级越高，该中断的逻辑优先级就越低。



通过将 configMAX_SYSCALL_INTERRUPT_PRIORITY 设置为比 configKERNEL_INTERRUPT_PRIORITY 更高的逻辑中断优先级，可以创建一个完整的中断嵌套模型。这在图61中得到了证明，图中显示了这样一种情况：



- 处理器有七种独特的中断优先级。
- 分配给数字优先级为7的中断比分配给数字优先级为1的中断有更高的逻辑优先级。

- configKERNEL_INTERRUPT_PRIORITY 被设置为1。
- configMAX_SYSCALL_INTERRUPT_PRIORITY 被设置为3。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636338947247-e45b09e7-f928-400c-ac68-47b21a63983a.png)

图61. 影响中断嵌套行为的常量

参考图61：



- 当内核或应用程序处于关键部分时，使用优先级1到3（包括）的中断被阻止执行。以这些优先级运行的ISR可以使用中断安全的FreeRTOS API函数。关键部分的描述见第7章。
- 使用优先级为4或以上的中断不受关键部分的影响，因此在硬件本身的限制下，调度员所做的任何事情都不会阻止这些中断立即执行。以这些优先级执行的ISR不能使用任何 FreeRTOS的API函数。

- 通常，需要非常严格的时间精度的功能（例如电机控制）会使用高于configMAX_SYSCALL_INTERRUPT_PRIORITY的优先级，以确保调度器不会在中断响应时间中引入抖动。

### 对ARM Cortex-M1和ARM GIC用户的说明

Cortex-M处理器上的中断配置是混乱的，而且容易出错。为了帮助你的开发，FreeRTOS Cortex-M端口自动检查中断配置，但只有在 configASSERT() 被定义的情况下可以。configASSERT() 在第11.2节描述。

1本节仅部分适用于Cortex-M0和Cortex-M0+内核。

ARM Cortex内核和ARM通用中断控制器（GIC）使用数字上的低优先级数字来表示逻辑上的高优先级中断。这似乎有悖于直觉，而且很容易忘记。如果你想给一个中断分配一个逻辑上的低优先级，那么它必须被分配一个数字上的高值。如果你想给一个中断分配一个逻辑上的高优先级。 那么它就必须被分配一个数字上的低值。



Cortex-M中断控制器允许最多有8位用于指定每个中断的优先级，使255成为最低的优先级,零是最高优先级。然而，Cortex-M微控制器通常只实现8个可能的位数。实际实现的位数取决于微控制器系列。



当八个可能的位中只有一个子集被实现时，只有字节中最有意义的位可以被使用--留下最没有意义的位。未实现的位可以取任何值，但把它们设置为1是正常的。 图62展示了二进制101的优先级是如何存储在一个实现了四个优先位的Cortex-M微控制器中。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636339801928-abdc7bff-8d30-4479-86b1-3cfc39b1608b.png)

图62 二进制101的优先级是如何被一个实现了四个优先位的Cortex-M微控制器存储的



在图62中，二进制值101被移到了最有意义的四个位，因为最没有意义的四个位没有实现。未实现的位被设置为1。



一些库函数希望在优先权值被移到已执行的（最重要的）位子上之后再指定。当使用这样的函数时，图62中所示的优先级可以被指定为十进制95。十进制95是二进制101向上移位4，成为二进制101nnnn（其中"n"是一个未实现的位），未实现的位设置为1，成为二进制1011111。



一些库函数希望在优先权值被移到已执行的（最有意义的）位之前就被指定。当使用这样的函数时，图62中的优先权必须被指定为十进制5。十进制5是二进制101，没有任何移位。



configMAX_SYSCALL_INTERRUPT_PRIORITY 和 configKERNEL_INTERRUPT_PRIORITY 必须以允许它们直接写入Cortex-M寄存器的方式来指定，所以在优先级值被上移到实现的位后。



configKERNEL_INTERRUPT_PRIORITY必须始终设置为可能的最低中断优先级。 未实现的优先级位可以被设置为1，所以不管实际实现了多少个优先级位，这个常数总是可以被设置为255。



Cortex-M中断的默认优先级为0--可能的最高优先级。Cortex-M硬件的实现不允许configMAX_SYSCALL_INTERRUPT_PRIORITY 被设置为0，所以使用FreeRTOS API的中断的优先级决不能停留在其默认值。



# **资源管理**



## 引言及范围

在多任务系统中，如果一个任务开始访问资源，但在转换出运行状态之前没有完成其访问，则可能发生错误。如果任务使资源处于不一致的状态，那么任何其他任务或中断对相同资源的访问都可能导致数据损坏或其他类似问题。



以下是一些例子:

1. 访问外设

​      考虑以下场景，其中两个任务试图写入一个液晶显示器(LCD)。

1. 1. 任务A执行并开始写入字符串 “Hello world”到LCD。
   2. 在输出字符串“Hello w”的开头部分后，任务A被任务B抢占。

1. 1. 任务B在进入阻塞状态前将“中止, 重试, 失败?”写入LCD。
   2. 任务A从它被抢占的位置继续，并完成其字符串-“orld”的剩余字符的输出。

液晶显示器现在显示损坏的字符串“Hello w中止, 重试, 失败? orld”。

1. 读、修改、写操作

清单111显示了一行C代码，以及一个如何将C代码转换为汇编代码的示例。可以看到，PORTA的值首 先从内存中读入寄存器，在寄存器中修改，然后写回内存。这被称为读、修改、写操作。

```groovy
 /* 正在编译的C代码。*/
 PORTA |= 0x01; 
 
 /* 编译C代码时生成的汇编代码。*/
 LOAD   R1, [#PORTA]  ;Read a value from PORTA into R1
 MOVE   R2, #0x01     ; Move the absolute constant 1 into R2
 OR     R1, R2        ; Bitwise OR R1 (PORTA) with R2 (constant 1)
 STORE  R1, [#PORTA]  ; Store the new value back to PORTA
```

清单 111. 读、修改、写顺序的示例



这是一个“非原子”操作，因为它需要多个指令才能完成，并且可以被中断。考虑以下场景，其中两个任务试图更新名为PORTA的内存映射寄存器。

1. 1. 任务A将PORTA的值加载到寄存器中——操作的读部分。
   2. 任务A在完成同一操作的修改和写部分之前被任务B抢占。

1. 1. 任务B更新PORTA值。则进入阻塞状态。
   2. 任务A从它被抢占的位置继续执行。在将更新后的值写回PORTA之前，它修改已经保存在寄存器中的PORTA值的副本。



在这个场景中，任务A更新和回写PORTA的过期值。任务B在任务A获取PORTA值的副本之后修改PORTA，并且在任务A将其修改后的值写回PORTA寄存器之前修改PORTA。当任务A写入PORTA时，它会覆盖任务B已经执行的修改，从而有效地破坏PORTA寄存器值。



本例使用外设寄存器，但在对变量执行读、修改、写操作时也适用相同的原则。



1. 变量的非原子访问

更新结构的多个成员，或更新比体系结构的自然字长更大的变量（例如，在16位机器上更新32位变量），这些都是非原子操作的例子。如果它们被打断，可能会导致数据丢失或损坏。



1. 函数可重入性

如果从多个任务调用函数是安全的，那么该函数就是可重入的。或者来自任务和中断。重入函数被称为线程安全，因为它们可以从多个执行线程访问，而不会有数据或逻辑操作被破坏的风险。



每个任务维护自己的堆栈和自己的一组处理器(硬件)寄存器值。如果函数只访问存储在堆栈或寄存器中的数据，那么该函数是可重入的，线程安全的。清单112是一个 重入函数的例子。清单113是一个非可重入函数的例子。

```groovy
/* 传递一个参数给函数。这将被传递到堆栈上。或在处理器寄存器中。两种方法都是安全的，因为调用该函数的每个任务或中断都维护自己的堆栈和自己的寄存器值集，所以调用该函数的每个任务或中断都有自己的
1Var1副本。*/
long lAddOneHundred( long lVar1 )
{
/*该函数的作用域变量也将被分配到堆栈或寄存器，这取决于编译器和优化级别。每个调用该函数的任务或中断都有自己的1Var2副本。*/
long lVar2;
    
    lVar2 = lVar1 + 100;
    return lVar2;
}
```

清单112. 一个可重入函数的例子



```groovy
/*在本例中，1Varl是一个全局变量，因此每个调用1NonsenseFunction的任务都会访问该变量的同一个
副本。*/
long lVar1;

long lNonsenseFunction( void )
{
/* lstate是静态的，所以没有在堆栈上分配。每个调用该函数的任务都将访问变量的同一副本。*/
static long lState = 0;
long lReturn;
    
     switch( lState )
 	{
 		case 0 : lReturn = lVar1 + 10;
 				 lState = 1;
 				 break;
		case 1 : lReturn = lVar1 + 20;
 				 lState = 0;
 				 break;
 	}   
 }
```

清单113. 一个非可重入函数的例子



### 互斥

为了确保始终保持数据一致性，可以访问任务之间或任务与中断之间共享的资源。必须使用“互斥”进行管理。目标是确保，一旦一个任务开始访问不可重入且非线程安全的共享资源，在资源返回到一致状态之前，相同的任务对资源具有独占访问权。



FreeRTOS提供了几个可以用来实现互斥的特性。但是，最好的互斥方法是(只是可能，因为通常不实用)将应用程序设计成不共享资源的方式，并且只从单个任务访问每个资源。



### 范围

本章旨在让读者更好地理解：

- 何时以及为什么需要资源管理和控制。
- 多么关键的部分啊。

- 互斥是什么意思。
- 暂停调度器意味着什么。

- 如何使用互斥锁。
- 如何创建和使用看门人任务。

- 什么是优先级反转，以及优先级继承如何减少(而不是消除)它的影响。



## 关键部分和暂停调度器

### 基本的关键部分

基本临界区是由调用宏taskENTER_critical()和taskEXIT_CRITICAL()包围的代码区域有个。关键部分也是被称为临界区域。

taskENTER_CRITICAL()和taskEXIT_CRITICAL()不接受任何参数，或返回一个值1。注释清单114演示了它们的使用。

```groovy
/*确保对PORTA寄存器的访问不会被中断，将其置于临界区，进入临界区。*/
taskENTER_CRITICAL();

/*在调用taskENTER_CRITICAL()和调用taskEXIT_CRITICAL()之间不能切换到另一个任务。中断仍然可以在允许中断嵌套的EreeRTos端口上执行，但只允许逻辑优先级高于configMAX_SYSCALL_INTERRUPT_PRIORITY常量的值的中断执行，并且这些中断不允许调用FreeRTOS API函数。*/
PORTA |= 0x01;

/*对PORTA的访问已经完成，因此可以安全退出临界区。*/
taskEXIT_CRITICAL();
```

清单111.  使用临界区来保护对寄存器的访问



本书附带的示例项目使用了一个名为vPrintString()的函数来将字符串写入标准输出(即使用FreeRTOS Windows端口时的终端窗口)。vPrintString()从许多不同的任务调用;因此，理论上，它的实现可以使用临界区保护对标准输出的访问，如清单115所示。

值1：像macro这样的函数并不像实际函数那样“返回一个值”。本书将术语“返回值”应用到宏上，最简单的做法是把宏看成一个函数。



```groovy
void vPrintString( const char *pcString )
{
 	/*将字符串写入标准输出，使用临界区作为一个粗略的互斥方法。 */
 	taskENTER_CRITICAL();
 	{
 		printf( "%s", pcString );
 		fflush( stdout );
 	}
 	taskEXIT_CRITICAL();
}
```

清单115.  vPrintString()的一个可能实现



以这种方式实现的临界区是一种提供互斥的非常粗糙的方法。它们通过禁用中断来工作，要么完全禁用，要么直到configMAX_SYSCALL_INTERRUPT_PRIORITY设置的中断优先级—取决于正在使用的FreeRTOS por。抢占式的上下文切换只能发生在一个中断中，因此，只要中断保持禁用，调用taskENTER_CRITICAL()的任务就会被保证保持在运行状态，直到临界区段退出。



基本临界段必须保持很短，否则它们将对中断响应时间产生不利影响。对taskENTER_CRITICAL()的每个调用都必须与对taskEXIT_CRITICAL()的调用紧密匹配。因此，不应该使用临界区来保护标准输出(stdout，即计算机写入其输出数据的流)(如清单115所示)，因为向终端写入数据可能是一个相对较长的操作。本章的例子探讨了可选的解决方案。



关键部分嵌套是安全的，因为内核保留了嵌套深度的计数。只有当嵌套深度返回0时，即每次调用taskEXIT_CRITICAL()都执行一次对taskENTER_CRITICAL()的调用，临界区才会退出。



调用taskENTER_CRITICAL()和taskEXIT_CRITICAL()是任务改变FreeRTOS运行的处理器的中断启用状态的唯一合法方法。通过任何其他方法改变中断启用状态都会使宏的嵌套计数无效。



taskENTER_CRITICAL()和taskEXIT_CRITICAL()不会以“FromlSR”结束，所以一定不能从中断服务程序中调用。taskENTER_CRITICAL_FROM_ISR() 是一个中断安全版本的taskENTER_CRITICAL()，和taskEXIT_CRITICAL_FROM_ISR()是一个允许中断嵌套的FreeRTOS端口，它们将在不允许中断嵌套的端口中被废弃。



taskENTER_CRITICAL_FROM_ISR()返回一个值，这个值必须被传递到匹配的调用taskEXIT_CRITICAL_FROM_ISR()。清单116演示了这一点。

```groovy
void vAnInterruptServiceRoutine( void )
{
/*声明一个变量，该变量的返回值来自taskENTER_CRITICAL_FROM_ISR()将被保存。*/
UBaseType_t uxSavedInterruptStatus;
    
	/* ISR的这一部分可以被任何更高优先级的中断中断。*/ 
	/*使用taskENTER_CRITICAL_FROM_ISR()来保护该ISR的一个区域。保存从taskENTER_CRITICAL_FROM_ISR()返回的值，以便它可以被传递到匹配的调用taskEXIT_CRITICAL_FROM_ISR()。*/
	uxSavedInterruptStatus = taskENTER_CRITICAL_FROM_ISR();
 
	/* ISR的这一部分在调用taskENTER_CRITICAL_FROM_ISR()和taskEXIT_CRITICAL_FROM_ISR()之间，所	以只能被优先级高于configMAX_SYSCALL_INTERRUPT_PRIORITY常量设置的优先级。*/
    
	/*通过调用taskEXIT_CRITICAL_FROM_ISR()再次退出临界区，并传入由匹配的调用返回的值	taskENTER_CRITICAL_FROM_ISR()。*/
 	taskEXIT_CRITICAL_FROM_ISR( uxSavedInterruptStatus );
	/* ISR的这一部分可以被任何更高优先级的中断中断。*/
}
```

清单116.  在中断服务程序中使用临界区



使用更多的处理时间执行进入并随后退出临界区段的代码，而不是执行实际受临界区段保护的代码，这是一种浪费。基本临界区段进入和退出都非常快，而且总是确定性的，因此当受保护的代码区域非常短时，它们的使用非常理想。



### 挂起(或锁定)调度器

还可以通过挂起调度器来创建临界区。暂停调度器有时也被称为“锁定”调度器。



基本临界区保护代码区域不受其他任务和中断的访问。通过暂停调度器实现的临界区仅保护代码区域不被其他任务访问。因为中断仍然处于启用状态。



如果临界区段太长，不能简单地通过禁用中断来实现，则可以通过暂停调度程序来实现。然而，在调度器挂起时中断活动会使恢复(或“取消挂起”)调度器成为一个相对较长的操作，因此必须考虑在每种情况下使用哪种方法最好。



### vTaskSuspendAll() API函数

```groovy
void vTaskSuspendAll( void );
```

清单 117.  vTaskSuspendAll() API函数原型



调度程序通过调用vTaskSuspendAll()被挂起。挂起调度程序可以防止发生上下文切换，但保留中断。如果在调度器挂起时中断请求上下文切换，则请求将被挂起。并且只在恢复(未挂起)调度程序时执行。



当调度器挂起时，不能调用FreeRTOS API函数。



### xTaskResumeAll()的API函数

```groovy
BaseType_t xTaskResumeAll( void );
```

清单118.  xTaskResumeAll() API函数原型



通过调用xTaskResumeAl()恢复(未挂起)调度器。



表 40. xTaskResumeAll()的返回值

| 返回值 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 返回值 | 在调度器挂起时请求的上下文切换将被挂起并仅在恢复调度器时执行。如果在xTaskResumeAll()返回之前执行了挂起的上下文切换，则返回pdTRUE。否则返回pdFALSE。 |



对vTaskSuspendAll()和xTaskResumeAl()的调用嵌套是安全的，因为内核保留了嵌套深度的计数。调度器只有在嵌套深度返回0时才会恢复，也就是在每次调用vTaskSuspendAll()都执行一次xTaskResumeAll()时。



清单119显示了vPrintString()的实际实现，它挂起调度程序以保护对终端输出的访问。

```groovy
void vPrintString( const char *pcString )
{
 	/*将字符串写入标准输出，以互斥的方式挂起调度程序*/
 	vTaskSuspendScheduler();
 	{
 		printf( "%s", pcString );
 		fflush( stdout );
 	}
 	xTaskResumeScheduler();
}
```

清单119. vPrintString()的实现



### 互斥锁(和二进制信号量)

互斥锁是一种特殊类型的二进制信号量，用于控制对两个或多个任务共享的资源的访问。MUTEX这个词来源于“互斥”。configUSE_MUTEXES必须在FreeRTOSConfig.h中设置为1，互斥才能可用。



当在互斥场景中使用互斥时，可以将互斥看作与共享资源相关联的令牌。以便任务合法地访问资源。它必须首先成功地“接受”令牌(作为令牌持有者)。当令牌持有者使用完资源后，它必须“归还”令牌。只有当该令牌被返回时，另一个任务才能成功地获取该令牌，然后安全地访问相同的共享资源。任务不允许访问共享资源，除非它持有令牌。这种机制如图63所示。



尽管互斥锁和二进制信号量共享许多特性。图63所示的场景(其中使用互斥锁进行互斥)与图53所示的场景(其中使用二进制信号量进行同步)完全不同。主要的区别在于信号量被获取后发生了什么：

- 必须始终返回用于互斥的信号量。
- 用于同步的信号量通常被丢弃且不返回。

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/23129846/1636013046498-a572266e-5d62-4262-8ab5-36482102960d.jpeg)

图63 使用互斥对象实现的互斥



该机制纯粹通过应用程序编写人员的规程来工作。没有理由一个任务不能在任何时候访问资源，但是每个任务“同意”不这样做。除非它能成为互斥锁持有者。



### xSemaphoreCreateMutex() API函数

FreeRTOS V9.0.0还包括xSemaphoreCreateMutexStatic()函数，该函数在编译时分配静态创建互斥锁所需的内存:互斥锁是一种信号量类型。所有不同类型的FreeRTOS信号量的句柄都存储在SemaphoreHandle_t类型的变量中。



在使用互斥锁之前，必须先创建它。要创建互斥量类型的信号量，请使用xSemaphoreCreateMutex() APl函数。

```groovy
SemaphoreHandle_t xSemaphoreCreateMutex( void );
```

清单120. xSemaphoreCreateMutex() API函数原型



表 41. xSemaphoreCreateMutex()的返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| 返回值          | 如果返回NULL，则无法创建互斥锁，因为没有足够的堆内存供FreeRTOS分配互斥锁数据结构。第2章提供了堆内存管理的更多信息。非null返回值表明互斥锁已经成功创建。返回值应该存储为创建的互斥锁的句柄。 |



### 例20. 重写vPrintString()以使用信号灯

本例创建了vPrintString()的新版本，称为prvNewPrintString()，然后从多个任务中调用这个新函数。prvNewPrintString()在功能上与vPrintString()相同，但使用互斥锁来控制对标准输出的访问，而不是通过锁定调度器。清单121显示了prvNewPrintString()的实现。

```groovy
static void prvNewPrintString( const char *pcString )
{
	/*互斥锁是在调度器启动之前创建的，所以在任务执行时已经存在尝试获取互斥锁，如果互斥锁不能立即可用，		则无限期阻塞以等待它。对xSemaphoreTake()的调用只有在成功获得互斥锁时才会返回。所以不需要检查函		 数的返回值。如果使用了任何其他延迟周期，那么代码必须在访问共享资源(在本例中是标准输出)之前检查 		xSemaphorerake()是否返回pdTRUE。正如本书前面提到的，不建议在生产代码中使用无限期超时。*/
 	xSemaphoreTake( xMutex, portMAX_DELAY );
 	{
	/*只有成功获得互斥锁后，才会执行下面的行。现在可以自由访问标准输出，因为在任何时候只有一个任务可以		拥有互斥锁。*/
	 printf( "%s", pcString );
     fflush( stdout );
        
    /* 互斥锁必须返回! */
 	}
 	xSemaphoreGive( xMutex );
}
```

清单121. prvNewPrintString()的实现



prvNewPrintString()由prvPrintTask()实现的任务的两个实例反复调用。每次调用之间使用随机延迟时间。task参数用于向任务的每个实例传递唯一的字符串。清单122显示了prvPrintTask()的实现。

```groovy
static void prvPrintTask( void *pvParameters )
{
char *pcStringToPrint;
const TickType_t xMaxBlockTimeTicks = 0x20;
 	/*创建该任务的两个实例。任务打印的字符串“使用任务的参数传递给任务”。参数被转换为所需的类型。*/
 	pcStringToPrint = ( char * ) pvParameters;
    
 	for( ;; )
 	{
 		/*使用新定义的函数输出字符串。*/
 		prvNewPrintString( pcStringToPrint );
		/*等待一个伪随机时间。请注意rand()不一定是可重入的，但在这种情况下，它实际上并不重要，因为代			码并不关心返回的值是什么。在更安全的应用程序中，应该使用已知可重入的rand()版本——或者应该在		   临界区段保护rand()的调用。*/
 		 vTaskDelay( ( rand() % xMaxBlockTimeTicks ) );
 	} 
}
```

清单122.  例20中prvPrintTask()的实现



正常情况下，main()只是创建互斥锁，创建任务，然后启动调度器。实现如清单123所示。



prvPrintTask()的两个实例以不同的优先级创建，因此优先级较低的任务有时会被优先级较高的任务抢占。由于使用互斥锁来确保每个任务对终端的访问是互斥的，所以即使发生了抢占，被删除的字符串也将是正确的，并且不会损坏。通过减少任务处于阻塞状态的最大时间，可以增加抢占的频率，该时间是由xMaxBlockTimeTicks常量设置的。



使用FreeRTOS Windows端口的示例20的注意事项:

- 调用printf()将生成一个Windows系统调用。Windows系统调用不在FreeRTOS的控制范围之内，并且会带来不稳定性。
- Windows系统调用的执行方式意味着，即使没有使用互斥锁，也很少看到损坏的字符串。

```groovy
int main( void )
{
	/*在使用信号量之前，必须显式地创建信号量。在这个例子中，创建了一个互斥量类型的信号量。*/
 	xMutex = xSemaphoreCreateMutex();
 	/*在创建任务之前检查信号量是否创建成功。*/
 	if( xMutex != NULL )
 	{
		/*创建两个写入stdout的任务实例。他们写入的字符串作为任务的参数传递给任务。任务按不同的优先级			创建，因此会发生一些抢占。*/
 		xTaskCreate( prvPrintTask, "Print1", 1000, 
 					"Task 1 ***************************************\r\n", 1, NULL );
 		xTaskCreate( prvPrintTask, "Print2", 1000, 
 					"Task 2 ---------------------------------------\r\n", 2, NULL );
        
		/*启动调度程序，使创建的任务开始执行。*/
 		vTaskStartScheduler();
 }
 
/*如果一切正常，那么main()将永远不会到达这里，因为调度程序现在将运行任务。如果main()确实到达了这里，那么很可能是没有足够的堆内存可用来创建空闲任务。第2章提供了堆内存管理的更多信息。*/
 for( ;; );
}
```

清单123. 例20的main()的实现



执行例20时产生的输出如图64所示。图65描述了可能的执行顺序

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636015921770-e7998136-2952-4ab0-85cf-b2098474a4c4.png)

图64. 执行例20时产生的输出



如图64所示，终端上显示的字符串没有损坏。随机排序是任务所使用的随机延迟周期的结果。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636016039460-db7c1686-d562-441a-b61c-13b7c2b2e5d0.png)

图65. 例20中可能的执行序列



### 优先级反转

图65演示了使用互斥锁提供互斥的一个潜在缺陷。所描述的执行顺序显示了高优先级任务2必须等待低优先级任务1放弃对互斥锁的控制。高优先级的任务被低优先级的任务以这种方式延迟，我们称之为“优先级反转”。如果一个中等优先级的任务在高优先级任务等待信号量的同时开始执行，这种不受欢迎的行为将进一步扩大——结果将是一个高优先级任务在等待一个低优先级任务——而低优先级任务甚至无法执行。图66显示了这种最糟糕的情况。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636016540177-16c01691-37c6-413e-a887-810009f27c71.png)

图66 最坏情况下的优先权反转情况



优先级反转可能是一个严重的问题，但在小型嵌入式系统中，通过考虑如何访问资源，通常可以在系统设计时避免这个问题。



### 优先级继承

FreeRTOS互斥体和二进制信号量非常相似，不同之处是互斥体包含一个基本的优先级继承机制。而二进制信号量则不然。优先级继承是一种最小化优先级倒置负面影响的方案。它并不修复优先级倒置，只是通过确保倒置总是有时间限制来减少其影响。但是，优先级继承使系统定时分析变得复杂，依赖它来进行正确的系统操作并不是一个好的实践。



优先级继承的工作原理是，暂时将互斥锁持有者的优先级提高到试图获取同一个互斥锁的优先级最高的任务的优先级。持有互斥锁的低优先级任务继承了等待互斥锁的任务的优先级。图67演示了这一点。互斥锁持有者的优先级在返回互斥锁时自动重置为初始值。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636016800950-a3500eeb-745b-4217-b4a4-ae26f73ce0ee.png)

图67. 优先权继承使优先权反转的影响最小化



正如刚才看到的，优先级继承功能会影响使用互斥锁的任务的优先级。因此，互斥锁不能在中断服务程序中使用。



### 死锁（或致命拥抱）

“死锁”是使用互斥对象进行互斥的另一个潜在陷阱。僵局有时也被称为“致命拥抱”。



当两个任务都在等待对方持有的资源而无法继续时，就会发生死锁。考虑以下场景，任务A和任务B都需要获取互斥锁X和互斥锁Y来执行一个操作：

1. 任务A执行并成功接受互斥锁X。

2. 任务A被任务B抢占。

3. 任务B在尝试获取互斥锁X之前成功获取了互斥锁Y，但是互斥锁X被任务A持有，所以对任务B不可用。任务B选择进入阻塞状态，等待互斥锁X被释放。

4. 任务A继续执行。它尝试取互斥量Y，但是互斥量Y被任务B持有，所以不能被任务A	使用。任务A选择进入阻塞状态来等待互斥量Y被释放。

在这个场景的最后，任务A正在等待任务B持有的互斥锁，任务B也在等待任务A持有的互斥锁。



与优先级倒置一样，避免死锁的最佳方法是在设计时考虑它的潜力，并设计系统以确保不会发生死锁。特别是，正如本书前面所述，任务无限期地等待(没有超时)来获得互斥锁通常是不好的做法。相反，使用比期望等待互斥锁的最大时间稍长一点的超时时间——那么在此时间内无法获得互斥锁将是设计错误的症状，可能是死锁。



在实践中，死锁在小型嵌入式系统中不是一个大问题，因为系统设计人员可以很好地理解整个应用程序，因此可以识别和消除可能发生死锁的区域。



### 递归互斥锁

任务本身也有可能死锁。如果任务试图执行，就会发生这种情况。多次使用同一个互斥锁，而不首先返回互斥锁。考虑以下场景：

1. 任务成功获取互斥锁。

2. 在持有互斥锁时，任务调用库函数

3. 库函数的实现尝试获取相同的互斥锁，并进入阻塞状态以等待互斥锁可用。

在本场景结束时，任务处于阻塞状态，等待返回互斥锁，但任务已经是互斥锁持有者了。出现死锁，因为任务处于阻止状态以等待其自身。



这种类型的死锁可以通过使用递归锁来代替标准互斥锁来避免。一个递归锁可以被同一个任务多次使用。并且只有在每次调用“取”递归互斥时执行一次“给”递归后才返回。



标准互斥体和递归互斥体的创建和使用方式类似:

- 标准互斥对象是使用xSemaphoreCreateMutex()创建的。递归互斥是使用xSemaphoreCreateRecursiveMutex()创建的。这两个API函数具有相同的原型。
- 标准互斥对象使用xSemaphoreTake()来“获取”。使用xSemaphoreTakeRecursive()“获取”递归互斥。这两个API函数具有相同的原型。

- 标准互斥对象使用xsemaphoregve()来“给定”。递归互斥是使用xSemaphoreTakeRecursive()给出的。这两个API函数具有相同的原型。



清单124. 演示了如何创建和使用递归锁。

```groovy
/*递归互斥是SemaphoreHandle_t类型的变量。 */
SemaphoreHandle_t xRecursiveMutex;

/* 创建和使用递归互斥锁的任务的实现。 */
void vTaskFunction( void *pvParameters )
{
const TickType_t xMaxBlock20ms = pdMS_TO_TICKS( 20 );
 	/* 在使用递归互斥锁之前，必须显式地创建它。 */
 	xRecursiveMutex = xSemaphoreCreateRecursiveMutex();
    
	/*检查信号量是否创建成功。configASSERT()在11.2节中有描述。*/
 	configASSERT( xRecursiveMutex );
    
 	/*对于大多数任务，这个任务是作为一个无限循环来实现的。*/
 	for( ;; )
 	{
		/* ... */
        /*取递归互斥锁。 */
        if( xSemaphoreTakeRecursive( xRecursiveMutex, xMaxBlock20ms ) == pdPASS )
        {
            /*成功获取递归互斥锁。任务现在可以访问互斥锁保护的资源。此时，递归调用计数(即对					xSemaphoreTakeRecursive()的嵌套调用的数量)为1，因为递归互斥对象只被取过一次。* /
            
 			/*当它已经持有递归的互斥对象时，任务再次接受互斥对象。在实际的应用程序中这只可能发生			    在该任务调用的子函数中，因为没有实际的理由要多次使用同一个互斥锁。调用任务已经是互斥			   锁的持有者，所以对xSemaphorerakeRecursive()的第二次调用只是将递归调用的声音增加到2。*/
 		   xSemaphoreTakeRecursive( xRecursiveMutex, xMaxBlock20ms );
            
 			/* ... */
            /*任务在访问了互斥锁保护的资源后返回互斥锁。此时，递归调用计数为2，因此对							xSemaphoreGiveRecursive()的第一次调用不返回互斥锁。相反它只是将递归调用计数减回1。*/
 			xSemaphoreGiveRecursive( xRecursiveMutex );
            
			/*下一次调用xSemaphoreGiveRecursive()时，返回的调用计数减少为0。所以这次返回的是递归			  互斥锁。*/
 			xSemaphoreGiveRecursive( xRecursiveMutex );
            
			/*现在每次调用xSemaphoreGiveRecursive()都会执行一个xSemaphoreGiveRecursive()，所以				任务不再是互斥锁的持有者。*/
		}
	} 
}
```

清单124.  创建和使用递归互斥锁



### 互斥体和任务调度

如果两个具有不同优先级的任务使用同一个互斥锁，那么FreeRTOS调度策略会明确任务执行的顺序;能够运行的最高优先级任务将被选择为进入运行状态的任务。例如，如果高优先级任务处于阻塞状态，等待低优先级任务持有的互斥锁，那么一旦低优先级任务返回互斥锁，高优先级任务就会抢占低优先级任务。高优先级的任务将成为互斥锁的持有者。在图67中已经看到了这个场景。



然而，当任务具有相同的优先级时，通常会对任务的执行顺序做出错误的假设。如果任务1和任务2有相同的优先级。任务1处于阻塞状态，等待由任务2持有的互斥，那么当任务2 "给出 "突变时，任务1不会抢占任务2。相反，任务2将保持在 运行状态，而任务1将简单地从阻塞状态移动到就绪状态。这种情况由图68显示了，其中垂直线标记了勾号中断发生的时间。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636018020725-90971f22-7bdb-457e-a772-b0aa1f623fe6.png)

图68 具有相同优先级的任务使用同一个互斥锁时可能的执行顺序



在图68所示的场景中，当互斥锁可用时，FreeRTOS调度器不会让任务一成为运行状态任务，因为:

1. 任务1和任务2具有相同的优先级，所以除非任务2进入阻塞状态，否则在下一次tick中断之前不会切换到任务1(假设configUSE TIME SLICING在FreeRTOSConfig.h中设置为1)。
2. 如果一个任务在紧循环中使用了一个互斥锁，并且每次任务给出互斥锁时都会发生上下文切换，那么这个任务只会在很短的时间内保持运行状态。如果两个或多个任务在一个紧循环中使用同一个互斥锁，那么在任务之间快速切换会浪费处理时间。



如果一个互斥锁在一个紧循环中被多个任务使用，并且使用互斥锁的任务具有相同的优先级，那么必须小心确保任务收到大约相等的处理时间。图69演示了任务可能无法获得相同数量的处理时间的原因。它显示了以相同优先级创建清单125所示任务的两个实例时可能发生的执行序列。

```groovy
/*在紧循环中使用互斥锁的任务的实现。该任务在本地缓冲区中创建一个文本字符串，然后将该字符串写入显示。对显示器的访问受互斥锁的保护。*/
void vATask( void *pvParameter )
{
extern SemaphoreHandle_t xMutex;
char cTextBuffer[ 128 ];
    for( ;; )
	{
		/* 生成文本字符串-这是一个快速的操作。 */
        vGenerateTextInALocalBuffer( cTextBuffer );
        
		/*获取保护对显示的访问的互斥锁。*/
 		xSemaphoreTake( xMutex, portMAX_DELAY );
        
		/* 将生成的文本写入显示器-这是一个缓慢的操作。*/
		vCopyTextToFrameBuffer( cTextBuffer );
        
 		/* 文本已经写入显示器，因此返回互斥锁。*/
 		xSemaphoreGive( xMutex );
 	} 
}
```

清单125. 一个在紧密循环中使用互斥的任务



清单125中的注释注意到，创建字符串是一个快速的操作，而更新显示则是一个缓慢的操作。因此，当显示被更新时，互斥锁被持有，任务将在大部分运行时间内持有互斥锁。



在图69中，垂直线标记了记号中断发生的时间。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636018507067-3d56d6a9-cf05-44e0-a084-80d0db8b5523.png)

图69 任务的两个实例可能发生的执行序列如清单125所示，以相同的优先级创建



图69中的第7步显示任务1重新进入阻塞状态，这发生在 xSemaphoreTake() API函数中发生。



图69表明，任务1将被阻止获得互斥锁，直到时间片的开始与任务2不是互斥锁持有者的短时间的重合。



图69中所示的情况可以通过在调用taskYIELD()后添加一个调用来避免。调用xSemaphoreGive()后，可以避免这种情况。这在清单126中得到了证明，如果在任务持有互斥锁时taskYIELD()被调用，那么它就会被调用。如果在任务持有互斥锁的时候，标记计数发生了变化，就会调用taskYIELD()。

```groovy
void vFunction( void *pvParameter )
{
extern SemaphoreHandle_t xMutex;
char cTextBuffer[ 128 ];
TickType_t xTimeAtWhichMutexWasTaken;
 
    for( ;; )
    {
 		/* 生成文本字符串-这是一个快速的操作。 */
 		vGenerateTextInALocalBuffer( cTextBuffer );
        
 		/*获取对显示器进行protectinq访问的互斥锁。 */
 		xSemaphoreTake( xMutex, portMAX_DELAY );
        
 		/* 记录使用互斥锁的时间。 */
 		xTimeAtWhichMutexWasTaken = xTaskGetTickCount();
        
 		/* 将生成的文本写入显示器——这是一个缓慢的操作。 */
 		vCopyTextToFrameBuffer( cTextBuffer );
        
 		/* 文本已经写入显示器，因此返回互斥锁。*/
 		xSemaphoreGive( xMutex );
        
		/*如果每次迭代都调用taskYIELD()，那么该任务只会在很短的一段时间内保持运行状态，而在任务之间		快速切换会浪费处理时间。因此，只有当在互斥锁被持有时时钟计数改变时才调用taskYIELD()。* /
 		if( xTaskGetTickCount() != xTimeAtWhichMutexWasTaken )
 		{
 			taskYIELD();
		 }
 	} 
}
```

清单126. 确保在循环中使用互斥锁的任务可以获得更多的处理时间，同时也确保不会因为在任务之间切换太快而浪费处理时间。



## 看门人任务

看门人任务提供了一种干净的实现互斥的方法，并且没有优先级倒置或死锁的风险。



看门人任务是对资源拥有唯一所有权的任务。只有看门人任务被允许直接访问资源，其他需要访问资源的任务只能通过使用看门人的服务间接访问资源。



### 示例21. 重写vPrintString()以使用看门人任务

示例21提供了vPrintString()的另一种替代实现。这一次，使用一个看门人任务来管理对标准输出的访问。当任务想要将消息写入标准输出时，它不会直接调用print函数，而是将消息发送给看门人。



看门人任务使用FreeRTOS队列将访问序列化到标准输出。任务的内部实现不必考虑互斥，因为它是唯一允许直接访问标准输出的任务。



看门人任务大部分时间处于阻塞状态，等待消息到达队列。当消息到达时，看门人只需将消息写入标准输出，然后返回阻塞状态以等待下一条消息。看门人任务的实现如清单128所示。



中断可以发送到队列，因此中断服务程序也可以安全地使用看门人的服务向终端写入消息。在本例中，使用了一个钩子函数，每200次写一条消息。



滴答钩子(或滴答回调)是内核在每次滴答中断期间调用的函数。使用滴答钩子函数:

1. 在FreeRTOSConfig.h中设置configUSE_TICK_HOOK为1。
2. 提供钩子函数的实现，使用清单127所示的准确的函数名称和原型如清单127所示。

```groovy
void vApplicationTickHook( void );
```

清单127.  钩子函数的名称和原型



钩子函数在勾选中断的上下文中执行，因此必须保持非常短的时间。所以必须保持非常短，必须只使用适量的堆栈空间，并且不能调用任何不以“FromISR()”结尾的FreeRTOS API 函数，而不是以 “FromISR() ”结尾。



调度器将总是在滴答钩子函数之后立即执行，所以中断安全。从滴答钩子调用的FreeRTOS API函数不需要使用其 pxHigherPriorityTaskWoken参数，并且该参数可以被设置为NULL。

```groovy
static void prvStdioGatekeeperTask( void *pvParameters )
{
char *pcMessageToPrint;
    
    /*这是唯一允许写入标准输出的任务。任何其他想要将字符串写入输出的任务都不会直接访问标准输出，而是将		字符串发送给该任务。因为只有该任务访问标准输出，所以在任务本身的实现中不需要考虑互斥或序列化问		  题。*/
 	for( ;; )
	{
		/*等待消息到达。指定了一个不确定的块时间，因此不需要检查返回值-该函数只在成功接收到消息时返			回。*/
 		xQueueReceive( xPrintQueue, &pcMessageToPrint, portMAX_DELAY );
        
 		/* Output the received string. */
 		printf( "%s", pcMessageToPrint );
 		fflush( stdout );
        
 		/*返回循环以等待下一条消息。*/
 	} 
}
```

清单128.  看门人任务



写入队列的任务如清单129所示。与前面一样，将创建任务的两个独立实例，并使用任务参数将任务写入队列的字符串传递给任务。

```groovy
static void prvPrintTask( void *pvParameters )
{
int iIndexToString;
const TickType_t xMaxBlockTimeTicks = 0x20;
    
    /*创建该任务的两个实例。任务参数用于将字符串数组的索引传递给任务。将其转换为所需的类型。*/
 	iIndexToString = ( int ) pvParameters;
    
    for( ;; )
 	{
        /*输出字符串，不是直接输出，而是通过队列将字符串的指针传递给看门人任务。队列是在调度程序			启动之前创建的，因此在该任务第一次执行时已经存在。没有指定块时间，因为队列中总是有空间。*/
 		xQueueSendToBack( xPrintQueue, &( pcStringsToPrint[ iIndexToString ] ), 0 );
        
        /*等待一个伪随机时间。请注意rand()不一定是可重入的，但在这种情况下，它实际上并不重要，因为代		  码并不关心返回的值是什么。在更安全的应用程序中，应该使用已知可重入的rand()版本——或者应该使		    用临界区来保护对rand()的调用。*/
 		vTaskDelay( ( rand() % xMaxBlockTimeTicks ) );
 	}
}
```

清单129.  例21的打印任务实现



滴答钩子函数对自己被调用的次数进行计数，每次计数达到200时，就向看门人任务发送消息。仅出于演示目的，滴答钩子写到队列的前面，而任务写到队列的后面。滴答钩子实现如清单130所示。

```groovy
void vApplicationTickHook( void )
{
static int iCount = 0;

	/*每200个节拍打印一条消息。消息不是直接写出来的，而是发送给看门人任务。*/
 	iCount++;
 	if( iCount >= 200 )
 	{
		/*由于xQueueSendToFrontFromISR()是从标记钩子调用的，所以不需要使用								xHigherPriorityTaskwoken参数(第三个参数)，该参数设置为NULL。*/
 		xQueueSendToFrontFromISR( xPrintQueue, 
 							  	  &( pcStringsToPrint[ 2 ] ), 
 							      NULL );
 
 		/* 重置计数，准备在200滴答时间内再次打印字符串。 */
 		iCount = 0;
 	} 
}
```

清单130.  滴答钩子函数的实现



正常情况下，main()创建运行示例所需的队列和任务，然后启动调度程序。main()的实现如清单131所示。

```groovy
/*定义任务和中断将通过看门人打印出来的字符串。 */
static char *pcStringsToPrint[] =
{
 	"Task 1 ****************************************************\r\n",
 	"Task 2 ----------------------------------------------------\r\n",
 	"Message printed from the tick hook interrupt ##############\r\n"
};
    
/*-----------------------------------------------------------*/
/*声明一个类型为QueueHandle_t的变量。该队列用于发送消息，从打印任务和标记中断到看门人任务。*/
QueueHandle_t xPrintQueue;
/*-----------------------------------------------------------*/

int main( void )
{
	/*在使用队列之前，必须显式创建队列。队列被创建为最多容纳5个字符的指针。*/
 	xPrintQueue = xQueueCreate( 5, sizeof( char * ) );
 	/* 检查队列是否创建成功。 */
 	if( xPrintQueue != NULL )
 	{
        /*创建任务的两个实例，发送消息给守门人。任务使用的字符串的索引通过任务参数传递给任务				(xTaskcreate()的第4个参数)。这些任务按不同的优先级创建，因此高优先级任务偶尔会抢占低优先级		 任务。*/
 		xTaskCreate( prvPrintTask, "Print1", 1000, ( void * ) 0, 1, NULL );
 		xTaskCreate( prvPrintTask, "Print2", 1000, ( void * ) 1, 2, NULL );
       
        /*创建看门人任务。这是唯一允许直接访问标准输出的任务。*/
 		xTaskCreate( prvStdioGatekeeperTask, "Gatekeeper", 1000, NULL, 0, NULL );
 
 		/* 启动调度程序，使创建的任务开始执行。 */
 		vTaskStartScheduler();
 	}
    
    /*如果一切正常，那么main()将永远不会到达这里，因为调度程序现在将运行任务。如果main()确实到达了这		里，那么很可能是没有足够的堆内存可用来创建空闲任务。第2章提供了堆内存管理的更多信息。*/
 	for( ;; );
}
```

清单131.  例21的main()的实现



执行示例21时产生的输出如图70所示。可以看到，来自任务的字符串和来自中断的字符串都正确输出，没有损坏。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636078578045-f89e7516-8666-4c7d-9fa8-aabc2e5f588e.png)

图70  执行例21时产生的输出



看门人任务的优先级比打印任务低，因此发送给看门人的消息会一直留在队列中，直到两个打印任务都处于阻塞状态。在某些情况下，为看门人分配更高的优先级是合适的，这样消息就可以立即得到处理——但是这样做的代价是，看门人会延迟较低优先级的任务，直到完成对受保护资源的访问。





# **事件组**



## 引言与范围

人们已经注意到，实时嵌入式系统必须对事件采取行动。前几章描述了FreeRTOS允许事件与任务通信的特性。这类特性的例子包括信号量和队列，它们都具有以下属性:

- 它们允许任务在阻塞状态中等待单个事件的发生。
- 当事件发生时，它们解除阻塞单个任务——解除阻塞的任务是等待事件的最高优先级的任务。



事件组是FreeRTOS的另一个特性，它允许事件与任务通信。与队列和信号量不同：

- 事件组允许任务在阻塞状态下等待多个事件的组合发生。
- 事件组在事件发生时解除等待同一事件或事件组合的所有任务的阻塞。



事件组的这些独特的性质使其有用的多个任务同步广播事件的多个任务,允许任务阻塞状态等待任何发生的一系列事件之一,并允许一个任务在阻塞状态等多个操作完成。



事件组还提供了减少应用程序使用的RAM的机会，因为通常可以用单个事件组替换许多二进制信号量。



事件组功能是可选的。要包含事件组功能，构建FreeRTOS源文件event_groups.c作为项目的一部分。



### 范围

本章旨在让读者更好地理解:

- 事件组的实际用途。
- 事件组相对于其他FreeRTOS特性的优点和缺点。

- 如何设置事件组中的位。
- 如何在阻塞状态下等待事件组中的位被设置。

- 如何使用事件组同步一组任务。



## 事件组的特征

### 事件组、事件标志和事件位

事件 "标志 "是一个布尔值（1或0），用于指示一个事件是否发生。事件 "组 "是一组事件标志。



事件标志只能为1或0，允许事件标志的状态存储在单个位中，事件组中所有事件标志的状态存储在单个变量中；事件组中每个事件标志的状态由类型为EventBits_t的变量中的单个位表示。因此，事件标志也被称为事件“位”。如果EventBits_t变量中的一位被设为1，则该位表示的事件已经发生。如果在EventBits_t变量中一个位被设置为0。那么由该位表示的事件没有发生。



图71显示了如何将单个事件标志映射到类型变量中的单个位EventBits_t。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636082654638-c4484f4f-5c5b-4348-bcbd-3d96004c940b.png)

图71 EventBits_t类型变量中的事件标志到位号的映射



例如，如果一个事件组的值是0x92(二进制1001 0010)，那么只有事件位1、4和7被设置，因此只有由1、4和7表示的事件发生。图72显示了EventBits_t类型的变量，设置了事件位1、4和7，并清除了所有其他事件位，使事件组的值为0x92。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636082893303-94a007a8-cf5e-46b1-a4f5-caab3caaed2c.png)

图72 一个事件组，其中只设置了1、4和7位，并且清除了所有其他事件标志，使事件组的值为0x92



由应用程序编写器为事件组中的单个位分配意义。例如，应用程序编写器可以创建一个事件组，然后：

- 将事件组中的第0位定义为“已从网络接收到消息”。
- 将事件组中的第1位定义为“消息已准备好发送到网络上”。

- 将事件组中的第2位定义为“终止当前网络连接”。



关于EventBits_t数据类型的更多信息

事件组中的事件位数依赖于FreeRTOSConfig.h1中的configUSE_16_BIT_ TICKS编译时配置常量：

- 如果configUSE_16_BIT_TICKS为1，则每个事件组包含8个可用的事件位。
- 如果configUSE_16_BIT_TICKS为0，则每个事件组包含24个可用的事件位。

FreeRTOSConfig.h1：configUSE_16_BIT_TICKS配置用于保存RTOS滴答数的类型，因此似乎与事件组特性无关。它对EventBits_t类型的影响是FreeRTOS内部实现的结果，当FreeRTOS在一个能够比32位tvpes更有效地处理16位类型的架构上执行时，configUSE_16_BIT_TICKS应该只设置为1。



### 多任务访问

事件组是具有自身权限的对象，可以被任何知道其存在的任务或ISR访问。任意数量的任务可以在同一事件组中设置位，任意数量的任务可以从同一事件组中读取位。



### 一个使用事件组的实例

FreeRTOS+TCP TCP/IP栈的实现提供了一个实例，说明如何使用事件组同时简化设计，并最小化资源使用。



一个TCP套接字必须响应许多不同的事件。事件的例子包括接受事件、绑定事件、读取事件和关闭事件。套接字在任何给定时间所期望的事件取决于套接字的状态。例如，如果套接字已经创建，但还没有绑定到地址，那么它可以预期接收绑定事件，但不会预期接收读事件(如果没有地址，它就无法读取数据)。



一个FreeRTOS+TCP套接字的状态保存在一个叫做FreeRTOS_Socket_t的结构中。该结构包含一个事件组，该事件组为套接字必须处理的每个事件定义一个事件位。FreeRTOS+TCP API调用这个阻塞来等待一个事件或一组事件，只是在事件组上阻塞。



事件组还包含一个中止位，允许TCP连接被终止，不管套接字当时在等待哪个事件。



## 使用事件组的事件管理

### xEventGroupCreate() API函数

FreeRTOS V9.0.0还包括xEventGroupCreateStatic()函数，该函数在编译时分配静态创建事件组所需的内存:在使用事件组之前，必须显式地创建它。



使用EventGroupHandle_t类型的变量引用事件组。API函数xEventGroupCreate()用于创建事件组，并返回一个EventGroupHandle_t来引用它创建的事件组。

```groovy
EventGroupHandle_t xEventGroupCreate( void );
```

清单 132.  xEventGroupCreate() API函数原型



表 42. xEventGroupCreate() 的返回值

| 参数名 | 描述                                                         |
| :----- | ------------------------------------------------------------ |
| 返回值 | 如果返回NULL，则不能创建事件组，因为没有足够的堆内存供FreeRTOS分配事件组数据结构。第2章提供了堆内存管理的更多信息。如果返回非null值，则表示事件组创建成功。返回值应该存储为创建的事件组的句柄。 |



### xEventGroupSetBits() API函数

xEventGroupSetBits() API函数在事件组中设置一个或多个位，通常用于通知任务，由被设置的位表示的事件已经发生。

注意:永远不要在中断服务程序中调用xEventGroupSetBits()。应该使用中断安全版本xEventGroupSetBitsFromISR()来代替它。

```groovy
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,
								const EventBits_t uxBitsToSet );
```

清单133.  xEventGroupSetBits() API函数原型



表 43.  xEventGroupSetBits()参数和返回值

| 参数名      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| xEventGroup | 要在其中设置位的事件组的句柄。事件组句柄已经从用于创建事件组的xEventGroupCreate()调用中返回。 |
| uxBitsToSet | 一个位掩码，用于指定事件组中的事件位或事件位设置为1。事件组的值通过用uxBitsToSet传递的值按位或事件组的现有值来更新。例如，将uxBitsToSet设置为0x04(二进制0100)将导致事件组中的事件位3被设置(如果它还没有被设置)，而事件组中的所有其他事件位保持不变。 |
| 返回值      | 调用xEventGroupSetBits()返回时事件组的值。注意，返回的值不一定是uxBitsToSet指定的位，因为这些位可能已经被另一个任务再次清除。 |



### xEventGroupSetBitsFromlSR() API函数

xEventGroupSetBitsFromlSR()是xEventGroupSetBits()的中断安全版本。



给出信号量是一个确定性操作，因为预先知道给出信号量最多会导致一个任务离开阻塞状态。当在事件组中设置位时，不知道有多少任务将离开阻塞状态，因此在事件组中设置位不是确定性操作。



FreeRTOS设计和实现标准不允许不确定的操作在一个中断服务程序执行,或者当中断禁用这个原因，xEventGroupSetBitsFromlSR()不直接设置事件比特在中断服务程序,而是延缓RTOS守护进程的行动任务。

```groovy
BaseType_t xEventGroupSetBitsFromISR( EventGroupHandle_t xEventGroup,
 									  const EventBits_t uxBitsToSet,
									  BaseType_t *pxHigherPriorityTaskWoken );
```

清单 134.  xEventGroupSetBitsFromISR() API函数原型



表 44. xEventGroupSetBitsFromISR()参数和返回值

| 参数名                    | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| xEventGroup               | 要在其中设置位的事件组的句柄。事件组句柄已经从用于创建事件组的XEventGroupCreate()调用中返回。 |
| uxBitsToSet               | 一个位掩码，用于指定事件组中的事件位或事件位设置为1。事件组的值通过用uxBitsToSet传递的值按位或事件组的现有值来更新。例如，将uxBitsToSet设置为0x05(二进制0101)将导致事件组中的事件位3和事件位0被设置(如果它们还没有被设置)，而事件组中的所有其他事件位保持不变。 |
| pxHigherPriorityTaskWoken | xEventGroupSetBitsFromlSR()不直接在中断服务例程中设置事件位，而是通过在定时器命令队列上发送命令，将操作推迟给RTOS守护进程任务。如果守护任务处于阻塞状态以等待定时器命令队列上的数据可用，那么写入定时器命令队列将导致守护任务离开阻塞状态。如果守护进程任务的优先级高于当前正在执行的任务(被中断的任务)的优先级，那么，xEventGroupSetBitsFromlSR()将在内部将*pxHigherPriorityTaskWoken设置为pdTRUE。如果xEventGroupSetBitsFromlSR()将这个值设置为pdTRUE，那么应该在中断退出之前执行上下文切换。这将确保中断直接返回到守护任务，因为守护任务将是最高优先级的就绪状态任务。 |
| 返回值                    | 有两个可能的返回值：pdPASSpdPASS只在数据成功发送到定时器命令队列时返回。pdFALSE如果设置位命令不能写入定时器命令队列，因为队列已经满了，将返回pdFALSE， |

### xEventGroupWaitBits() API函数

xEventGroupWaitBits() APl函数允许任务读取事件组的值，并且可以选择在阻塞状态等待事件组中的一个或多个事件位被设置，如果这些事件位还没有设置。

```groovy
EventBits_t xEventGroupWaitBits( const EventGroupHandle_t xEventGroup,
 								 const EventBits_t uxBitsToWaitFor,
 								 const BaseType_t xClearOnExit,
 								 const BaseType_t xWaitForAllBits,
 								 TickType_t xTicksToWait );
```

清单135.  xEventGroupWaitBits() API函数原型



调度程序用来确定任务是否进入阻塞状态，以及任务何时离开阻塞状态的条件称为“解封条件”。解封条件由uxBitsToWaitFor和xWaitForAllBits参数值的组合指定：

- uxBitsToWaitFor指定要测试事件组中的哪个事件位
- xWaitForAllBits指定是使用位数OR测试，还是位数AND测试。



如果在调用xEventGroupWaitBits()时，任务的解锁条件得到满足，那么该任务将不会进入阻塞状态。



表45提供了导致任务进入阻塞状态或退出阻塞状态的条件示例。表45只显示了事件组和uxBitsToWaitFor值中最不重要的四个二进制位——这两个值的其他位被假定为零。



表45 uxBitsToWaitFor和xWaitForAllBits参数的影响

| Existing Event Group Value | uxBitsToWaitFor value | xWaitForAllBitsvalue | Resultant Behavior                                           |
| -------------------------- | --------------------- | -------------------- | ------------------------------------------------------------ |
| 0000                       | 0101                  | pdFALSE              | 由于在事件组中没有设置0位或2位，调用任务将进入阻塞状态，并且在事件组中设置0位或2位时将离开阻塞状态。 |
| 0100                       | 0101                  | pdTRUE               | 由于0位和2位没有同时设置在事件组中，调用任务将进入阻塞状态，当0位和2位同时设置在事件组中，调用任务将离开阻塞状态。 |
| 0100                       | 0110                  | pdFALSE              | 调用任务不会进入阻塞状态，因为xWaitForAllBits是pdFALSE，并且uxBitsToWaitFor指定的两个位中的一个已经在事件组中设置。 |
| 0100                       | 0110                  | pdTRUE               | 由于xWaitForAllBits为pdTRUE，并且uxBitsToWaitFor指定的两个位中只有一个已经在事件组中设置，因此调用任务将进入阻塞状态。当事件组中的第2位和第3位都被设置时，任务将离开阻塞状态。 |



调用任务使用uxBitsToWaitFor参数指定要测试的位，调用任务很可能需要在满足解封条件后将这些位清除为零。事件位可以使用xEventGroupClearBits() API函数来清除，但如果使用该函数手动清除事件位将导致应用程序代码中的竞争条件：

- 使用同一事件组的任务不止一个。
- 位由不同的任务或中断服务程序在事件组中设置。



提供了xClearOnExit参数以避免这些潜在的竞争条件。如果xClearOnExit设置为pdTRUE，那么对调用任务来说，事件位的测试和清除似乎是一个原子操作(不能被其他任务或中断中断)。



表46 xEventGroupWaitBits()参数和返回值

| 参数名          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xEventGroup     | 事件组的句柄，其中包含正在读取的事件位。事件组句柄已经从用于创建事件组的xEventGroupCreate()调用中返回。 |
| uxBitsToWaitFor | 指定事件组中要测试的事件位或事件位的位掩码。例如，如果调用任务想要等待事件位0和/或事件位2在事件组中被设置，那么将uxBitsToWaitFor设置为0x05(二进制0101)。更多的例子请参见表45。 |
| xClearOnExit    | 如果调用任务的解封条件已经被满足，并且xClearOnExit被设置为pdTRUE，那么在调用任务退出xEventGroupWaitBits() API函数之前，uxBitsToWaitFor指定的事件位将被清除回事件组中的0。如果xClearOnExit设置为pdFALSE，则事件组中的事件位的状态不会被xEventGroupWaitBits() API函数修改。 |
| xWaitForAllBits | uxBitsToWaitFor参数指定要在事件组中测试的事件位。xWaitForAllBits指定当uxBitsToWaitFor参数指定的一个或多个事件位被设置时，或者只有当uxBitsToWaitFor参数指定的所有事件位被设置时，调用任务才应该从阻塞状态中删除。如果xWaitForAllBits为pdFALSE,那么一个任务进入阻塞状态等待其开启条件满足时将阻塞状态的任何部分规定uxBitsToWaitFor成为集(或指定的超时xTicksToWait参数到期)。示例请参见表45。 |
| xTicksToWait    | 任务保持阻塞状态以等待其解除阻塞条件满足的最大时间。如果xTicksTolWait为零，或者在调用xEventGroupWaitBits()时满足解封条件，则xEventGroupWaitBits()将立即返回。块时间以滴答周期指定，因此它所代表的绝对时间依赖于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位指定的时间转换为以ticks为单位指定的时间。将xTicksToWait设置为portMAX_DELAY将导致任务无限期等待(不会超时)，前提是在FreeRTOSConfig.h中将INCLUDE_vTaskSuspend设置为1。 |
| Returned Value  | 如果xEventGroupWaitBits()返回是因为调用任务的解封条件被满足，那么返回值是调用任务的解封条件被满足时的事件组的值(在xClearOnExit为pdTRUE时自动清除任何位之前)。在这种情况下，返回值也将满足解封条件。如果xEventGroupWaitBits()返回是因为xTicksToWait参数指定的块时间过期，那么返回的值是块时间过期时事件组的值。在这种情况下，返回值将不满足解封条件。 |



### 例22. 实验事件组

这个例子演示了如何进行：

- 创建事件组。
- 从中断服务程序中设置事件组中的位。

- 从任务中设置事件组中的位。
- 阻塞事件组。



xEventGroupWaitBits() xWaitForAllBits参数的效果是通过首先执行xWaitForAllBits设置为pdFALSE的示例，然后执行xWaitForAllBits设置为pdTRUE的示例来演示的。



事件位0和事件位1由一个任务设置。事件位2是由中断服务程序设置的。例程设置。这三个位通过#define语句被赋予描述性的名字，如清单136所示。

```groovy
/*"事件组中事件位的定义。 */
#define mainFIRST_TASK_BIT n( 1UL << 0UL ) /* 事件位0，由任务设置。 */
#define mainSECOND_TASK_BIT ( 1UL << 1UL ) /* 事件位1，由任务设置。 */
#define mainISR_BIT         ( 1UL << 2UL ) /* 事件位2，由ISR设置。 */
```

清单136. 例22中使用的事件位定义



清单137显示了设置事件位0和事件位1的任务的实现。它位于一个循环中，反复设置一个位，然后再设置另一个位，每次调用xEventGroupSetBits()之间有200毫秒的延迟。在设置每个位之前打印一个字符串，以允许在控制台中看到执行序列。

```groovy
static void vEventBitSettingTask( void *pvParameters )
{
const TickType_t xDelay200ms = pdMS_TO_TICKS( 200UL ), xDontBlock = 0;
	for( ;; )
	{
 		/*在开始下一个循环之前稍作延迟。 */
 		vTaskDelay( xDelay200ms );
        
 		/*输出一条消息，表示任务即将设置事件位0，然后设置事件位0。*/
 		vPrintString( "Bit setting task -\t about to set bit 0.\r\n" );
 		xEventGroupSetBits( xEventGroup, mainFIRST_TASK_BIT );
        
 		/*在设置其他位之前稍作延迟。 */
 		vTaskDelay( xDelay200ms );
        
 		/*输出一个消息，说事件位1即将被任务设置，然后设置事件位1。*/
 		vPrintString( "Bit setting task -\t about to set bit 1.\r\n" );
 		xEventGroupSetBits( xEventGroup, mainSECOND_TASK_BIT );
 	} 
}
```

清单137.  例22中设置事件组中两个比特的任务



清单138显示了在事件组中设置第2位的中断服务例程的实现。同样，在设置位之前打印一个字符串，以允许在控制台中看到执行序列。但是，在这种情况下，因为控制台输出不应该直接在中断服务例程中执行，所以使用xTimerPendFunctionCallFromISR()在RTOS守护进程任务的上下文中执行输出。



与前面的示例一样，中断服务程序是由一个简单的周期性任务触发的，该任务强制软件中断。在本例中，中断每500毫秒产生一次。

```groovy
static uint32_t ulEventBitSettingISR( void )
{
/* 该字符串没有在中断服务程序中打印，而是被发送到RTOS守护进程任务中打印。因此，它被声明为static，以确保编译器不会在ISR的堆栈上分配字符串，因为当从守护进程任务打印字符串时，ISR的堆栈帧将不存在。*/
static const char *pcString = "Bit setting ISR -\t about to set bit 2.\r\n";
BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    /*输出一个消息说第2位即将被设置。消息不能从ISR打印，因此通过挂起函数调用在RTOS守护进程任务上下文	 中运行，将实际输出推迟到RTOS守护进程任务。*/
 	xTimerPendFunctionCallFromISR( vPrintStringFromDaemonTask, 
 								  ( void * ) pcString, 
 								  0, 
								  &xHigherPriorityTaskWoken );
    
    /* 在事件组中设置位2。 */
 	xEventGroupSetBitsFromISR( xEventGroup, mainISR_BIT, &xHigherPriorityTaskWoken );
    
    /* xTimerPendFunctionCallFromISR()和xEventGroupsetBitsFromISR()都写定时器命令队列，并使用		相同的xHigherPriorityraskwoken变量。如果写入定时器命令队列导致RTOS守护进程任务，并且RTOS守护进     程任务的优先级较高，则返回Blocked状态比当前正在执行的任务(此中断的任务)的优先级高然后				xhigherprioritytaskkoken将被设置为pdTRUE。
    
    xHigherPriorityTaskwoken用作portYIELD FROM ISR()的参数。如果xHigherPriorityraskwoken等于     pdTRUE，那么从ISR()调用portYIEID将请求一个上下文切换。如果xHigherPriorityraskwoken仍然是         pdFALSE，那么从ISR()调用portYIELD将没有效果。
    
    由windows端口使用的portYIELD_FROM_ISR()的实现包括一个返回语句，这就是为什么这个函数没有显式返	回值。*/
	portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
```

清单138.  例22中设置事件组中第2位的ISR



清单139显示了调用xEventGroupWaitBits()来阻塞事件组的任务的实现。任务为事件组中设置的每个位打印一个字符串。



xEventGroupWaitBits() xClearOnExit参数被设置为pdTRUE，因此导致调用xEventGroupWaitBits()返回的事件位或位将在xEventGroupWaitBits()返回之前被自动清除。

```groovy
static void vEventBitReadingTask( void *pvParameters )
{
EventBits_t xEventGroupValue;
const EventBits_t xBitsToWaitFor = ( mainFIRST_TASK_BIT  | 
 									 mainSECOND_TASK_BIT | 
 									 mainISR_BIT );
 	for( ;; )
 	{
        /* 阻塞以等待事件位在事件组中被设置。*/
 		xEventGroupValue = xEventGroupWaitBits( /* 要读取的事件组。 */
 												xEventGroup,
            
 												/* 位测试。 */
 												xBitsToWaitFor,
            
 												/*如果满足解封条件，则在退出时清除位。*/
 												pdTRUE,
            
 												/*不要等待所有的位。对于第二次执行，该参数被设置												 为pdTRUE。 */
 												pdFALSE,
            
 												/* 不要超时。 */
 												portMAX_DELAY );
 		/*为设置的每个位打印一条消息。 */
 		if( ( xEventGroupValue & mainFIRST_TASK_BIT ) != 0 )
 		{
 			vPrintString( "Bit reading task -\t Event bit 0 was set\r\n" );
 		}
 		if( ( xEventGroupValue & mainSECOND_TASK_BIT ) != 0 )
 		{
 			vPrintString( "Bit reading task -\t Event bit 1 was set\r\n" );
 		}
 		if( ( xEventGroupValue & mainISR_BIT ) != 0 )
 		{
 			vPrintString( "Bit reading task -\t Event bit 2 was set\r\n" );
 		}
 	} 
}
```

清单139. 例22中等待事件位被设置而阻塞的任务



main()函数在启动调度程序之前创建事件组和任务。有关它的实现，请参见清单140。从事件组中进行读操作的任务优先级高于向事件组中进行写操作的任务优先级，确保每次满足读任务的解阻塞条件时，读任务都会抢占写任务。

```groovy
int main(void)
{
    /* 在使用事件组之前，必须先创建事件组。 */
    xEventGroup = xEventGroupCreate();
    
    /* 创建在事件组中设置事件位的任务。 */
    xTaskCreate(vEventBitSettingTask, "Bit Setter", 1000, NULL, 1, NULL);
    
    /* 创建等待事件组中事件位设置的任务。*/
    xTaskCreate(vEventBitReadingTask, "Bit Reader", 1000, NULL, 2, NULL);
    
    /* 创建用于周期性产生软件中断的任务。 */
    xTaskCreate(vInterruptGenerator, "Int Gen", 1000, NULL, 3, NULL);
    
    /*安装软件中断的处理程序。完成此操作所需的语法取决于所使用的FreeRTOS端口。这里显示的语法只能在	FreeRros Windows端口中使用，在该端口中这种中断只能被模拟。 */
    vPortSetInterruptHandler(mainINTERRUPT_NUMBER, ulEventBitSettingISR);
    
    /* 启动调度程序，使创建的任务开始执行。 */
    vTaskStartScheduler();
    
    /* 启动调度程序，使已创建的任务开始执行。 */
    for (;;);
    return 0;
}
```

清单140. 创建例22中的事件组和任务



在执行示例22时，将xEventGroupWaitBits() xWaitForAllBits参数设置为pdFALSE，所产生的输出如图73所示。在图73中，可以看到，由于调用xEventGroupWaitBits()中的xWaitForAllBits参数被设置为pdFALSE，从事件组中读取的任务将离开阻塞状态，并在每次设置任何事件位时立即执行。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636101093971-7eb3bbab-0706-410b-a4db-9aac8e2f3dc6.png)

图73 示例22执行xWaitForAllBits设置为pdFALSE时产生的输出



在执行示例22时，将xEventGroupWaitBits() xWaitForAllBits参数设置为pdTRUE，所产生的输出如图74所示。在图74中可以看到，因为xWaitForAllBits参数被设置为pdTRUE，从事件组中读取的任务只有在设置了所有三个事件位之后才会离开状态阻塞。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636101269447-89193a38-41d7-4f19-be10-3e4ac2e23fb6.png)

图74 将xWaitForAlBits设置为pdTRUE执行示例22时产生的输出



## 使用事件组进行任务同步

有时，应用程序的设计需要两个或多个任务来彼此同步。例如，考虑这样一种设计:任务A接收一个事件，然后将该事件所需的一些处理委托给另外三个任务:任务B、任务C和任务D。如果任务A无法接收到另一个事件，直到任务B、C和D都完成了对前一个事件的处理，那么这四个任务就需要彼此同步。每个任务的同步点将在该任务完成其处理之后，并且不能继续进行，直到其他每个任务都完成了同样的处理。只有当所有四个任务都到达它们的同步点时，任务A才能接收到另一个事件。



需要这种类型的任务同步的一个不那么抽象的例子可以在一个FreeRTOS+TCP演示项目中找到。演示在两个任务之间共享一个TCP套接字;一个任务向套接字发送数据，另一个任务从同一个套接字1接收数据。在确定其他任务不会再次尝试访问该套接字之前，关闭TCP套接字对任何一个任务来说都是不安全的。如果两个任务中有一个希望关闭套接字，那么它必须通知另一个任务它的意图，然后等待另一个任务停止使用该套接字，然后再继续。清单140所示的伪代码演示了将数据发送到希望关闭套接字的任务的场景。



清单140所展示的情景是微不足道的，因为只有两个任务需要互相同步，但很容易看出，如果有其他任务在执行同步，该方案会变得更复杂，需要更多的任务加入同步，如果其他的任务在执行依赖于套接字的处理的话。处理依赖于套接字被打开。



套接字1 ：在编写本文的时候，这是在任务之间共享单个FreeRTOS+TCP套接字的唯一方法。

```groovy
void SocketTxTask(void *pvParameters)
{
xSocket_t xSocket;
uint32_t ulTxCount = 0UL;
    
    for (;;)
    {
       /*创建一个新的socket。这个任务将发送到这个套接字，而另一个任务将从这个套接字接收。*/
        xSocket = FreeRTOS_socket(...);
        
        /* 连接套接字。 */
        FreeRTOS_connect(xSocket, ...);

        /* 使用队列将套接字发送到接收数据的任务。 */
        xQueueSend(xSocketPassingQueue, &xSocket, portMAX_DELAY);
        
        /* 在关闭套接字之前向套接字发送1000条消息。*/
        for (ulTxCount = 0; ulTxCount < 1000; ulTxCount++)
        {
            if (FreeRTOS_send(xSocket, ...) < 0)
            {
                /* 意外错误-退出循环，之后套接字将被关闭。 */
                break;
            }
        }
        
        /* 让Rx任务知道Tx任务想要关闭套接字。 */
        TxTaskWantsToCloseSocket();
        
        /* 这是Tx任务的同步点。Tx任务在这里等待Rx任务到达它的同步点。Rx任务只有在不再使用套接字时才			会到达它的同步点，并且可以安全地关闭套接字。*/
        xEventGroupSync(...);
        
        /* 这两个任务都没有使用套接字。关闭连接，然后关闭套接字。*/
        FreeRTOS_shutdown(xSocket, ...);
        WaitForSocketToDisconnect();
        FreeRTOS_closesocket(xSocket);
    }
}
/*-----------------------------------------------------------*/

void SocketRxTask(void *pvParameters)
{
xSocket_t xSocket;
    
    for (;;)
    {
        /* 等待接收由Tx任务创建并连接的套接字。 */
        xQueueReceive(xSocketPassingQueue, &xSocket, portMAX_DELAY);
        
        /* 继续接收套接字，直到Tx任务想要关闭套接字。 */
        while (TxTaskWantsToCloseSocket() == pdFALSE)
        {
            /* 接收然后处理数据。 */
            FreeRTOS_recv(xSocket, ...);
            ProcessReceivedData();
        }
        
        /*这是Rx任务的同步点——它只在不再使用套接字时到达这里，因此Tx任务关闭套接字是安全的。*/
        xEventGroupSync(...);
    }
}
```

清单141.两个任务的伪代码，它们彼此同步以确保在套接字关闭之前，任何一个任务都不再使用共享的TCP套接字



事件组可用于创建同步点:

- 必须参与同步的每个任务在事件组中被分配一个唯一的事件位。
- 每个任务在到达同步点时设置自己的事件位。

- 设置了自己的事件位后，每个任务阻塞在事件组上，等待代表所有其他同步任务的事件位也被设置。



但是，xEventGroupSetBits()和xEventGroupWaitBits() API函数不能在此场景中使用。如果使用它们，那么位的设置(指示一个任务已经到达它的同步点)和位的测试(确定其他同步任务是否已经到达它们的同步点)将作为两个独立的操作执行。为了了解为什么这是一个问题，考虑一个场景，任务A任务B和任务C尝试使用事件组进行同步:

1. 任务A和任务B已经到达同步点，事件位已经在事件组中设置，处于阻塞状态，等待任务C的事件位也被设置。
2. 任务C到达同步点后，使用xEventGroupSetBits()设置其在事件组中的位。一旦任务C的位被设置，任务A和任务B就会离开阻塞状态，并清除所有三个事件位。

1. 任务C然后调用xEventGroupWaitBits()等待这三个事件比特成为集,但到那时,所有三个事件已经被清除,任务和任务B离开各自的同步点,所以同步失败了。



要成功地使用事件组创建同步点，事件位的设置和后续事件位的测试必须作为一个单独的不可中断操作执行。为此提供了xEventGroupSync() API函数。



### xEventGroupSync() API函数

提供xEventGroupSync()允许两个或多个任务使用事件组彼此同步。该功能允许任务在一个事件组中设置一个或多个事件位，然后等待在同一事件组中设置多个事件位的组合，作为一个单独的不可中断操作。



xEventGroupSync() uxBitsTolaitFor参数指定调用任务的解除阻塞条件。如果xEventGroupSync()返回是因为满足了不阻塞条件，uxBitsTolaitFor指定的事件位将在xEventGroupSync()返回之前被清除回零。

```groovy
EventBits_t xEventGroupSync(EventGroupHandle_t xEventGroup,
                            const EventBits_t uxBitsToSet,
                            const EventBits_t uxBitsToWaitFor,
                            TickType_t xTicksToWait);
```

清单142. xEventGroupSync() API函数原型



表47. xEventGroupSync()参数和返回值

| 参数名          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xEventGroup     | 要在其中设置事件位然后测试的事件组的句柄。事件组句柄已经从用于创建事件组的xEventGroupCreate()调用中返回。 |
| uxBitsToSet     | 一个位掩码，用于指定事件组中的事件位或事件位设置为1。事件组的值通过用uxBitsToSet传递的值按位或事件组的现有值来更新。例如，将uxBitsToSet设置为0x04(二进制0100)将导致事件3位被设置(如果它还没有被设置)，而事件组中的所有其他事件位保持不变。 |
| uxBitsToWaitFor | 指定事件组中要测试的事件位或事件位的位掩码。例如，如果调用任务想要等待事件位0，1和2在事件组中被设置，那么将uxBitsToWaitFor设置为0x07(二进制111). |
| xTicksToWait    | 任务保持阻塞状态以等待其解除阻塞条件满足的最大时间。如果xTicksToWait为零，或者在调用xEventGroupSync()时满足解除条件，则xEventGroupSync()将立即返回。块时间以滴答周期指定，因此它所代表的绝对时间依赖于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位的时间转换为以节拍为单位的时间。将xTicksToWait设置为portMAX_DELAY将导致任务无限期等待(不会超时)，前提是在FreeRTOSConfig.h中将INCLUDE_vTaskSuspend设置为1。 |
| Returned Value  | 如果xEventGroupSync()返回是因为调用任务的阻塞条件被满足，那么返回的值是调用任务的阻塞条件被满足时的事件组的值(在任何位被自动清除回零之前)。在这种情况下，返回值也将满足调用任务的阻塞条件。如果xEventGroupSync()返回是因为xTicksToWait参数指定的块时间过期，那么返回的值是块时间过期时事件组的值。在这种情况下，返回值将不满足调用任务的阻塞条件。 |



### 例23. 同步任务

例23使用xEventGroupSync()同步一个任务实现的三个实例。任务参数用于将任务调用xEventGroupSync()时设置的事件位传递给每个实例。



任务在调用xEventGroupsync()之前打印一条消息，在调用xEventGroupsync()返回之后再次打印一条消息。每条消息都包含一个时间戳。这允许在生成的输出中观察执行顺序。伪随机延迟用于防止所有任务同时到达同步点。



有关任务的实现，请参见清单143。

```groovy
static void vSyncingTask(void *pvParameters)
{
    const TickType_t xMaxDelay = pdMS_TO_TICKS(4000UL);
    const TickType_t xMinDelay = pdMS_TO_TICKS(200UL);
    TickType_t xDelayTime;
    EventBits_t uxThisTasksSyncBit;
    const EventBits_t uxAllSyncBits = (mainFIRST_TASK_BIT |
                                       mainSECOND_TASK_BIT |
                                       mainTHIRD_TASK_BIT);

    /*创建该任务的三个实例-每个任务在同步中使用不同的事件位。使用的事件位通过任务参数传递到每个任务实		例。将其存储在uxthisaskssyncbit变量中。*/
    uxThisTasksSyncBit = (EventBits_t)pvParameters;
    
    for (;;)
    {
        /*通过延迟一个伪随机时间来模拟这个任务花费一些时间来执行一个动作。这可以防止该任务的所有三个			实例同时到达同步点，因此可以更容易地观察示例的行为。*/
        xDelayTime = (rand() % xMaxDelay) + xMinDelay;
        vTaskDelay(xDelayTime);

        /*打印一条消息，显示这个任务已经到达它的同步点。pcTaskGetTaskName()是一个API函数，它返回在		 创建任务时分配给任务的名称。*/
        vPrintTwoStrings(pcTaskGetTaskName(NULL), "reached sync point");

        /*等待所有任务到达各自的同步点。*/
        xEventGroupSync(/* 用于同步的事件组。 */
                        xEventGroup,
            
                        /* 该任务设置的位，表示它已到达同步点。 */
                        uxThisTasksSyncBit,
            
                        /*需要等待的比特，每个参与同步的任务都有一个位*/
                        uxAllSyncBits,
            
                        /*无限等待所有三个任务到达vnchronization点。*/
                        portMAX_DELAY);

        /*打印一条消息，显示这个任务已经通过了它的同步点。由于使用了无限期延迟，所以只有在所有任务到			达各自的同步点后才会执行下面的行。*/
        vPrintTwoStrings(pcTaskGetTaskName(NULL), "exited sync point");
    }
}
```

清单143.  例23中使用的任务的实现



main()函数创建事件组，创建所有三个任务，然后启动调度程序。有关它的实现，请参见清单144。

```groovy
/* 事件组中的事件位的定义。 */
#define mainFIRST_TASK_BIT (1UL << 0UL) /* 事件位0，由第一个任务设置。 */
#define mainSECOND_TASK_BIT(1UL << 1UL) /* 事件位1，由第二个任务设置。 */
#define mainTHIRD_TASK_BIT (1UL << 2UL) /* 事件位2，由第三个任务设置。 */

/* 声明用于同步三个任务的事件组。 */
EventGroupHandle_t xEventGroup;

int main(void)
{
    /* 在使用事件组之前，必须先创建它。 */
    xEventGroup = xEventGroupCreate();
    
    /*创建任务的三个实例。每个任务都有一个不同的名称，稍后将打印出来，以直观地指示正在执行的任务。当任		务到达其同步点时要使用的事件位使用任务参数传递给任务。*/
    xTaskCreate(vSyncingTask, "Task 1", 1000, mainFIRST_TASK_BIT, 1, NULL);
    xTaskCreate(vSyncingTask, "Task 2", 1000, mainSECOND_TASK_BIT, 1, NULL);
    xTaskCreate(vSyncingTask, "Task 3", 1000, mainTHIRD_TASK_BIT, 1, NULL);
    
    /* 启动调度器，使创建的任务开始执行。 */
    vTaskStartScheduler();
    
    /* 像往常一样，永远不要达到下面这行。 */
    for (;;);
    return 0;
}
```

清单144. 例23中使用的main()函数



执行示例23时产生的输出如图75所示。可以看到，即使每个任务在不同的(伪随机)时间到达同步点，每个任务在同一时间退出同步点1(这是最后一个任务到达同步点的时间)。



同步点1：图75显示了在FreeRTOS Windows端口中运行的示例，它不提供真正的实时行为(特别是当使用Windows系统调用打印到控制台时)，因此会显示一些时间变化。



![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636106241870-6e67e703-5286-4d20-ad9b-6e847b751628.png)

图75执行示例23时产生的输出



# **任务通知**



## 引言与范围

我们已经看到，使用FreeRTOS的应用程序被构造成一组独立的任务，而且这些自主的任务很可能必须相互通信，这样它们才能共同提供有用的系统功能。



### 通过中介对象进行通信

这本书已经描述了任务之间相互通信的各种方式。到目前为止，所描述的方法都需要创建通信对象。通信对象的示例包括队列、事件组和各种不同类型的信号量。



当使用通信对象时，事件和数据不会直接发送给接收任务或接收ISR，而是发送给通信对象。同样，任务和ISR从通信对象接收事件和数据，而不是直接从发送事件或数据的任务或ISR接收事件和数据。如图76所示。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636334710668-2719753a-8da1-41d1-b629-77bf4ee588a9.png)

图76 用于将事件从一个任务发送到另一个任务的通信对象



### 任务通知-直接与任务通信

任务通知允许任务与其他任务交互，并与ISR同步，不需要单独的通信对象。通过使用任务通知，任务或ISR可以直接向接收任务发送事件。如图77所示。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636335089505-b72e3997-de68-4788-8626-f2939872cc4f.png)

图77 用于将事件从一个任务直接发送到另一个任务的任务通知



任务通知功能是可选的。要包含任务通知功能，请在FreeRTOSConfig.h 中将configUSE_TASK_NOTIFICATIONS 设置为1。



当configUSE_TASK_NOTIFICATIONS被设置为1时，每个任务有一个 "通知状态"。可以是 "待定 "或 "未定"，还有一个 "通知值"，它是一个32位无符号整数。无符号整数。当一个任务收到一个通知时，它的通知状态被设置为待定。当一个任务读取其通知值时，其通知状态被设置为非待定。



任务可以在阻塞状态下等待其通知状态变为挂起（可选超时）。



### 范围

本章旨在让读者更好地理解：

- 任务的通知状态和通知值。
- 何时以及如何使用任务通知来代替通信对象(如信号量)。

- 使用任务通知代替通信对象的优点。



## 任务通知；优势和限制

### 任务通知的性能优势

使用任务通知向任务发送事件或数据要比使用队列、信号量或事件组执行等价操作快得多。



### 任务通知的RAM占用优势

同样，使用任务通知向任务发送事件或数据所需的RAM要比使用队列、信号量或事件组执行等价操作所需的RAM少得多。这是因为必须先创建每个通信对象(队列、信号量或事件组)，然后才能使用它，而启用任务通知功能的开销固定为每个任务8个字节的RAM。



### 任务通知的局限性

任务通知比通信对象更快，使用的RAM更少，但任务通知不能在所有场景中使用。介绍不支持任务通知的场景：

- 向ISR发送事件或数据

通信对象可用于将事件和数据从ISR发送到任务，或从任务发送到ISR。

任务通知可以用于从ISR向任务发送事件和数据，但不能用于从任务向ISR发送事件或数据。



- 启用多个接收任务

通信对象可以被任何知道其句柄(可能是队列句柄、信号量句柄或事件组句柄)的任务或ISR访问。任意数量的任务和isr都可以处理发送到任何给定通信对象的事件或数据。

任务通知直接发送到接收任务，因此只能由将通知发送到的任务。然而，这在实际情况中很少是一个限制因为，虽然有多个任务和isr向同一个通信对象发送消息是很常见的，但有多个任务和isr从同一个对象接收消息是很少见的沟通对象。



- 缓冲多个数据项

队列是一次可以保存多个数据项的通信对象。已发送到队列但尚未从队列接收的数据将在队列对象中进行缓冲。

任务通知通过更新接收任务的通知值向任务发送数据。任务的通知值一次只能保存一个值。



- 向多个任务广播

事件组是一种通信方式，可用于一次向多个任务发送事件。

任务通知直接发送给接收任务，只能由接收任务处理。



- 在阻塞状态下等待发送完成

如果沟通对象是暂时的状态,这意味着没有更多的数据或事件可以写(例如,当队列满是没有更多的数据可以被发送到队列),然后试图写入任务对象可以选择进入阻塞状态等待写操作完成。

如果任务尝试向已挂起通知的任务发送任务通知，则发送任务不可能在“阻塞”状态等待接收任务重置其通知状态。正如我们将看到的，在使用任务通知的实际情况中，这很少是一个限制。



## 使用任务通知

### 任务通知API选项

任务通知是一个非常强大的特性，经常可以用来代替二进制信号量、计数信号量、事件组，有时甚至是队列。这种广泛的使用场景可以通过使用xTaskNotify() API函数发送任务通知和使用xTaskNotifyWait() API函数接收任务通知来实现。



但是，在大多数情况下，不需要xTaskNotify()和xTaskNotifyWait() API函数提供的全部灵活性，简单的函数就足够了。因此，提供了xTaskNotifyGive() API函数作为xTaskNotify的一个更简单但更不灵活的替代方案，而ulTaskNotifyTake() API函数作为xTaskNotifyWait()的一个更简单但更不灵活的替代方案。



### xTaskNotifyGive() API函数

xTaskNotifyGive()直接向任务发送通知，并增加(向)接收任务的通知值。调用xTaskNotifyGive()将把接收任务的通知状态设置为挂起(如果它还没有挂起)。



提供了xTaskNotifyGive()1的API函数，允许任务通知作为二进制或计数信号量的轻量级和更快的替代。

```groovy
BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify );
```

清单 145. xTaskNotifyGive() API函数原型



xTaskNotifyGive()1实际上是作为宏实现的，而不是一个函数。为了简单起见，本书将它称为一个函数。



表48. xTaskNotifyGive()参数和返回值

| 参数名称/返回的值 | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| xTaskToNotify     | 被发送通知的任务的句柄——有关获取任务句柄的信息，请参阅xTaskCreate() API函数的pxCreatedTask参数。 |
| Returned value    | xTaskNotifyGive()是一个调用xTaskNotify()的宏。宏传递给xTaskNotify()的参数被设置为pdPASS是唯一可能的返回值。xTaskNotify()将在本书后面进行描述。 |



### vTaskNotifyGiveFromISR() API函数

vTaskNotifyGiveFromlSR()是xTaskNotifyGive()的一个版本，可以在中断服务例程中使用。

```groovy
void vTaskNotifyGiveFromISR( TaskHandle_t xTaskToNotify, 
 							 BaseType_t *pxHigherPriorityTaskWoken );
```

清单146. vTaskNotifyGiveFromISR() API函数原型



表49. vTaskNotifyGiveFromISR()参数和返回值

| 参数名称/返回的值         | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| xTaskToNotify             | 被发送通知的任务的句柄——有关获取任务句柄的信息，请参阅xTaskCreate() API函数的pxCreatedTask参数。 |
| pxHigherPriorityTaskWoken | 如果正在发送通知的任务处于阻塞状态等待接收通知，则发送通知将导致任务离开阻塞状态。如果调用vTaskNotifyGiveFromISR()导致任务离开阻塞状态，并且未阻塞的任务的优先级高于当前正在执行的任务(被中断的任务)的优先级，那么，vTaskNotifyGiveFromISR()将在内部将*pxhigherpriorityTaskWoken设置为pdTRUE。如果vTaskNotifyGiveFromlSR()将这个值设置为pdTRUE，那么应该在中断退出之前执行上下文切换。这将确保中断直接返回到最高优先级的就绪状态任务。和所有中断安全API函数一样，pxHigherPriorityTaskWoken参数在使用之前必须设置为pdFALSE。 |



### ulTaskNotifyTake() API函数

ulTaskNotifyTake()允许任务在阻塞状态中等待它的通知值大于0，并且在返回之前减少(减去1)或清除任务的通知值。



提供了ulTaskNotifyTake() API函数，以允许任务通知作为二进制或计数信号量的轻量级和更快的替代方法。

```groovy
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, TickType_t xTicksToWait );
```

清单147.  ulTaskNotifyTake() API函数原型



表50. ulTaskNotifyTake()参数和返回值

| 参数名称/返回值   | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| xClearCountOnExit | 如果xClearCountOnExit设置为pdTRUE，那么在调用ulTaskNotifyTake()返回之前，调用任务的通知值将被清除为零。如果xClearCountOnExit设置为pdFALSE，并且调用任务的通知值大于零，那么调用任务的通知值将在调用ulTaskNotifyTake()返回之前递减。 |
| xTicksToWait      | 调用任务在等待其通知值大于零时应保持在状态阻塞的最大时间量。块时间以滴答周期指定，因此它所代表的绝对时间依赖于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位指定的时间转换为以节拍为单位指定的时间。设置xTicksToWait为portMAX_DELAY将导致任务无限期等待(不会超时)在FreeRTOSConfig.h中将INCLUDE_vTaskSuspend设置为1。 |
| 返回值            | 返回值是调用任务的通知值，在它被清除为零或减少之前，由xClearCountOnExit参数的值指定。如果一块指定时间(xTicksToWait不是零),返回值不为零,那么它有可能调用任务是放置进入阻塞状态,等待其通知值大于零,和它的值更新阻塞时间到期前通知。如果指定了块时间(xTicksToWait不为零)，并且返回值为零，然后调用任务被置于阻塞状态，等待其通知值大于零，但指定的阻塞时间在此之前已过期。 |



### 例24. 方法1使用任务通知来代替信号量

例16使用了一个二进制信号量来解除中断服务程序中的任务阻塞——有效地将任务与中断同步。这个示例复制了示例16的功能，但是使用了一个直接到任务通知来代替二进制信号量。



清单148显示了与中断同步的任务的实现。在示例16中使用的对xSemaphoreTake()的调用已经被对ulTaskNotifyTake()的调用所取代。



ulTaskNotifyTake() xClearCountOnExit参数被设置为pdTRUE，这会导致接收任务的通知值在ulTaskNotifyTake()返回之前被清除为零。因此，有必要处理在每次调用ulTaskNotifyTake()之间已经可用的所有事件。在示例16中，因为使用了一个二进制信号量，所以挂起事件的数量必须从硬件中确定，这并不总是实际的。在示例24中，从ulTaskNotifyTake()返回挂起事件的数量。



在调用ulTaskNotifyTake之间发生的中断事件被锁定在任务的通知值中，如果调用的任务已经有挂起的通知，那么对ulTaskNotifyTake()的调用将立即返回。

```groovy
/* 周期性任务生成软件中断的速率。 */
const TickType_t xInterruptFrequency = pdMS_TO_TICKS(500UL);

static void vHandlerTask(void *pvParameters)
{
/* xMaxExpectedBlockTime设置为略长于事件之间的最大预期时间。 */
const TickType_t xMaxExpectedBlockTime = xInterruptFrequency + pdMS_TO_TICKS(10);
uint32_t ulEventsToProcess;
    
    /* 对于大多数任务来说，这个任务是在一个无限循环中实现的。 */
    for (;;)
    {
        /* 等待接收从中断服务程序直接发送到此任务的通知。 */
        ulEventsToProcess = ulTaskNotifyTake(pdTRUE, xMaxExpectedBlockTime);
        if (ulEventsToProcess != 0)
        {
            /* 要到达这里，必须至少发生一件事。在此循环，直到处理完所有挂起事件(在本例中，只需为每个			  事件打印一条消息)。*/
            while (ulEventsToProcess > 0)
            {
                vPrintString("Handler task - Processing event.\r\n");
                ulEventsToProcess--;
            }
        }
        else
        {
            /* 如果到达函数的这一部分，则中断没有在预期的时间内到达，并且(在真实的应用程序中)可能需			  要执行一些错误恢复操作。*/
        }
    }
}
```

清单148。中断处理所涉及的任务的实现 的实现（与中断同步的任务）。



用于生成软件中断的周期任务在中断生成之前打印一条消息，并在中断生成之后再次打印一条消息。这允许在生成的输出中观察执行顺序。



清单149显示了中断处理程序。除了直接向延迟中断处理的任务发送通知外，这几乎不做什么。

```groovy
static uint32_t ulExampleInterruptHandler(void)
{
BaseType_t xHigherPriorityTaskWoken;
    
    /* xHigherPriorityTaskWoken参数必须初始化为pdFALSE，因为如果需要进行上下文切换，它将在中断安全	API函数中被设置为pdTRUE。*/
    xHigherPriorityTaskWoken = pdFALSE;
    
    /* 直接向延迟中断处理的任务发送通知。 */
    vTaskNotifyGiveFromISR(/* 正在发送通知的任务的句柄。句柄是在创建任务时保存的。*/
                           xHandlerTask,
        
                           /* xHigherPriorityTaskWoken以通常的方式使用。 */
                           &xHigherPriorityTaskWoken);

    /*将xHigherPriorityTaskWoken值传递给portYIELD_FROM_ISR()。如果在vTaskNotifyGiveFromISR()		中xHigherPriorityTaskWoken被设置为pdTRUE，那么调用portYIELD_FROM_ISR()将请求一个上下文切换。	   如果xHigherPriorityTaskWoken仍然是pdFALSE，那么调用portYIELD_FROM_ISR()将没有效果。Windows	 端口使用的portYIELD_FROM_ISR()的实现包含一个return语句，这就是为什么该函数没有显式返回值。*/
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单149. 例24中使用的中断服务例程的实现



执行示例24时产生的输出如图78所示。正如预期的那样，它与执行示例16时产生的结果相同。vHandlerTask()在生成中断后立即进入运行状态，因此任务的输出将分割周期性任务产生的输出。图79提供了进一步的解释。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636340508634-f4ae301a-d216-4a7a-ac50-4045d007b87a.png)

图78  执行例16时产生的输出



![img](https://cdn.nlark.com/yuque/0/2021/png/23129846/1636340626206-22890559-016b-4f87-8268-e7af8d15f79f.png)

图79  执行例24时的执行顺序



### 25例. 方法2使用任务通知来代替信号量

在例24中，ulTaskNotifyTake() xClearOnExit参数被设置为pdTRUE。示例25稍微修改了示例24，以演示将ulTaskNotifyTake() xClearOnExit参数改为pdFALSE时的行为。



当xClearOnExit为pdFALSE时，调用ulTaskNotifyTake()只会减少(减少1)调用任务的通知值，而不是将其清除为零。因此，通知计数是已发生事件的数量与已处理事件的数量之间的差值。这允许vHandlerTask()的结构以两种方式简化：

1. 等待处理的事件数量保存在通知值中，因此不需要保存在局部变量中。
2. 每次调用ulTaskNotifyTake()之间只需要处理一个事件。



示例25中使用的vHandlerTask()的实现如清单150所示。

```groovy
static void vHandlerTask(void *pvParameters)
{
/*xMaxExpectedBlockTime设置为比事件之间的最大预期时间稍长一点。*/
const TickType_t xMaxExpectedBlockTime = xInterruptFrequency + pdMS_TO_TICKS(10);
    
    /* 对于大多数任务来说，这个任务是在一个无限循环中实现的。*/
    for (;;)
    {
        /*等待接收从中断服务程序直接发送到此任务的通知。xClearCountOnExit参数现在是pdFALSE，因此任		  务的通知值将由ulTaskNotifyTake()递减，而不是清除为零。*/
        if (ulTaskNotifyTake(pdFALSE, xMaxExpectedBlockTime) != 0)
        {
            /*要到达这里，必须有事件发生。处理事件(在本例中只打印一条消息)。*/
            vPrintString("Handler task - Processing event.\r\n");
        }
        else
        {
            /*如果到达了函数的这一部分，那么在预期的时间内中断没有到达，并且(在真实的应用程序中)可能			  需要执行一些错误恢复操作。*/
        }
    }
}
```

清单150. 例25中中断处理延迟到的任务的实现(与中断同步的任务)。



出于演示的目的，中断服务程序也被修改为每个中断发送多个任务通知，这样做，模拟在高频率下发生的多个中断。示例25中使用的中断服务例程的实现如清单151所示。

```groovy
static uint32_t ulExampleInterruptHandler(void)
{
BaseType_t xHigherPriorityTaskWoken;
    
    xHigherPriorityTaskWoken = pdFALSE;
    
    /* 多次向处理程序任务发送通知。第一个“给出”将解除对任务的阻止，下面的“给出”将演示接收任务的通知值	正在用于计数（锁定）事件-允许任务依次处理每个事件。*/
    vTaskNotifyGiveFromISR(xHandlerTask, &xHigherPriorityTaskWoken);
    vTaskNotifyGiveFromISR(xHandlerTask, &xHigherPriorityTaskWoken);
    vTaskNotifyGiveFromISR(xHandlerTask, &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单151. 例25中使用的中断服务例程的实现



执行示例25时产生的输出如图80所示。可以看到，vHandlerTask()在每次生成中断时处理所有这三个事件。

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/23129846/1636341690974-60c2b176-1dc3-48b8-90f4-aed271d6d082.jpeg)

图80  执行例25时产生的输出



### xTaskNotify()和xTaskNotifyFromlSR() API函数

xTaskNotify()是xTaskNotifyGive()的一个更强大的版本，可以用以下任何一种方式来更新接收任务的通知值：

- 增加接收任务的通知值（向其中添加一个），在这种情况下，xTaskNotify()相当于xTaskNotifyGive()。
- 在接收任务的通知值中设置一个或多个位。这允许将任务的通知值用作事件组的更轻的权重和更快的替代。

- 向接收任务的通知值中写入一个全新的数字，但只有当接收任务在上次更新后已经读取了它的通知值时才可以。这允许任务的通知值提供类似于长度为1的队列所提供的功能。
- 向接收任务的通知值中写入一个全新的数字，即使接收任务自上次更新后还没有读到它的通知值。这允许任务的通知值提供与xQueueOverwrite() API函数类似的功能。结果行为有时被称为“邮箱”。



xTaskNotify()比xTaskNotifyGive()更灵活和强大，而且由于这种额外的灵活性和强大功能，它使用起来也有点复杂。



xTaskNotifyFromISR()是xTaskNotify()的一个版本，可以在中断服务例程中使用，因此有一个额外的pxHigherPriorityTaskWoken参数。



调用xTaskNotify()将始终将接收任务的通知状态设置为挂起(如果它还没有挂起)。

```groovy
BaseType_t xTaskNotify(TaskHandle_t xTaskToNotify,
                       uint32_t ulValue,
                       eNotifyAction eAction);

BaseType_t xTaskNotifyFromISR(TaskHandle_t xTaskToNotify,
                              uint32_t ulValue,
                              eNotifyAction eAction,
                              BaseType_t *pxHigherPriorityTaskWoken);
```

清单152.  xTaskNotify()和xTaskNotifyFromISR() API函数的原型



表 51.  xTaskNotify()参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xTaskToNotify   | 被发送通知的任务的句柄——有关获取任务句柄的信息，请参阅xTaskCreate() API函数的pxCreatedTask参数。 |
| ulValue         | ulValue的使用方式取决于eNotifyAction值。见表52。             |
| eNotifyAction   | 指定如何更新接收任务的通知值的枚举类型返回值                 |
| 返回值          | xTaskNotify()将返回pdPASS，除了表52中提到的一种情况。        |



表 52. 有效的xTaskNotify() eNotifyAction参数值，以及它们对接收任务的通知值的结果影响

| 参数名称/返回值           | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| eNoAction                 | 接收任务的通知状态被设置为挂起，而它的通知值不被更新。没有使用xTaskNotify() uIValue参数。eNoAction操作允许任务通知作为二进制信号量的更快和更轻的替代。 |
| eSetBits                  | 接收任务的通知值是按位或通过xTaskNotify()的ulValue参数传递的值。例如，ulValue设置为0x01，则接收任务的通知值中设置0位。另一个例子，如果ulValue是0x06(二进制0110)，那么第1位和第2位将被设置在接收任务的通知值中。eSetBits动作允许任务通知被用作事件组的更快和更轻的替代。 |
| eIncrement                | 接收任务的通知值增加。没有使用xTaskNotify() uIValue参数。elncrement操作允许任务通知作为二进制或计数信号量的更快和更轻的替代方法，它相当于更简单的xTaskNotifyGive() API函数。 |
| eSetValueWithoutOverwrite | 如果接收任务在调用xTaskNotify()之前有一个通知挂起，则不会采取任何操作，xTaskNotify()将返回pdFAlL。如果在调用xTaskNotify()之前接收任务没有挂起通知，那么接收任务的通知值将被设置为在xTaskNotify() uNValue参数中传递的值。 |
| eSetValueWithOverwrite    | 接收任务的通知值被设置为在xTaskNotify() ulValue参数中传递的值，不管接收任务在调用xTaskNotify()之前是否有一个通知挂起。 |



### xTaskNotifyWait() API函数

xTaskNotifyWait()是ulTaskNotifyTake()的一个更强大的版本。它允许任务等待(带有可选超时)，以便调用任务的通知状态变为挂起状态(如果它还没有挂起)。xTaskNotifyWait()提供了在进入函数和退出函数时要在调用任务的通知值中清除的位的选项。

```groovy
BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry,
 							uint32_t ulBitsToClearOnExit,
 							uint32_t *pulNotificationValue,
 							TickType_t xTicksToWait );
```

清单153.  xTaskNotifyWait() API函数原型



表 53.  xTaskNotifyWait()参数和返回值

| 参数名称/返回值      | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| ulBitsToClearOnEntry | 如果调用任务在调用xTaskNotifyWait()之前没有一个通知挂起，那么ulBitsToClearOnEntry中设置的任何位都将在进入函数时在任务的通知值中被清除。例如，如果ulBitsToClearOnEntry为0x01，则任务通知值的位0将被清除。另一个例子是，将ulBitsToClearOnEntry设置为0xFFFFFF（ULONG_MAX）将清除任务通知值中的所有位，有效地将该值清除为0。 |
| ulBitsToClearOnExit  | 如果调用任务退出xTaskNotifyWait()因为它收到一个通知,或因为它已经通知等待当xTaskNotifyWait()被称为,那么任何部分设置在ulBitsToClearOnExit将被清除在任务前的通知价值任务退出xTaskNotifyWait()函数。任务的通知值保存在*pulNotificationValue中后，这些位将被清除（请参见下面的pulNotificationValue说明）。例如，如果ulBitsToClearOnExit为0x03，那么在函数退出之前，任务通知值的0位和1位将被清除。将ulBitsToClearOnExit设置为0xfff (ULONG_MAX)将清除任务通知值中的所有位，有效地将该值清除为0。 |
| pulNotificationValue | 用于传递任务的通知值。复制到*pulNotificationValue的值是任务的通知值，与由于ulBitsToClearOnExit设置而清除任何位之前的值相同。pulNotificationValue是一个可选参数，如果不需要，可以设置为NULL。 |
| xTicksToWait         | 调用任务应保持在阻止状态以等待其通知状态变为挂起的最长时间。块时间以滴答周期为单位指定，因此它表示的绝对时间取决于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位指定的时间转换为以TICKS为单位指定的时间。如果FreeRTOSConfig.h中将INCLUDE_vTaskSuspend设置为1，则将xTicksToWait设置为portMAX_DELAY将导致任务无限期等待（不超时）。 |
| 返回值               | 有两个可能的返回值：pdTRUE 这表明返回xTaskNotifyWait()是因为收到了通知，或者是因为调用xTaskNotifyWait()时，调用任务已经有一个通知挂起。如果指定了阻止时间（xTicksToWait不是零），则调用任务可能被置于阻止状态，以等待其通知状态变为挂起，但其通知状态在阻止时间到期之前被设置为挂起。pdFALSE这表明xTaskNotifyWait()在调用任务没有收到任务通知的情况下返回。如果xTicksToWait不是零，则调用任务将一直处于阻止状态，以等待其通知状态变为挂起，但指定的阻止时间在此之前已过期。 |



### 外围设备驱动程序中使用的任务通知：UART示例

外设驱动程序库提供了在硬件接口上执行常用操作的函数。通常提供此类库的外设示例包括通用异步接收器和发射器(UARTs)、串行外设接口(SPI)端口、模拟到数字转换器(adc)和以太网端口。这些库通常提供的函数示例包括初始化外设、向外设发送数据和从外设接收数据的函数。



外围设备上的一些操作需要较长的时间才能完成。这类操作的例子包括高精度ADC转换，以及在UART上传输大数据包。在这些情况下，驱动程序库函数可以实现轮询反复读取外设的状态寄存器，以确定操作何时完成。然而，以这种方式轮询几乎总是浪费时间，因为它利用了处理器100%的时间，而没有执行有效的处理。在多任务系统中，这种浪费尤其昂贵，在该系统中，轮询外围设备的任务可能会阻止执行具有生产性处理的低优先级任务。



为了避免浪费处理时间的可能性，高效的RTOS感知设备驱动程序应该是中断驱动的，并为启动长时间操作的任务提供在阻塞状态下等待操作完成的选项。这样，低优先级任务可以在执行长时间操作的任务处于阻塞状态时执行，并且没有任务会使用处理时间，除非它们能够有效地使用处理时间。



RTOS驱动程序库通常使用二进制信号量将任务置于阻塞状态。清单154中所示的伪代码演示了该技术，它提供了在UART端口上传输数据的支持RTOS的库函数的概要。在清单154中：

- xUART是描述UART外围设备并保存状态信息的结构。该结构的xTxSemaphore成员是semaphhorehandle_t类型的变量。它假设信号量已经被创建。
- xUART_Send()函数不包含任何互斥逻辑。如果有多个任务要使用xUART_Send()函数，那么应用程序编写人员必须在应用程序本身中管理互斥。例如，任务可能需要在调用xUART_Send()之前获取互斥锁。

- xSemaphoreTake() API函数用于在启动UART传输之后将调用任务置于阻塞状态。
- xSemaphoreGiveFromISR() API函数用于在传输完成后将任务从阻塞状态中删除，此时UART外设的传输端中断服务例程执行。

```groovy
/*驱动程序库函数，用于向UART发送数据。*/
BaseType_t xUART_Send(xUART *pxUARTInstance, uint8_t *pucDataSource, size_t uxLength)
{
BaseType_t xReturn;
    
    /*通过尝试在没有超时的情况下获取信号量，确保UART的传输信号量不可用。*/
    xSemaphoreTake(pxUARTInstance->xTxSemaphore, 0);
    
    /* Start the transmission. */
    UART_low_level_send(pxUARTInstance, pucDataSource, uxLength);

    /*阻塞信号灯以等待传输完成。如果获得信号量，那么xReturn将设置为pdPASS。如果信号量take操作超时，	 那么xReturn将设置为pdFAIL。注意，如果中断发生在被调用的UART_low_level_send()和被调用的			xSemaphoreTake()之间，那么事件将被锁存在二进制信号量中，对xSemaphoreTake()的调用将立即返回。*/
    xReturn = xSemaphoreTake(pxUARTInstance->xTxSemaphore, pxUARTInstance->xTxTimeout);

    return xReturn;
}
/*-----------------------------------------------------------*/

/*UART传输结束中断的服务例程，在最后一个字节发送到UART后执行。 */
void xUART_TransmitEndISR(xUART *pxUARTInstance)
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    /* 清除中断。 */
    UART_low_level_interrupt_clear(pxUARTInstance);

    /* 发送Tx信号量以指示传输结束。如果任务在等待信号量时被阻塞，则该任务将从阻塞状态中删除。*/
    xSemaphoreGiveFromISR(pxUARTInstance->xTxSemaphore, &xHigherPriorityTaskWoken);
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单154。演示如何在驱动程序库传输函数中使用二进制信号量的伪代码



清单154中演示的技术是完全可行的，确实是常见的做法，但它有一些缺点：

- 库使用多个信号量，这增加了内存占用。
- 信号量只有在创建之后才能使用，所以使用信号量的库只有在显式初始化之后才能使用。

- 信号量是适用于广泛用例的通用对象；它们包括允许任意数量的任务在阻塞状态下等待信号量变为可用的逻辑，以及在信号量变为可用时选择（以确定性方式）从阻塞状态中移除哪个任务的逻辑。执行该逻辑需要有限的时间，在清单154所示的场景中，处理开销是不必要的，在该场景中，在任何给定时间都不能有多个任务等待信号量。



清单155演示了如何使用任务通知代替二进制信号量来避免这些缺点。



注意:如果库使用任务通知，那么库的文档必须清楚地说明调用库函数可以更改调用任务的通知状态和通知值。



在清单155中：

- xUART结构的xTxSemaphore成员已被xTaskToNotify成员替换。xTaskToNotify是TaskHandle_t类型的变量，用于保存等待UART操作完成的任务的句柄。
- xTaskGetCurrentTaskHandle() FreeRTOS API函数用于获取运行状态任务的句柄。

- 库不创建任何FreeRTOS对象，因此不会引起内存开销，也不需要显式初始化。
- 任务通知将直接发送到等待UART操作完成的任务，因此不会执行不必要的逻辑。



xTaskToNotify结构的xTaskToNotify成员可以从任务和中断服务程序中访问，需要考虑处理器将如何更新它的值：

- 如果xTaskToNotify通过一个内存写操作更新，那么它可以在临界区外更新，如清单155所示。如果xTaskToNotify是一个32位变量(TaskHandle_t是一个32位类型)，并且运行FreeRTOS的处理器是一个32位处理器，就会出现这种情况。
- 如果更新xTaskToNotify需要一个以上的内存写入操作，则xTaskToNotify只能从关键部分中更新，否则中断服务例程可能会在xTaskToNotify处于不一致状态时访问xTaskToNotify。如果xTaskToNotify是32位变量，并且FreeRTOS运行的处理器是16位处理器，则会出现这种情况，因为它需要两个16位内存写入操作来更新所有32位。



在FreeRTOS实现内部，TaskHandle_t是一个指针，所以sizeof(TaskHandle_t)总是等于sizeof(void *)。

```groovy
/*驱动程序库函数，用于向UART发送数据。*/
BaseType_t xUART_Send(xUART *pxUARTInstance, uint8_t *pucDataSource, size_t uxLength)
{
BaseType_t xReturn;
    
    /*保存调用此函数的任务的句柄。这本书的正文包含了关于以下行是否需要受关键部分保护的注释。*/
    pxUARTInstance->xTaskToNotify = xTaskGetCurrentTaskHandle();

    /*通过调用ulTaskNotifyTake()，将xClearCountOnExit参数设置为pdTRUE，并将阻塞时间设置为0（不阻	塞），确保调用任务尚未挂起通知。*/
    ulTaskNotifyTake(pdTRUE, 0);
    
    /*启动变速器。*/
    UART_low_level_send(pxUARTInstance, pucDataSource, uxLength);
    
    /*阻塞，直到通知传输完成。如果收到通知，则xReturn将设置为1，因为ISR已将此任务的通知值增加为1		（pdTRUE）。如果操作超时，则xReturn将为0（pdFALSE），因为此任务的通知值在上面清除为0后不会更		改。请注意，如果ISR在对UART_low_level_send()的调用和对ulTaskNotifyTake()的调用之间执行，则事		件将锁定在任务的通知值中，对ulTaskNotifyTake()的调用将立即返回。*/
    xReturn = (BaseType_t)ulTaskNotifyTake(pdTRUE, pxUARTInstance->xTxTimeout);
    
    return xReturn;
}
/*-----------------------------------------------------------*/

/*最后一个字节发送到UART后执行的ISR。*/
void xUART_TransmitEndISR(xUART *pxUARTInstance)
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    /*除非有任务等待通知，否则不应执行此函数。使用断言测试此条件。严格来说，此步骤不是必需的，但有助于		调试。第11.2节介绍了configASSERT()。*/
    configASSERT(pxUARTInstance->xTaskToNotify != NULL);

    /*清除中断。*/
    UART_low_level_interrupt_clear(pxUARTInstance);
    
    /*直接向名为xUART_Send()任务发送通知。如果任务在等待通知时被阻止，则该任务将从阻止状态中删除。*/
    vTaskNotifyGiveFromISR(pxUARTInstance->xTaskToNotify, &xHigherPriorityTaskWoken);

    /*现在没有任务等待通知。将xUART结构的xTaskToNotify成员设置回NULL。此步骤并非绝对必要，但有助于		调试。*/
    pxUARTInstance->xTaskToNotify = NULL;
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单155. 演示如何在驱动程序库传输函数中使用任务通知的伪代码



任务通知还可以替换接收函数中的信号量，如伪代码清单156所示，它提供了一个RTOS感知库函数的概要，该库函数在UART端口上接收数据。参见清单156：

- xUART_Receive()函数不包含任何互斥逻辑。如果有多个任务要使用xUART_Receive()函数，那么应用程序编写器必须管理应用程序本身中的互斥。例如，在调用xUART_Receive()之前，可能需要一个任务来获取互斥体。
- UART的接收中断服务程序将UART接收到的字符放到RAM缓冲区中。函数的作用是:从RAM缓冲区返回字符。

- xUART_Receive() uxWantedBytes参数用于指定要接收的字符数。如果RAM缓冲区尚未包含请求的数字字符，则调用任务将处于阻止状态，以等待通知缓冲区中的字符数已增加。while()循环用于重复此序列，直到接收缓冲区包含请求的字符数，或者发生超时。
- 调用任务可能多次进入阻塞状态。因此，将块时间调整为考虑自调用xUART_Receive()以来已经经过的时间。这些调整确保在xUART_ Receive()内花费的总时间不会超过由xUART结构的xRxTimeout成员指定的块时间。块时间是使用FreeRTOS VTaskSetTimeOutState()和xTaskCheckForTimeOut()辅助函数调整的。

```groovy
/*用于从UART接收数据的驱动程序库函数。*/
size_t xUART_Receive(xUART *pxUARTInstance, uint8_t *pucBuffer, size_t uxWantedBytes)
{
size_t uxReceived = 0;
TickType_t xTicksToWait;
TimeOut_t xTimeOut;
    
    /*记录输入此功能的时间。*/
    vTaskSetTimeOutState(&xTimeOut);
    
    /*xTicksToWait是超时值-它最初设置为此UART实例的最大接收超时。*/
    xTicksToWait = pxUARTInstance->xRxTimeout;
    
    /*保存调用此函数的任务的句柄。这本书的正文包含了关于以下行是否需要受关键部分保护的注释。*/
    pxUARTInstance->xTaskToNotify = xTaskGetCurrentTaskHandle();
    
    /*循环，直到缓冲区包含所需的字节数，或者发生超时。*/
    while (UART_bytes_in_rx_buffer(pxUARTInstance) < uxWantedBytes)
    {
        /*查找超时，调整xTicksToWait以说明到目前为止在该函数中花费的时间。*/
        if (xTaskCheckForTimeOut(&xTimeOut, &xTicksToWait) != pdFALSE)
        {
            /*在所需字节数可用之前超时，请退出循环。*/
            break;
        }
        
        /*接收缓冲区尚未包含所需的字节数。等待通知接收中断服务例程已将更多数据放入缓冲区的					xTicksToWait ticks的最大值。调用此函数时，调用任务是否已挂起通知并不重要，如果已挂起，则只		  需围绕此循环进行一次额外的循环。*/
        ulTaskNotifyTake(pdTRUE, xTicksToWait);
    }
    
    /*没有任务等待接收通知，因此将xTaskToNotify设置回NULL。这本书的正文包含了关于以下行是否需要受关	 键部分保护的注释。*/
    pxUARTInstance->xTaskToNotify = NULL;
    
    /*尝试将uxWantedBytes从接收缓冲区读取到pucBuffer中。返回读取的实际字节数（可能小于				uxWantedBytes）。*/
    uxReceived = UART_read_from_receive_buffer(pxUARTInstance, pucBuffer, uxWantedBytes);
    
    return uxReceived;
}
/*-----------------------------------------------------------*/

/*UART接收中断的中断服务程序。*/
void xUART_ReceiveISR(xUART *pxUARTInstance)
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    /*将接收到的数据复制到此UART的接收缓冲区并清除中断。*/
    UART_low_level_receive(pxUARTInstance);
    
    /*如果任务正在等待新数据的通知，请立即通知它。*/
    if (pxUARTInstance->xTaskToNotify != NULL)
    {
        vTaskNotifyGiveFromISR(pxUARTInstance->xTaskToNotify, &xHigherPriorityTaskWoken);
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
    }
```

清单156. 演示如何在驱动程序库接收函数中使用任务通知的伪代码



### 外围设备驱动程序中使用的任务通知：ADC示例

上一节演示了如何使用vTaskNotifyGiveFromISR()将任务通知从中断发送到任务。vTaskNotifyGiveFromISR()是一个简单的函数，但其功能有限；它只能将任务通知作为无值事件发送，不能发送数据。本节演示如何使用xTaskNotifyFromISR()发送带有任务通知事件的数据。清单157中所示的伪代码演示了该技术，它提供了用于模数转换器（ADC）的RTOS感知中断服务例程的概要。在清单157中：

- 假设ADC转换至少每50毫秒启动一次。
- ADC_ConversionEndISR()是ADC转换结束中断的中断服务程序，它是每次有新的ADC值可用时执行的中断。

- 由vADCTask()实现的任务处理ADC生成的每个值。假设任务的句柄在创建任务时存储在xADCTaskToNotify中。
- ADC_ConversionDisr()使用xTaskNotifyFromISR()，并将eAction参数设置为EsetValueWithoutOverwrite，向vADCTask()任务发送任务通知，并将ADC转换的结果写入任务的通知值。

- VADCTask()任务使用xTaskNotifyWait()来等待收到新ADC值可用的通知，并从其通知值检索ADC转换的结果。

```groovy
/*使用ADC的任务。 */
void vADCTask(void *pvParameters)
{
uint32_t ulADCValue;
BaseType_t xResult;
    
    /*触发ADC转换的速率。*/
    const TickType_t xADCConversionFrequency = pdMS_TO_TICKS(50);
    
    for (;;)
    {
        /*等待下一个ADC转换结果。*/
        xResult = xTaskNotifyWait(
            /*新的ADC值将覆盖旧值，因此在等待新的通知值之前无需清除任何位。*/
            0,
            /*未来的ADC值将覆盖现有值，因此在退出xTaskNotifyWait()之前无需清除任何位。*/
            0,
            /*将任务的通知值（保存最新ADC转换结果）复制到的变量的地址。*/
            &ulADCValue,
            /*应在每个XADCConversionFrequenct滴答声中接收新的ADC值。*/
            xADCConversionFrequency * 2);

        if (xResult == pdPASS)
        {
            /*接收到新的ADC值。现在就处理它。*/
            ProcessADCResult(ulADCValue);
        }
        else
        {
            /*对xTaskNotifyWait()的调用未在预期时间内返回，触发ADC转换的输入或ADC本身一定有问题。				在这里处理错误。*/
        }
    }
}
/*-----------------------------------------------------------*/

/*每次ADC转换完成时执行的中断服务例行程序。*/
void ADC_ConversionEndISR(xADC *pxADCInstance)
{
uint32_t ulConversionResult;
BaseType_t xHigherPriorityTaskWoken = pdFALSE, xResult;
    
    /*读取新的ADC值并清除中断。*/
    ulConversionResult = ADC_low_level_read(pxADCInstance);
    
    /*将通知和ADC转换结果直接发送到vADCTask()。*/
    xResult = xTaskNotifyFromISR(xADCTaskToNotify,          /* xTaskToNotify参数。*/
                                 ulConversionResult,        /* ulValue参数。*/
                                 eSetValueWithoutOverwrite, /* eAction参数。*/
                                 &xHigherPriorityTaskWoken);
    /*如果对xTaskNotifyFromISR()的调用返回pdFAIL，则该任务无法跟上生成ADC值的速率。第11.2节介绍了	configASSERT()。*/
    configASSERT(xResult == pdPASS);
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
```

清单157. 演示如何使用任务通知将值传递给任务的伪代码



### 直接在应用程序中使用的任务通知

本节通过在一个假设的应用程序中演示任务通知的使用来增强任务通知的功能，该应用程序包含以下功能：

1. 应用程序通过缓慢的互联网连接进行通信，向远程数据服务器发送数据，并从远程数据服务器请求数据。从这里开始，远程数据服务器被称为云服务器。
2. 从云服务器请求数据后，请求任务必须在阻塞状态下等待请求的数据被接收。

1. 发送任务向云服务器发送数据后，必须处于阻塞状态等待云服务器正确接收数据的确认。



软件设计原理图如图81所示。图81：

- 处理到云服务器的多个互联网连接的复杂性被封装在一个FreeRTOS任务中。该任务充当FreeRTOS应用程序中的代理服务器，被称为服务器任务。
- 应用程序任务通过调用CloudRead()从云服务器读取数据。CloudRead()不直接与云服务器通信，而是将读取请求发送到队列上的服务器任务，并从服务器任务接收请求的数据作为任务通知。

- 应用程序任务通过调用CloudWrite()将日期写入云服务器。CloudWrite()不直接与云服务器通信，而是将写请求发送到队列上的服务器任务，并从服务器任务接收写操作的结果作为任务通知。



由ClouddRead()和CloudWrite()函数发送到服务器任务的结构如清单158所示。

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/23129846/1636352261318-a237622a-4f80-4a4b-8b49-99a0e9809e0b.jpeg)

图81  从应用程序任务到云服务器的通信路径



```groovy
typedef enum CloudOperations
{
    eRead, 						/*将数据发送到云服务器。*/
    eWrite 						/*从云服务器接收数据。*/
} Operation_t;
typedef struct CloudCommand
{
    Operation_t eOperation;     /*要执行的操作（读或写）。*/
    uint32_t ulDataID;          /*标识正在读取或写入的数据。*/
    uint32_t ulDataValue;       /*仅在将数据写入云服务器时使用。*/
    TaskHandle_t xTaskToNotify; /*执行操作的任务的句柄。*/
} CloudCommand_t;
```

清单158. 队列上发送到服务器任务的结构和数据类型



CloudRead()的伪代码如清单159所示。该函数将其请求发送到服务器任务，然后调用xTaskNotifyWait()以在阻塞状态下等待，直到通知请求的数据可用为止。



显示服务器任务如何管理读取请求的伪代码如清单160所示。当从云服务器接收到数据时，服务器任务将取消阻止应用程序任务，并通过调用xTaskNotify()将接收到的数据发送到应用程序任务，并将eAction参数设置为eSetValueWithOverwrite。



清单160显示了一个简化的场景，因为它假设GetCloudData()不必等待从云服务器获取值。

```groovy
/*ulDataID标识要读取的数据。pulValue保存从云服务器接收的数据要写入的变量的地址。*/
BaseType_t CloudRead(uint32_t ulDataID, uint32_t *pulValue)
{
CloudCommand_t xRequest;
BaseType_t xReturn;
    
    /*将CloudCommand_t结构成员设置为此读取请求正确。*/
    xRequest.eOperation = eRead;                          /*这是一个读取数据的请求。*/
    xRequest.ulDataID = ulDataID;                         /*标识要读取的数据的代码。*/
    xRequest.xTaskToNotify = xTaskGetCurrentTaskHandle(); /*调用任务的句柄。*/
    
    /*通过读取块时间为0的通知值，确保没有已挂起的通知，然后将结构发送到服务器任务。*/
    xTaskNotifyWait(0, 0, NULL, 0);
    xQueueSend(xServerTaskQueue, &xRequest, portMAX_DELAY);
    
    /*等待来自服务器任务的通知。服务器任务将从云服务器接收的值直接写入该任务的通知值，因此在进入或退出		xTaskNotifyWait()函数时，无需清除通知值中的任何位。接收到的值被写入*pulValue，因此pulValue		作为通知值写入的地址被传递。*/
    xReturn = xTaskNotifyWait(0,                   /*输入时没有清除位。*/
                              0,                   /*退出时没有要清除的位。*/
                              pulValue,            /*通知值转换为*pulValue。*/
                              pdMS_TO_TICKS(250)); /*最多等待250毫秒。*/

    /*如果xReturn是pdPASS，则获得该值。如果xReturn为pdFAIL，则请求超时。*/
    return xReturn;
}
```

清单159. 云读取API函数的实现



```groovy
void ServerTask(void *pvParameters)
{
    CloudCommand_t xCommand;
    uint32_t ulReceivedValue;
    for (;;)
    {
        /*等待从任务接收下一个CloudCommand\t结构。*/
        xQueueReceive(xServerTaskQueue, &xCommand, portMAX_DELAY);
        switch (xCommand.eOperation) /*是读还是写请求？*/
        {
        case eRead:
            /*从远程云服务器获取请求的数据项。*/
            ulReceivedValue = GetCloudData(xCommand.ulDataID);
            /*调用xTaskNotify()将通知和从云服务器接收的值发送给发出请求的任务。任务的句柄是从CloudCommand_t结构获得的。*/
            xTaskNotify(xCommand.xTaskToNotify, /*任务的句柄位于结构中。*/
                        ulReceivedValue,        /*作为通知值发送的云数据。*/
                        eSetValueWithOverwrite);
            break;

            /*其他开关箱在这里。*/
        }
    }
}
```

清单160. 处理读取请求的服务器任务



CloudWrite()的伪代码如清单161所示。出于演示的目的，CloudWrite()返回一个按位状态代码，其中状态代码中的每一位都被赋予一个唯一的含义。清单161顶部的#define语句显示了四个示例状态位。



任务清除四个状态位，将其请求发送到服务器任务，然后调用xTaskNotifyWait()以在阻塞状态下等待状态通知。

```groovy
/*云写入操作使用的状态位。*/
#define SEND_SUCCESSFUL_BIT 			( 0x01 << 0 )
#define OPERATION_TIMED_OUT_BIT 		( 0x01 << 1 )
#define NO_INTERNET_CONNECTION_BIT 		( 0x01 << 2 )
#define CANNOT_LOCATE_CLOUD_SERVER_BIT 	( 0x01 << 3 )

/*设置了四个状态位的掩码。*/
#define CLOUD_WRITE_STATUS_BIT_MASK 	( SEND_SUCCESSFUL_BIT |
 										  OPERATION_TIMED_OUT_BIT |
										  NO_INTERNET_CONNECTION_BIT | 
 										  CANNOT_LOCATE_CLOUD_SERVER_BIT ) 

uint32_t CloudWrite( uint32_t ulDataID, uint32_t ulDataValue )
 {
 CloudCommand_t xRequest;
 uint32_t ulNotificationValue;
     
     /*将CloudCommand_t结构成员设置为此写入请求正确。*/
     xRequest.eOperation = eWrite;                         /*这是一个写入数据的请求。*/
     xRequest.ulDataID = ulDataID;                         /*标识正在写入的数据的代码。*/
     xRequest.ulDataValue = ulDataValue;                   /*写入云服务器的数据的值。*/
     xRequest.xTaskToNotify = xTaskGetCurrentTaskHandle(); /*调用任务的句柄。*/
     
     /*通过调用xTaskNotifyWait()，将ulBitsToClearOnExit参数设置为CLOUD_write_status_BIT_MASK，		并将块时间设置为0，清除与写入操作相关的三个状态位。当前通知值不是必需的，因此					  pulNotificationValue参数设置为NULL。*/
     xTaskNotifyWait(0, CLOUD_WRITE_STATUS_BIT_MASK, NULL, 0);
     
     /*将请求发送到服务器任务。*/
     xQueueSend(xServerTaskQueue, &xRequest, portMAX_DELAY);
     
     /*等待来自服务器任务的通知。服务器任务将按位状态代码写入此任务的通知值，该值将写入					 ulNotificationValue。*/
     xTaskNotifyWait(0,                           /*输入时没有清除位。*/
                     CLOUD_WRITE_STATUS_BIT_MASK, /*退出时将相关位清除为0。*/
                     &ulNotificationValue,        /*通知值。*/
                     pdMS_TO_TICKS(250));         /*最多等待250毫秒。*/

     /*将状态代码返回给调用任务。*/
     return (ulNotificationValue & CLOUD_WRITE_STATUS_BIT_MASK);
```

清单161. Cloud Write API函数的实现



清单162显示了演示服务器任务如何管理写入请求的伪代码。将数据发送到云服务器后，服务器任务将取消阻止应用程序任务，并通过调用xTaskNotify()将按位状态代码发送到应用程序任务，并将eAction参数设置为eSetBits。只有CLOUD_WRITE_STATUS_BIT_MASK常量定义的位才能在接收任务的通知值中更改，因此接收任务可以将其通知值中的其他位用于其他目的。



清单162显示了一个简化的场景，因为它假设SetCloudData()不必等待从远程云服务器获得确认。

```groovy
void ServerTask(void *pvParameters)
{
CloudCommand_t xCommand;
uint32_t ulBitwiseStatusCode;
    
    for (;;)
    {
        /*等待下一条消息。*/
        xQueueReceive(xServerTaskQueue, &xCommand, portMAX_DELAY);
        
        /*是读还是写请求？*/
        switch (xCommand.eOperation)
        {
        case eWrite:
            
            /*将数据发送到远程云服务器。SetCloudData()返回一个按位状态代码，该代码只使用 						CLOUD_WRITE_STATUS_BIT_MASK定义定义的位（如清单161所示）。*/
            ulBitwiseStatusCode = SetCloudData(xCommand.ulDataID, xCommand.ulDataValue);
            
            /*向发出写入请求的任务发送通知。使用该操作时，ulBitwiseStatusCode中设置的任何状态位都			 将在被通知任务的通知值中设置。所有其他位保持不变。任务的句柄是从CloudCommand_t结构获				得的。*/
            xTaskNotify(xCommand.xTaskToNotify, /*任务的句柄位于结构中。*/
                        ulBitwiseStatusCode,    /*作为通知值发送的云数据。*/
                        eSetBits);
            break;

            /*其他开关箱在这里。*/
        }
    }
}
```

清单162.  处理发送请求的服务器任务





# **低功耗支持**

待定。本章将在最终出版前撰写。





# **开发者支持**

## 引言与范围

- 提供对应用程序行为方式的深入了解。
- 突出强调优化的机会。

- 在错误发生的位置上进行捕获。



## configASSERT()

在C语言中，宏 assert() 用于验证程序所做的断言（假设）。断言被写成C语言表达式，如果表达式的值为false（0），那么断言就被视为失败。例如，清单163测试指针 pxMyPointer 不是 NULL 的断言。

```verilog
/* 测试pxMyPointer不为空 */
assert( pxMyPointer != NULL );
```

清单163 使用标准的C语言 assert() 宏来检查pxMyPointer不是 NULL 。



应用程序编写者通过提供 assert() 宏的实现，指定在断言失败时要采取的行动。



FreeRTOS的源代码没有调用 assert()，因为 assert() 在所有编译FreeRTOS的编译器中都不能使用。相反，FreeRTOS的源代码包含了对一个叫做 configASSERT() 的宏的大量调用，这个宏可以由应用程序编写者在FreeRTOSConfig.h 中定义，其行为与标准C语言的 assert() 完全一样。



一个失败的断言必须被视为一个致命的错误。不要试图执行超过断言失败的那一行。



使用 configASSERT() 可以立即捕获和识别许多最常见的错误来源，从而提高生产力。我们强烈建议在开发或调试FreeRTOS应用程序时，定义 configASSERT() 。



定义 configASSERT() 将大大有助于运行时的调试，但也会增加应用程序的代码大小，从而减慢其执行速度。如果没有提供 configASSERT() 的定义，那么将使用默认的空定义，所有对 configASSERT() 的调用将被C预处理器完全删除。

### configASSERT()定义的实例

清单164中显示的 configASSERT() 的定义在一个应用程序在调试器的控制下执行时非常有用。它将在任何断言失败的行上停止执行，因此断言失败的行将是调试器在暂停调试会话时显示的行。

```verilog
/* 禁用中断，使tick中断停止执行，然后坐在一个循环中，使执行不超过断言失败的那一行。如果硬件支持调试中断指令，那么可以用调试中断指令来代替for()循环。 */
#define configASSERT( x ) if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for(;;); }
```

清单164 一个简单的 configASSERT() 定义，在调试器的控制下执行时非常有用 



清单165中显示的 configASSERT() 的定义在应用程序没有在调试器的控制下执行时很有用。它打印出，或以其他方式记录断言失败的源代码行。使用标准的 C__FILE__ 宏来获取源文件的名称，使用标准的 C__LINE__ 宏来获取源文件的行号，来识别断言失败的行。

```verilog
/* 这个函数必须在C源文件中定义，而不是在FreeRTOSConfig.h头文件中。 */
void vAssertCalled(const char *pcFile, uint32_t ulLine)
{
    /* 在这个函数中，pcFile持有包含检测到错误的行的源文件的名称，ulLine持有源文件中的行号。pcFile和ulLine的值可以被打印出来，或者以其他方式记录下来，然后再进入下面的无限循环。 */
    RecordErrorInformationHere(pcFile, ulLine);
    /* 禁用中断，使tick中断停止执行，然后坐在一个循环中，使执行不超过断言失败的那一行。 */
    taskDISABLE_INTERRUPTS();
    for (;;)
        ;
}
/*-----------------------------------------------------------*/
/* 下面这两行必须放在FreeRTOSConfig.h中。 */
extern void vAssertCalled(const char *pcFile, uint32_t ulLine);
#define configASSERT(x) \
    if ((x) == 0)       \
    vAssertCalled(__FILE__, __LINE__)
```

清单165 记录断言失败的源代码行的 configASSERT() 定义



## FreeRTOS+Trace

FreeRTOS+Trace是一个运行时诊断和优化工具，由我们的合作伙伴Percepio公司提供。



FreeRTOS+Trace可以捕获有价值的动态行为信息，然后在相互连接的图形视图中展示捕获的信息。



在分析、排除故障或简单地优化FreeRTOS应用程序时，所捕获的信息是非常宝贵的。



FreeRTOS+Trace可以与传统的调试器并排使用，并以更高层次的时间视角来补充调试器的观点。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636343690571-cbaa44ca-b585-493e-97be-3c13e04fe035.png)

图82 FreeRTOS+Trace包括20多个相互连接的视图

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636343747247-99193b55-dad4-434d-92d9-abd590bcdab8.png)

图83 FreeRTOS+Trace主跟踪视图——20多个相互连接的跟踪视图之一

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636342876673-4dbbc85e-6046-4871-b829-f90d8c05dc1d.png)

图84 FreeRTOS+Trace CPU负载视图——20多个相互连接的跟踪视图之一

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636342969529-efd3b559-3c39-4991-b5f5-ffaab7e91bf1.png)

图85 FreeRTOS+Trace响应时间视图——20多个相互连接的跟踪视图之一

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636343055008-54ae9435-ee01-40bd-b879-94ce2a0e4e4f.png)

图86 FreeRTOS+Trace用户事件图视图——20多个相互连接的跟踪视图之一

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636343819967-8a79f228-0196-4cab-9ad6-35bee51e5643.png)

图87 FreeRTOS+Trace内核对象历史视图 - 20多个相互连接的跟踪视图之一



## 调试相关的钩（回调）函数

### Malloc失败的钩

malloc失败的钩（或回调）在第2章，堆内存管理中有所描述。



定义一个malloc失败钩可以确保在试图创建任务、队列、信号或事件组失败时立即通知应用开发者。



### 堆栈溢出钩

堆栈溢出钩的细节在堆栈溢出那一节中提供。



定义一个堆栈溢出钩可以确保在一个任务使用的堆栈量超过分配给该任务的堆栈空间时，应用程序开发人员得到通知。



## 查看运行时和任务状态信息

### 任务运行时间统计

任务运行时间统计提供了关于每个任务所获得的处理时间的信息。一个任务的运行时间是该任务自应用程序启动以来一直处于运行状态的总时间。



运行时统计的目的是在项目的开发阶段作为剖析和调试的辅助工具使用。项目的开发阶段。它们提供的信息只在作为运行时统计时钟的计数器溢出之前有效。收集运行时统计将增加任务上下文切换时间。



要获得二进制运行时统计信息，请调用 uxTaskGetSystemState() API函数。要获得以人类可读的ASCII表形式的运行时统计信息，请调用 vTaskGetRunTimeStats() 辅助函数。



### 运行时间统计时钟

运行时统计需要测量滴答周期的几分之一。因此，RTOS的tick计数不被用作运行时统计时钟，而是由应用程序代码提供时钟。建议使运行时统计时钟的频率比tick中断的频率快10到100倍。运行时统计时钟越快，统计就越准确，但时间值也会越早溢出。



理想情况下，时间值将由一个自由运行的32位外设定时器/计数器产生，其值可以在没有其他处理开销的情况下被读取。如果可用的外设和时钟速度不能使这种技术成为可能，那么替代性的但效率较低的技术包括：

1. 配置一个外设，使其在所需的运行时统计时钟频率下产生一个周期性中断，然后使用产生的中断数量的计数作为运行时统计时钟。



如果周期性中断只用于提供运行时的统计时钟，那么这种方法是非常低效的。然而，如果应用程序已经使用了频率合适的周期性中断，那么将产生的中断数量的计数加入到现有的中断服务例程中，就很简单和有效。



1. 用一个自由运行的16位外设定时器的当前值作为32位值中最没有意义的16位，用定时器溢出的次数作为32位值中最没有意义的16位，从而产生一个32位值。



通过适当的、有点复杂的操作，可以将RTOS的滴答声与ARM Cortex-M SysTick定时器的当前值结合起来，产生一个运行时统计时钟。FreeRTOS下载中的一些演示项目展示了如何实现这一目标。



### 配置应用程序以收集运行时统计信息

表54详细介绍了收集任务运行时统计数据所需的宏。原本打算将这些宏包含在RTOS端口层中，这就是为什么这些宏的前缀是 "port"，但事实证明在 FreeRTOSConfig.h 中定义它们更为实用。



表 54. 用于收集运行时统计数据的宏程序

| 宏观                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| configGENERATE_RUN_TIME_STATS                                | 这个宏必须在FreeRTOSConfig.h中设置为1。当这个宏被设置为1时，调度器将在适当的时候调用本表中详述的其他宏。 |
| portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()                     | 必须提供这个宏来初始化用于提供运行时统计时钟的任何一个外设。 |
| portGET_RUN_TIME_COUNTER_VALUE(), or portALT_GET_RUN_TIME_COUNTER_VALUE(Time) | 必须提供这两个宏中的一个来返回当前的运行时统计时钟值。这是自应用程序首次启动以来，应用程序运行的总时间，以运行时统计时钟为单位。如果使用第一个宏，它必须被定义为评估为当前的时钟值。如果使用第二个宏，它必须被定义为将其 “时间”参数设置为当前时钟值。 |

### uxTaskGetSystemState() API函数

uxTaskGetSystemState() 为FreeRTOS调度器控制下的每个任务提供了一个状态信息快照。这些信息是以TaskStatus_t结构数组的形式提供的，每个任务在数组中都有一个索引。TaskStatus_t由清单167和表56描述。

```verilog
UBaseType_t uxTaskGetSystemState(TaskStatus_t *const pxTaskStatusArray,
                                 const UBaseType_t uxArraySize,
                                 uint32_t *const pulTotalRunTime);
```

清单 166. uxTaskGetSystemState() API函数原型



表 55 uxTaskGetSystemState() 参数和返回值

| 参数名称          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| pxTaskStatusArray | 一个指向TaskStatus_t结构数组的指针。该数组必须为每个任务至少包含一个TaskStatus_t结构。任务的数量可以用uxTaskGetNumberOfTasks() API函数来确定。TaskStatus_t结构在清单167中显示，TaskStatus_t结构成员在表56中描述。 |
| uxArraySize       | pxTaskStatusArray参数所指向的数组的大小。该大小被指定为数组中的索引数（数组中包含的TaskStatus_t结构数），而不是数组中的字节数。 |
| pulTotalRunTime   | 如果configGENERATE_RUN_TIME_STATS被设置为1，在 中设置为1，那么*pulTotalRunTime就会被 uxTaskGetSystemState() 设置为目标启动后的总运行时间（由应用程序提供的运行时间统计时钟定义）。pulTotalRunTime是可选的，如果不需要总运行时间，可以设置为NULL。 |
| 返回值            | 返回由 uxTaskGetSystemState() 填充的TaskStatus_t结构的数量。返回的值应该等于 uxTaskGetNumberOfTasks() API函数返回的数字，但如果在uxArraySize参数中传递的值太小，则会为零。 |

```verilog
typedef struct xTASK_STATUS
{
    TaskHandle_t xHandle;
    const char *pcTaskName;
    UBaseType_t xTaskNumber;
    eTaskState eCurrentState;
    UBaseType_t uxCurrentPriority;
    UBaseType_t uxBasePriority;
    uint32_t ulRunTimeCounter;
    uint16_t usStackHighWaterMark;
} TaskStatus_t;
```

清单 167. TaskStatus_t结构



表 56. TaskStatus_t结构成员

| 参数名称/返回值      | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| xHandle              | 该结构中的信息所涉及的任务的句柄。                           |
| pcTaskName           | 任务的可读性文本名称。                                       |
| xTaskNumber          | 每个任务都有唯一的xTaskNumber值。如果一个应用程序在运行时创建和删除任务，那么一个任务有可能与之前被删除的任务有相同的句柄。xTaskNumber被提供给应用程序代码和内核感知调试器，以区分一个仍然有效的任务和一个与有效任务有相同句柄的被删除任务。 |
| eCurrentState        | 一个保持任务状态的枚举类型。eCurrentState可以是以下值之一：eRunning, eReady, eBlocked, eSuspended, eDeleted。一个任务只有在调用 vTaskDelete() 删除任务后，到Idle任务释放分配给被删除任务的内部数据结构和堆栈的内存这段时间内，才会被报告为处于eDeleted状态。在这段时间之后，该任务将不再以任何方式存在，试图使用其句柄是无效的。 |
| uxCurrentPriority    | 在调用 uxTaskGetSystemState() 时，任务运行的优先级。只有当任务按照7.3节 "互斥（和二进制半挂）"中描述的优先级继承机制暂时被分配了更高的优先级时，uxCurrentPriority才会高于应用编写者分配给任务的优先级。 |
| uxBasePriority       | 应用程序编写者分配给任务的优先级。只有在FreeRTOSConfig.h 中 configUSE_MUTEXES 被设置为1时，uxBasePriority才有效。 |
| ulRunTimeCounter     | 自任务创建以来，任务使用的总运行时间。总运行时间是以绝对时间的形式提供的，它使用应用程序编写者提供的时钟来收集运行时间的统计数据。ulRunTimeCounter 只有在 FreeRTOSConfig.h 中configGENERATE_RUN_TIME_STATS 被设置为1时才有效。 |
| usStackHighWaterMark | 任务的堆积高水位。这是自任务创建以来，任务所剩的最小堆栈空间。它是任务离溢出堆栈有多远的指示；这个值越接近零，任务就越接近溢出堆栈。 usStackHighWaterMark 的单位是字节。 |

### vTaskList() 辅助函数

vTaskList() 提供的任务状态信息与 uxTaskGetSystemState() 提供的信息类似，但它以人类可读的ASCII表而不是二进制值数组的形式呈现信息。



vTaskList() 是一个非常耗费处理器的函数，并且会让调度器在很长一段时间内悬浮。因此，建议该函数仅用于调试目的，而不是在生产性实时系统中使用。



如果 FreeRTOSConfig.h 中的 configUSE_TRACE_FACILITY 和 configUSE_STATS_FORMATTING_FUNCTIONS 都被设置为1，则 vTaskList() 可用。

```verilog
void vTaskList( signed char *pcWriteBuffer );
```

清单 168. vTaskList() API函数原型



表 57. vTaskList() 参数

| 参数名称      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| pcWriteBuffer | 一个指向字符缓冲区的指针，格式化和人类可读的表被写入其中。该缓冲区必须大到足以容纳整个表，因为不进行边界检查。 |

图88显示了 vTaskList() 生成的输出的一个例子。 在输出中：



- 每一行都提供了一个单一的任务信息。
- 第一列是任务名称。

- 第二列是任务的状态，其中'R'表示就绪，'B'表示阻塞，'S'表示暂停，'D'表示任务已被删除。一个任务只有在调用 vTaskDelete() 删除任务后，到Idle任务释放分配给被删除任务的内部数据结构和堆栈的内存这段时间内，才会被报告为处于删除状态。在这段时间之后，该任务将不再以任何方式存在，试图使用其句柄是无效的。
- 第三列是任务的优先级。

- 第四列是任务的堆栈高水位。见表56中对 usStackHighWaterMark 的描述。
- 第五列是分配给该任务的唯一编号。见表56中对xTaskNumber的描述。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636353441516-6a3312ed-11d9-435c-a274-b2a9ca36ade5.png)

图88 由vTaskList()生成的输出示例

### vTaskGetRunTimeStats() 辅助函数

vTaskGetRunTimeStats() 将收集到的运行时统计数据格式化为人类可读的ASCII表格。



vTaskGetRunTimeStats() 是一个非常耗费处理器的函数，并且会让调度器暂停很长一段时间。因此，建议该函数仅用于调试目的，而不是在生产型实时系统中使用。



当 FreeRTOSConfig.h 中 configGENERATE_RUN_TIME_STATS 和 configUSE_STATS_FORMATTING_FUNCTIONS 都被设置为1时，vTaskGetRunTimeStats() 可用。

```verilog
void vTaskGetRunTimeStats( signed char *pcWriteBuffer );
```

清单169. vTaskGetRunTimeStats() API函数原型



表 58. vTaskGetRunTimeStats() 参数

| 参数值        | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| pcWriteBuffer | 一个指向字符缓冲区的指针，格式化和人类可读的表被写入其中。该缓冲区必须大到足以容纳整个表，因为不进行边界检查。 |

图89显示了 vTaskGetRunTimeStats() 产生的一个输出例子。在输出中：

- 每一行都提供了一个单一的任务信息。
- 第一列是任务名称。

- 第二列是任务在运行状态下花费的时间，是一个绝对值。见表56中 ulRunTimeCounter 的描述。
- 第三列是任务在运行状态下花费的时间，占目标启动后总时间的百分比。显示的百分比时间的总和通常会小于预期的100%，因为统计数据的收集和计算是使用整数计算，四舍五入到最近的整数值。

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636354332626-801684d7-47e1-4065-aebc-b8326dc379d6.png)

图89 由 vTaskGetRunTimeStats() 生成的输出示例

### 生成和显示运行时的统计数据，一个工作实例

这个例子使用一个假设的16位定时器来产生一个32位的运行时统计时钟。该计数器被配置为在每次16位值达到最大值时产生一个中断，有效地产生一个溢出中断。中断服务程序对溢出发生的次数进行计数。



32位值是通过使用溢出发生次数作为32位值中最有意义的两个字节，以及使用当前16位计数器值作为32位值中最没有意义的两个字节来创建的。中断服务例程的伪代码显示在清单170中。

```verilog
void TimerOverflowInterruptHandler(void)
{
    /* 只需计算中断的数量。 */
    ulOverflowCount++;
    /* 清除中断。 */
    ClearTimerInterrupt();
}
```

清单 170. 用于计算定时器溢出的16位定时器溢出中断处理程序



清单171显示了添加到FreeRTOSConfig.h中的行，以实现对运行时统计数据的收集。

```verilog
/* 将 configGENERATE_RUN_TIME_STATS 设置为 1 以启用运行时统计数据的收集。这样做时， 必须同时定义 portCONFIGURE_TIMER_FOR_RUN_TIME_STATS() 和 portGET_RUN_TIME_COUNTER_VALUE() 或 portALT_GET_RUN_TIME_COUNTER_VALUE(x)。 */
#define configGENERATE_RUN_TIME_STATS 1
/* portCONFIGURE_TIMER_FOR_RUN_TIME_STATS()被定义为调用设置假设的16位定时器的函数(该函数的实现没有显示)。 */
void vSetupTimerForRunTimeStats(void);
#define portCONFIGURE_TIMER_FOR_RUN_TIME_STATS() vSetupTimerForRunTimeStats()
/* portALT_GET_RUN_TIME_COUNTER_VALUE()的定义是将其参数设置为当前的运行时间计数器/时间值。返回的时间值是32位的，是通过将16位定时器溢出的计数移到32位数字的前两个字节中，然后将结果与当前的16位计数器值进行位法OR形成的。 */
#define portALT_GET_RUN_TIME_COUNTER_VALUE(ulCountValue)             \
    {                                                                \
        extern volatile unsigned long ulOverflowCount;               \
                                                                     \
        /* 断开时钟与计数器的连接，使其不发生变化 										 \
        而其值正在被使用。 */                               					\
        PauseTimer();                                                \
                                                                     \
        /* 溢出的数量被转移到最重要的                   							 \
        返回的32位数值的两个字节。 */                   							\
        ulCountValue = (ulOverflowCount << 16UL);                    \
                                                                     \
        /* 当前的计数器值被用作最小有效值。          									\
        返回的32位数值的两个字节。 */                    							\
        ulCountValue |= (unsigned long)ReadTimerCount();             \
                                                                     \
        /* 重新将时钟连接到计数器。 */                   							\
        ResumeTimer();                                               \
    }
```

清单 171. 添加到 FreeRTOSConfig.h 中的宏，以实现对运行时统计数据的收集



清单172中显示的任务每5秒打印出收集到的运行时统计数据。

```verilog
/* 为了清楚起见，本代码清单中省略了对fflush()的调用。 */
static void prvStatsTask(void *pvParameters)
{
    TickType_t xLastExecutionTime;
    /* 用来保存格式化的运行时统计文本的缓冲区需要相当大。因此，它被声明为静态的，以确保它不被分配在任务栈上。这使得这个函数不能重入。 */
    static signed char cStringBuffer[512];
    /* 该任务将每5秒运行一次。 */
    const TickType_t xBlockPeriod = pdMS_TO_TICKS(5000);
    /* 将xLastExecutionTime初始化为当前时间。这是该变量唯一需要明确写入的时间。之后，它将在vTaskDelayUntil()API函数中进行内部更新。 */
    xLastExecutionTime = xTaskGetTickCount();
    /* 和大多数任务一样，这个任务是在一个无限循环中实现的。 */
    for (;;)
    {
        /* 等到再次运行这项任务的时候。 */
        vTaskDelayUntil(&xLastExecutionTime, xBlockPeriod);
        /* 从运行时统计信息中生成一个文本表。这必须适合于cStringBuffer数组。 */
        vTaskGetRunTimeStats(cStringBuffer);

        /* 打印运行时统计表的列标题。 */
        printf("\nTask\t\tAbs\t\t\t%%\n");
        printf("-------------------------------------------------------------\n");

        /* 打印出运行时统计数据本身。数据表格包含多行，所以调用vPrintMultipleLines()函数，而不是直接调用printf()。vPrintMultipleLines()只是对每一行单独调用printf()，以确保行缓冲按预期工作。 */
        vPrintMultipleLines(cStringBuffer);
    }
}
```

清单172. 打印出收集到的运行时统计数据的任务



## 追踪钩宏

跟踪宏是被放置在FreeRTOS源代码的关键点上的宏。默认情况下，这些宏是空的，因此不产生任何代码，也没有运行时间的开销。通过覆盖默认的空实现，应用程序编写者可以：



- 在不修改FreeRTOS源文件的情况下将代码插入FreeRTOS。
- 通过目标硬件上的任何手段输出详细的执行顺序信息。跟踪宏出现在FreeRTOS源代码中的足够多的地方，允许它们被用来创建一个完整和详细的调度器活动跟踪和分析日志。



### 可用的追踪钩宏

如果在这里详细介绍每一个宏，会占用太多的篇幅。表59详细列出了被认为对应用程序编写者最有用的宏子集。



表59中的许多描述提到了一个叫做pxCurrentTCB的变量。一个FreeRTOS的私有变量，它持有运行状态下任务的句柄，并且对任何从 FreeRTOS/Source/tasks.c 源文件中调用的宏是可用的。



表 59. 最常用的跟踪钩宏的选择

| 宏观                                        | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| traceTASK_INCREMENT_TICK(xTickCount)        | 在tick中断期间，在tick计数被增加后调用。xTickCount参数将新的tick计数值传递给宏。 |
| traceTASK_SWITCHED_OUT()                    | 在一个新的任务被选中运行之前被调用。此时，pxCurrentTCB包含即将离开运行状态的任务的句柄。 |
| traceTASK_SWITCHED_IN()                     | 在一个任务被选中运行后被调用。此时，pxCurrentTCB包含即将进入运行状态的任务的句柄。 |
| traceBLOCKING_ON_QUEUE_RECEIVE(pxQueue)     | 在当前执行的任务试图从空队列中读取数据，或者试图 "夺取 "空信号或突发事件后进入阻塞状态前立即调用。pxQueue参数将目标队列或 semaphore 的句柄传递给宏。 |
| traceBLOCKING_ON_QUEUE_SEND(pxQueue)        | 在试图向已满的队列写入后，当前执行的任务进入阻塞状态前立即调用。pxQueue参数将目标队列的句柄传入宏。 |
| traceQUEUE_SEND(pxQueue)                    | 在xQueueSend()、xQueueSendToFront()、xQueueSendToBack()或任何semaphore "give"函数中调用，当队列发送或semaphore "give"成功时，pxQueue参数将目标队列或 semaphore 的句柄传入该宏。 |
| traceQUEUE_SEND_FAILED(pxQueue)             | 从xQueueSend()、xQueueSendToFront()、xQueueSendToBack() 或任何semaphore "give"函数中调用，当队列发送或semaphore "give"操作失败时。如果队列已满，并且在指定的任何块时间内保持满的状态，那么队列发送或semaphore "give" 将失败。pxQueue参数将目标队列或信号灯的句柄传递给宏。 |
| traceQUEUE_RECEIVE(pxQueue)                 | 在 xQueueReceive() 或任何semaphore "take" 函数中调用，当队列接收或semaphore "take "成功时。pxQueue参数将目标队列或信号灯的句柄传递给宏。 |
| traceQUEUE_RECEIVE_FAILED(pxQueue)          | 当队列或信号灯接收操作失败时，从xQueueReceive() 或任何信号灯 "take" 函数中调用。如果队列或信号灯是空的，并且在指定的时间段内保持空的状态，那么队列接收或信号灯 "take" 的操作将失败。pxQueue参数将目标队列或信号灯的句柄传递给宏。 |
| traceQUEUE_SEND_FROM_ISR(pxQueue)           | 当发送操作成功时，从 xQueueSendFromISR() 中调用。pxQueue参数将目标队列的句柄传入宏。 |
| traceQUEUE_SEND_FROM_ISR_FAILED(pxQueue)    | 当发送操作失败时，从 xQueueSendFromISR() 中调用。如果队列已经满了，发送操作将失败。pxQueue参数将目标队列的句柄传入宏。 |
| traceQUEUE_RECEIVE_FROM_ISR(pxQueue)        | 当接收操作成功时，从 xQueueReceiveFromISR() 中调用。pxQueue参数将目标队列的句柄传给宏。 |
| traceQUEUE_RECEIVE_FROM_ISR_FAILED(pxQueue) | 当接收操作由于队列已经为空而失败时，在xQueueReceiveFromISR() 中调用。pxQueue参数将目标队列的句柄传入宏。 |
| traceTASK_DELAY_UNTIL()                     | 从内部调用 vTaskDelayUntil() 中，在调用任务进入阻塞状态前立即调用。 |
| traceTASK_DELAY()                           | 在调用任务进入阻塞状态之前，从 vTaskDelay() 中调用。         |

### 定义跟踪钩宏

每个跟踪宏都有一个默认的空定义。默认定义可以通过在 FreeRTOSConfig.h 中提供一个新的宏定义来覆盖。如果跟踪宏的定义变得很长或很复杂，那么它们可以在一个新的头文件中实现，然后本身包含在FreeRTOSConfig.h中。



根据软件工程的最佳实践，FreeRTOS坚持严格的数据隐藏政策。追踪宏允许用户代码被添加到FreeRTOS的源文件中，因此追踪宏所看到的数据类型将与应用代码所看到的数据类型不同：



- 在 FreeRTOS/Source/tasks.c 源文件中，任务句柄是一个指向描述任务的数据结构（任务的任务控制块，或TCB）的指针。在 FreeRTOS/Source/tasks.c 源文件之外，任务句柄是一个指向void的指针。
- 在 FreeRTOS/Source/queue.c 源文件中，队列句柄是一个指向描述队列的数据结构的指针。在FreeRTOS/Source/queue.c 源文件之外，一个队列句柄是一个指向void的指针。



如果一个通常是私有的FreeRTOS数据结构被跟踪宏直接访问，需要特别小心，因为私有数据结构在FreeRTOS版本之间可能会发生变化。

### FreeRTOS感知调试器插件

提供一些FreeRTOS意识的插件可用于以下IDE。 这个列表可能并不详尽:



- Eclipse (StateViewer)
- Eclipse (ThreadSpy)

- IAR
- ARM DS-5 

- Atollic TrueStudio
- Microchip MPLAB

- iSYSTEM WinIDEA

![img](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636357790972-483d19f5-328f-478b-929d-713b284d0f07.png)

图90 FreeRTOS ThreadSpy Eclipse 插件，来自Code Confidence Ltd



# **故障排除**



## 引言与范围

本章强调了刚接触FreeRTOS的用户最常遇到的问题。首先，它集中讨论了三个问题，这三个问题已被证明是多年来最频繁的支持请求的来源；不正确的中断优先级分配，堆栈溢出，以及不适当地使用 printf()。然后，它以FAQ的形式简要介绍了其他常见的错误，它们可能的原因，以及它们的解决方案。



使用 configASSERT() 可以立即捕获和识别许多最常见的错误来源，从而提高生产力。强烈建议在开发或调试FreeRTOS应用程序时定义 configASSERT() 。configASSERT() 在之前描述过。



## 中断优先级

注意: 这是导致支持请求的头号原因，在大多数端口中，定义 configASSERT() 将立即捕获这个错误



如果使用的FreeRTOS端口支持中断嵌套，并且中断的服务例程使用了FreeRTOS的API，那么必须将中断的优先级设置为或低于configMAX_SYSCALL_INTERRUPT_PRIORITY，如6.8节所述，中断嵌套。如果不这样做，将导致无效的关键部分，这反过来又会导致间歇性的故障。



如果在一个处理器上运行FreeRTOS，请特别注意：



- 中断的优先级默认为具有最高的优先级，在一些ARM Cortex处理器上是这样的，可能还有其他处理器。在这种处理器上，使用FreeRTOS API的中断的优先级不能不被初始化。
- 数字上的高优先级数字代表逻辑上的低中断优先级，这似乎有悖于直觉，因此会引起混淆。这也是ARM Cortex处理器的情况，也可能是其他处理器的情况。

- 例如，在这样的处理器上，一个正在执行的优先级为5的中断本身可以被一个优先级为4的中断打断。 因此，如果configMAX_SYSCALL_INTERRUPT_PRIORITY被设置为5，任何使用FreeRTOS API的中断只能被分配一个数字上高于或等于5的优先级。 在这种情况下，5或6的中断优先级将是有效的，但3的中断优先级肯定是无效的。
- 不同的库实现希望以不同的方式来指定中断的优先级。同样，这与以ARM Cortex处理器为目标的库特别相关，在那里，中断的优先级在写入硬件寄存器之前被移位。有些库会自己执行位移，而其他库则希望在优先级被传入库函数之前执行位移。

- 同一架构的不同实现方式实现了不同数量的中断优先位。例如，一个制造商的Cortex-M处理器可能实现3个优先位，而另一个制造商的Cortex-M处理器可能实现4个优先位。
- 定义中断优先级的位可以分成定义抢占式优先级的位和定义子优先级的位。确保所有的位都被分配给指定一个抢占性优先级，所以子优先级不被使用。

- 在某些 FreeRTOS 端口中， configMAX_SYSCALL_INTERRUPT_PRIORITY 有另一个名字 configMAX_API_CALL_INTERRUPT_PRIORITY。



## 堆栈溢出

堆栈溢出是支持请求的第二大来源。FreeRTOS提供了一些功能来帮助捕获和调试堆栈相关的问题1。



### uxTaskGetStackHighWaterMark() API函数

uxTaskGetStackHighWaterMark() 用于查询一个任务离溢出分配给它的堆栈空间还有多远。这个值被称为堆栈 "高水位线"。

```verilog
UBaseType_t uxTaskGetStackHighWaterMark( TaskHandle_t xTask );
```

清单173. uxTaskGetStackHighWaterMark() API函数原型



表 60. uxTaskGetStackHighWaterMark() 参数和返回值

| 参数名称/返回值 | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| xTask           | 堆栈高水位被查询的任务的句柄（主题任务——参见xTaskCreate() API函数的pxCreatedTask参数，了解获取任务句柄的信息。一个任务可以通过传递NULL来代替一个有效的任务句柄来查询自己的堆栈高水位。 |
| 返回值          | 任务使用的堆栈量随着任务的执行和中断的处理而增长和缩小。uxTaskGetStackHighWaterMark() 返回任务开始执行后剩余堆栈空间的最小量。这是堆栈使用量最大（或最深）时未使用的堆栈的数量。高水位线越接近于零，任务就越接近于溢出其堆栈。 |

1这些功能在FreeRTOS的Windows端口中是不可用的。

### 运行时间堆栈检查——概述

FreeRTOS包括两个可选的运行时堆栈检查机制。这两种机制由 FreeRTOSConfig.h 中的configCHECK_FOR_STACK_OVERFLOW编译时配置常数控制。



堆栈溢出钩（或堆栈溢出回调）是一个函数，当内核检测到堆栈溢出时，它被调用。调用的函数。要使用一个堆栈溢出钩函数：



1. 在 FreeRTOSConfig.h 中把configCHECK_FOR_STACK_OVERFLOW设置为1或2，如下面各小节所述。
2. 提供钩函数的实现，使用清单174中所示的确切的函数名称和原型。

```verilog
void vApplicationStackOverflowHook( TaskHandle_t *pxTask, signed char *pcTaskName );
```

清单174. 堆栈溢出钩函数原型



提供堆栈溢出钩是为了使捕获和调试堆栈错误更容易，但当堆栈溢出发生时，没有真正的方法来恢复。该函数的参数将已经溢出堆栈的任务的句柄和名称传递给钩函数。



堆栈溢出钩从中断的上下文中被调用。



一些微控制器在检测到不正确的内存访问时产生一个故障异常，有可能在内核有机会调用堆栈溢出钩函数之前就触发了一个故障。



### 运行时堆栈检查——方法一

当configCHECK_FOR_STACK_OVERFLOW被设置为1时，方法一被选中。



一个任务的整个执行环境在每次被换出时都会被保存到它的堆栈中。这很可能是堆栈使用达到高峰的时候。当configCHECK_FOR_STACK_OVERFLOW被设置为1时，内核会检查堆栈指针在上下文被保存后是否仍在有效的堆栈空间内。如果发现堆栈指针超出其有效范围，则调用堆栈溢出钩。



方法一执行起来很快，但会错过上下文切换之间发生的堆栈溢出。



### 运行时堆栈检查——方法二

方法二在方法一已经描述的基础上执行额外的检查。当configCHECK_FOR_STACK_OVERFLOW被设置为2时，它被选中。



当一个任务被创建时，它的堆栈被填充了一个已知的模式。方法二测试任务堆栈空间的最后20个有效字节，以验证该模式没有被覆盖。如果这20个字节中的任何一个改变了它们的预期值，就会调用堆栈溢出钩函数。



方法二的执行速度没有方法一快，但仍然比较快，因为只测试20个字节。 最有可能的是，它将捕捉到所有的堆栈溢出；然而，有可能（但极不可能）会错过一些溢出。



## 不当使用printf()和sprintf()

不适当地使用 printf() 是一个常见的错误来源，而且，由于没有意识到这一点，应用程序开发人员通常会进一步调用 printf() 来帮助调试，这样做会使问题更加严重。



许多交叉编译器供应商将提供一个适合在小型嵌入式系统中使用的 printf() 实现。即使是这样，该实现也可能不是线程安全的，可能不适合在中断服务例程中使用，而且根据输出的方向，需要花费相对较长的时间来执行。



如果没有专门为小型嵌入式系统设计的 printf() 实现，而使用通用的 printf() 实现，则必须特别小心，如：



- 仅仅包括对 printf() 或 sprintf() 的调用就会大量增加应用程序的可执行文件的大小。
- printf() 和 sprintf() 可能会调用 malloc() ，如果使用的是heap_3以外的内存分配方案，这可能是无效的。更多信息请参见内存分配方案示例。

- printf() 和 sprintf() 可能需要一个比原来大很多倍的栈。

### Printf-stdarg.c

许多FreeRTOS演示项目使用了一个名为printf——stdarg.c的文件，它提供了一个最小的、堆栈有效的sprintf() 实现，可以用来代替标准库版本。在大多数情况下，这将允许为每个调用 sprintf() 和相关函数的任务分配一个小得多的堆栈。



printf-stdarg.c 还提供了一种机制，用于将 printf() 的输出逐个引向一个端口，这虽然很慢，但可以进一步减少栈的使用。



请注意，并非所有包含在FreeRTOS下载中的printf——stdarg.c的副本都实现了snprintf()。 没有实现snprintf() 的副本只是忽略了缓冲区大小参数，因为它们直接映射到sprintf()。



Printf-stdarg.c是开源的，但由第三方拥有，因此与FreeRTOS分开许可。许可证条款包含在源文件的顶部。



## 其他常见的错误来源

### 症状：在演示中添加一个简单的任务会导致演示崩溃

创建一个任务需要从堆中获取内存。许多演示应用程序项目都要求堆的大小正好够创建演示任务——因此，在任务创建之后，将没有足够的堆来添加任何进一步的任务、队列、事件组或信号。



当 vTaskStartScheduler() 被调用时，空闲任务以及可能的RTOS守护任务会被自动创建。vTaskStartScheduler() 只会在没有足够的堆内存来创建这些任务时返回。在调用 vTaskStartScheduler() 后加入一个空循环[ for(;;);]可以使这个错误更容易被调试。



为了能够添加更多的任务，要么增加堆的大小，要么删除一些现有的演示任务。更多信息请参见内存分配方案示例。



### 症状：在一个中断中使用API函数会导致应用程序崩溃

不要在中断服务程序中使用API函数，除非API函数的名称以"...FromISR() "结尾。特别是，不要在一个中断中创建一个关键部分，除非使用中断安全宏。参见：从ISR中使用FreeRTOS API，以获得更多信息。



在支持中断嵌套的FreeRTOS端口中，不要在被分配的中断优先级高于configMAX_SYSCALL_INTERRUPT_PRIORITY的中断中使用任何API函数。更多信息请参见 ：中断嵌套。



### 症状：有时应用程序在一个中断服务例程中崩溃

首先要检查的是，中断没有导致堆栈溢出。有些端口只检查任务内的堆栈溢出，而不是中断内的堆栈溢出。



中断的定义和使用方式在不同的端口和不同的编译器之间是不同的。因此，第二件要检查的事情是，在中断服务例程中使用的语法、宏和调用约定，与所使用的端口的文档页上的描述完全一致，并且与端口提供的演示应用程序中的演示完全一致。



如果应用程序运行在一个使用数字上的低优先级数字来代表逻辑上的高优先级的处理器上，那么确保分配给每个中断的优先级考虑到这一点，因为它可能看起来是反直觉的。如果应用程序运行在一个将每个中断的优先级默认为最大可能的优先级的处理器上，那么要确保每个中断的优先级不停留在其默认值。更多信息请参见：中断嵌套和中断优先级。



### 症状：调度程序在试图启动第一个任务时崩溃

确保FreeRTOS的中断处理程序已经安装。请参考正在使用的FreeRTOS端口的文档页，以及为该端口提供的演示应用程序的例子。



一些处理器在启动调度程序之前必须处于特权模式。实现这一目标的最简单方法是在调用main()之前，C语言启动代码中把处理器放入特权模式。



### 症状：中断被意外地禁用，或关键部分不能正确嵌套

如果一个FreeRTOS API函数在调度器启动之前被调用，那么中断将被故意关闭，直到第一个任务开始执行时才重新启用。这样做是为了保护系统在系统初始化期间，在调度程序启动之前，以及在调度程序可能处于不一致的状态下，避免中断试图使用FreeRTOS API函数而导致的崩溃。



除了调用 taskENTER_CRITICAL() 和 taskEXIT_CRITICAL() 以外，不要用任何方法改变微控制器的中断使能位或优先级标志。这些宏保持其调用嵌套深度的计数，以确保只有当调用嵌套完全解开为零时，中断才会再次启用。请注意，一些库函数本身可能会启用和禁用中断。



### 症状：应用程序甚至在调度程序启动之前就崩溃

在调度器启动之前，不允许执行可能导致上下文切换的中断服务例程。同样的情况也适用于任何试图向FreeRTOS对象（如队列或信号）发送或接收的中断服务例程。在调度器启动之后，才能发生上下文切换。



许多API函数在调度器启动后才能被调用。最好是将API的使用限制在创建对象上，比如任务、队列和semaphores，而不是使用这些对象，直到调用 vTaskStartScheduler() 之后。



### 症状：在调度器暂停时调用API函数，或从关键部分内部调用API函数，导致应用程序崩溃

调用 vTaskSuspendAll() 暂停调度程序，并通过调用 xTaskResumeAll() 恢复（未暂停）。通过调用taskENTER_CRITICAL() 进入关键部分，并通过调用 taskEXIT_CRITICAL() 退出。



不要在调度程序暂停时或从关键部分内部调用API函数。