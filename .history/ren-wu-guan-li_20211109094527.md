

<a name="kEk4u"></a>
# **堆内存管理**

---

从 FreeRTOS V9.0.0 开始 FreeRTOS 应用程序可以完全静态分配，无需包含堆内存管理器。

---

<a name="vZVSz"></a>
## 引言与范围
<a name="WxuTa"></a>
### 先决条件
FreeRTOS 是作为一组 C 源文件提供的，因此作为一名称职的 C 程序员是使用 FreeRTOS 的先决条件，因此本章假定读者熟悉以下概念：<br />​<br />

- 如何构建C项目，包括不同的编译和链接阶段。
- 栈和堆是什么.
- 标准 C 库 malloc() 和 free() 函数。



<a name="bBvJt"></a>
### 动态内存分配及其与 FreeRTOS 的相关性
从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：本书的后续章节将介绍内核对象，如任务，队列，信号量和事件组。 为了使 FreeRTOS 尽可能易于使用，这些内核对象在编译时不是分配的，而是在运行时动态分配的; 每次创建内核对象时，FreeRTOS 都会分配 RAM，并在每次删除内核对象时释放 RAM。 此策略减少了设计和计划工作，简化了 API，并最大限度地减少了 RAM 占用空间。<br />本章讨论动态内存分配。 动态内存分配是一种 C 编程概念，而不是 FreeRTOS 或多任务特有的概念。 它与 FreeRTOS 相关，因为内核对象是动态分配的，而通用编译器提供的动态内存分配方案并不总是适用于实时应用程序。<br />可以使用标准 C 库中的 malloc() 和 free() 函数分配内存，但是由于以下一个或多个原因，它们可能不合适：

- 它们在小型嵌入式系统中并不总是可用。
- 它们的实现可能相对较大，占用了宝贵的代码空间。
- 它们很少是线程安全的。
- 它们不是确定性的; 执行功能所花费的时间因调用而异。
- 它们可能会遭受碎片的折磨。
- 它们会使链接器配置复杂化。
- 如果允许堆空间增长到其他变量使用的内存，它们可能是难以调试出来的错误来源。



<a name="fCceD"></a>
### 动态内存分配的选项
从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：早期版本的FreeRTOS 使用内存池分配方案，从而在编译时预先分配不同大小的内存块池，然后由内存分配功能。 虽然这是在实时系统中使用的常见方案，但它被证明是许多支持请求的来源，主要是因为它无法有效地使用 RAM 以使其适用于非常小的嵌入式系统，因此该方案被放弃。<br />FreeRTOS 现在将内存分配视为可移植层的一部分（而不是核心代码库的一部分）。 这是因为不同的嵌入式系统具有不同的动态存储器分配和时序要求，因此单个动态存储器分配算法将仅适用于应用的子集。 此外，从核心代码库中删除动态内存分配使应用程序编写者能够在适当时提供自己的特定实现。<br />当 FreeRTOS 需要 RAM 时，它调用 pvPortMalloc()，而不是调用 malloc()。 当释放 RAM 时，内核调用 vPortFree() 而不是调用 free()。 pvPortMalloc() 与标准 C 库 malloc() 函数具有相同的原型，而 vPortFree() 与标准 C 库 free() 函数具有相同的原型。<br />pvPortMalloc() 和 vPortFree() 是公共函数，因此也可以从应用程序代码中调用它们。<br />从 FreeRTOS V9.0.0 开始，内核对象可以在编译时静态分配，也可以在运行时动态分配：FreeRTOS 带有pvPortMalloc() 和 vPortFree() 的五个示例实现，所有这些都在本章中记录。FreeRTOS 应用程序可以使用其中一个示例实现，或提供自己的。<br />这五个示例分别在 heap_1.c，heap_2.c，heap_3.c，heap_4.c 和 heap_5.c 源文件中定义，所有这些文件都位于 FreeRTOS/Source/portable/MemMang 目录中。
<a name="Ih5cf"></a>
### 范围
本章的目的是让读者更好地理解：<br />​<br />

- 什么时候 FreeRTOS 分配RAM。
- FreeRTOS 提供的 5 个内存分配方案示例。
- 选择哪种内存分配方案。

---

<a name="Itcw8"></a>
## 内存分配方案示例
从 FreeRTOS V9.0.0 可以完全静态分配 FreeRTOS 应用程序，无需包含堆内存管理器。
<a name="td7cJ"></a>
### Heap_1
小型专用嵌入式系统通常只在调度程序启动之前创建任务和其他内核对象。 在这种情况下，内存仅在应用程序开始执行任何实时功能之前由内核动态分配，并且内存在应用程序的生命周期内保持分配。 这意味着所选择的分配方案不必考虑任何更复杂的内存分配问题，例如确定性和分段，而只需考虑代码大小和简单性等属性。<br />heap_1.c 实现了 pvPortMalloc() 的一个非常基本的版本，并没有实现 vPortFree()。 永远不会删除任务或其他内核对象的应用程序可能会使用 heap_1。<br />一些禁止使用动态内存分配的商业关键系统和安全关键系统也可能使用 heap_1。由于与非确定性、内存碎片和失败的分配相关的不确定性，关键系统通常禁止动态内存分配，但是heap_1总是确定性的，并且不能分割内存。<br />heap_1 分配方案将一个简单数组细分为更小的块，因为调用了 pvPortMalloc() 。 该数组称为FreeRTOS heap。<br />数组的总大小（以字节为单位）由 FreeRTOSConfig.h 中的 configTOTAL_HEAP_SIZE 定义设置。 以这种方式定义大型数组可能会使应用程序看起来消耗大量 RAM ，甚至在从数组分配任何内存之前。<br />每个创建的任务都需要一个任务控制块（TCB）和一个从堆中分配的栈。 图 5 演示了在创建任务时 heap_1 如何细分简单数组。<br />参考图 5：

- A 显示了在创建任何任务之前的数组：整个数组是空闲的。
- B 显示了在创建一个任务之后的数组。
- C 显示了子啊船舰了三个任务之后的数组。


<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600462-f47eedd8-7502-46fd-837d-bb8435e9b9a4.png#clientId=u80d8bf1b-4228-4&from=paste&height=570&id=uc44c0ee1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=695&originWidth=906&originalType=url&ratio=1&size=63156&status=done&style=none&taskId=u26278b0b-9939-44f6-aaa6-fcb9fecd47d&width=743)<br />图5. 每次创建任务时从 heap_1 阵列分配的 RAM
<a name="IOcFj"></a>
### Heap_2
heap_2 保留在 FreeRTOS 发行版中以实现向后兼容，但不建议将其用于新设计。 请优先考虑使用 heap_4 而不是heap_2，因为 heap_4 提供了增强的功能。<br />heap_2.c 也可以通过细分由 configTOTAL_HEAP_SIZE 标注的数组来工作。 它使用最合适的算法来分配内存，与 heap_1 不同，它允许释放内存。 同样，数组是静态声明的，因此即使在分配了阵列中的任何内存之前，也会使应用程序看起来消耗大量 RAM。<br />最合适的算法确保 pvPortMalloc() 使用与所请求的字节数最接近的空闲内存块。 例如，考虑场景：

- 堆包含三个可用内存块，分别为 5 个字节，25 个字节和 100 个字节。
- 调用 pvPortMalloc() 来请求 20 个字节的 RAM。

所请求的字节数适合的最小空闲 RAM 块是 25 字节块，因此 pvPortMalloc() 将 25 字节块拆分为一个 20 字节的块和一个 5 字节的块，然后返回指针 20 字节的块。 新的 5 字节块仍可用于将来调用 pvPortMalloc()。<br />与 heap_4 不同，heap_2 不会将相邻的空闲块组合成一个更大的块，因此更容易出现碎片。 但是，如果分配并随后释放的块总是大小相同，则碎片不是问题。 heap_2 适用于重复创建和删除任务的应用程序，前提是分配给创建的任务的堆栈大小不会更改。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600757-8b0c9bed-ef35-4132-917b-1b2b2610d5c6.png#clientId=u80d8bf1b-4228-4&from=paste&id=ud0513f9f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=730&originWidth=765&originalType=url&ratio=1&size=73993&status=done&style=none&taskId=uf75f12ee-f076-41f8-9a36-39fcb5ffc8b)<br />图 6. 在创建和删除任务时，正在分配和释放来自 heap_2 数组的 RAM<br />图 6 展示了创建，删除然后再创建任务时最佳拟合算法的工作原理。 参考图6：<br />​<br />

1. A 显示了创建了三个任务之后的数组。一个大的自由块仍在数组顶部。
1. B 显示了一个任务被删除之后的数组。数组顶部的大型自由块仍然存在。 现在还有两个较小的空闲块，这是先前已分配给 TCB 和栈的任务，而现在已被删除。
1. C 显示了在创建另一个任务后的情况。 创建任务导致两次调用 pvPortMalloc()，一次分配新 TCB，另一次分配任务堆栈。使用 xTaskCreate() API 函数创建任务，如 3.4 节所述。 对 pvPortMalloc() 的调用在 xTaskCreate()内部发生。每个 TCB 的大小完全相同，因此最合适的算法确保先前分配给已删除任务的 TCB 的 RAM 块被重用以分配新任务的 TCB。分配给新创建的任务的堆栈大小与分配给先前删除的相同，因此最合适的算法确保先前分配给已删除任务堆栈的 RAM 块被重用以分配新任务的堆栈。数组顶部较大的未分配块保持不变。

heap_2 不是确定性的，但比 malloc() 和 free() 的大多数标准库实现更快。
<a name="TV1fR"></a>
### Heap_3
heap_3.c 使用标准库 malloc() 和 free() 函数，因此堆的大小由链接器配置定义，而 configTOTAL_HEAP_SIZE 设置没有影响。heap_3 使 malloc() 和 free() 线程安全暂时挂起 FreeRTOS 调度程序。 线程安全和调度程序暂停都是第 7 章资源管理中介绍的主题。
<a name="E28Zv"></a>
### Heap_4
与 heap_1 和 heap_2 类似，通过将数组细分为较小的块来实现 heap_4。 和以前一样，数组是静态声明的，并且由 configTOTAL_HEAP_SIZE 标注，因此即使在实际从数组中分配任何内存之前，也会使应用程序看起来消耗大量 RAM。<br />heap_4 使用首次拟合算法来分配内存。 与 heap_2 不同，heap_4 将相邻的空闲内存块（合并）组合成一个更大的块，从而最大限度地降低了内存碎片的风险。<br />第一适合算法确保 pvPortMalloc()使用足够大的第一个空闲内存块来保存请求的字节数。 例如，考虑以下情况：

- 堆包含三个可用内存块，它们按照它们在数组中出现的顺序分别为 5 个字节，200 个字节和 100 个字节。
- 调用 pvPortMalloc() 来请求 20 个字节的 RAM。

请求的字节数适合的第一个RAM块是 200 字节块，因此 pvPortMalloc() 将 200 字节块拆分为一个 20 字节的块，以及一个 180 字节的块，然后返回指向 20 的字节块。 新的 180 字节块仍可用于将来调用 pvPortMalloc()。<br />heap_4 将相邻空闲块的（合并）组合成一个更大的块，最大限度地降低了碎片的风险，并使其适用于重复分配和释放不同大小的 RAM 块的应用程序。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600952-bda5943d-c3a2-410d-823b-9b6dbe693782.png#clientId=u80d8bf1b-4228-4&from=paste&id=u6d974dd8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=597&originWidth=1338&originalType=url&ratio=1&size=115311&status=done&style=none&taskId=u429d48e2-5bfc-466b-8d15-028de5f1fa7)<br />图 7. 正在分配并从 heap_4 数组中释放的 RAM<br />图 7 演示了如何分配和释放具有内存合并工作的 heap_4 首次拟合算法。 参考图 7：<br />​<br />

1. A 显示了创建三个任务后的数组。 一个大的自由块保留在数组的顶部。
1. B 显示了删除一个任务后的数组。巨大的自由块仍在数组顶部。还有一个空闲块，是先前已分配任务 TCB 和堆栈，现在已经删除。 请注意，与演示 heap_2 时不同，删除 TCB 时释放的内存以及删除堆栈时释放的内存不会保留为两个单独的空闲块，而是组合起来创建一个更大的单个空闲块。
1. C 显示创建 FreeRTOS 队列后的情况。 队列是使用 xQueueCreate() API 函数创建的，该函数在 4.3 节中描述。 xQueueCreate() 调用 pvPortMalloc() 来分配队列使用的 RAM。 由于 heap_4 使用第一适合算法，pvPortMalloc() 将从第一个空闲 RAM 块中分配 RAM，该块足够大以容纳队列，如图 7 所示，当任务被删除时，RAM 被释放。队列不会全部消耗然而，空闲块中的 RAM，因此块被分成两部分，未使用的部分仍然可用于将来调用 pvPortMalloc()。
1. D 显示了直接从应用程序代码调用 pvPortMalloc() 之后的情况，而不是通过 FreeRTOS API 函数的间接调用。 用户分配的块足够小，可以放入第一个空闲块，即分配给队列的内存和分配给后续 TCB 的内存之间的块。 删除任务时释放的内存现在已分成三个独立的块; 第一个块保存队列，第二个块保存用户分配的内存，第三个块保持空闲。
1. E 显示队列被删除后的情况，这会自动释放已分配给已删除队列的内存。现在，用户分配块的任一侧都有可用内存。
1. F 显示用户分配的内存已被释放后的情况。用户分配的块使用的内存已与任意一侧的空闲内存组合，创建一个更大的单个空闲块。


<br />heap_4 不是具有确定性的，但比 malloc() 和 free() 的大多数标准库实现更快。
<a name="vCG7m"></a>
### 设置 Heap_4 使用的数组的起始地址
本节包含高级信息。若只是为了使用 heap_4，没有必要阅读或理解本节。<br />​

有时，应用程序编写者需要将 heap_4 使用的数组放在特定的内存地址。例如，FreeRTOS 任务使用的堆栈是从堆中分配的，因此可能需要确保堆位于快速的内部存储器中， 而不是缓慢的外部存储器。<br />​

默认情况下，heap_4 使用的数组在 heap_4.c 源文件中声明，其起始地址由链接器自动设置。 但是，如果在 FreeRTOSConfig.h 中将 configAPPLICATION_ALLOCATED_HEAP 编译时配置常量设置为 1，则必须由使用 FreeRTOS 的应用程序声明该数组。 如果数组声明为应用程序的一部分，则应用程序的编写者可以设置其起始地址。<br />​

如果在 FreeRTOSConfig.h 中将 configAPPLICATION_ALLOCATED_HEAP 设置为 1，则必须在其中一个应用程序的源文件中声明一个名为 ucHeap 的 uint8_t 数组，其大小为 configTOTAL_HEAP_SIZE 设置。<br />​

将变量放在特定内存地址所需的语法取决于所使用的编译器，因此请参阅编译器的文档。 两个编译器的示例如下：<br />​<br />

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
<a name="rZ8vP"></a>
### Heap_5
heap_5 用于分配和释放内存的算法与 heap_4 使用的算法相同。 与 heap_4 不同的是，heap_5 不限于从单个静态声明的数组中分配内存; heap_5 可以从多个独立的内存空间分配内存。 当运行 FreeRTOS 的系统提供的 RAM 在系统的内存映射中不显示为单个连续（无空间）块时，heap_5 非常有用。<br />​

在编写本文时，heap_5 是唯一提供的内存分配方案，必须在调用 pvPortMalloc() 之前显式初始化。 使用 vPortDefineHeapRegions() API函数初始化 Heap_5。 当使用 heap_5 时，必须先调用 vPortDefineHeapRegions()，然后才能创建任何内核对象（任务，队列，信号量等）。
<a name="X8O6E"></a>
### vPortDefineHeapRegions() API函数
vPortDefineHeapRegions() 用于指定每个单独的内存区域的起始地址和大小，它们共同构成 heap_5 使用的总内存。
```csharp
void vPortDefineHeapRegions(const HeapRegion_t *const pxHeapRegions );
```
清单 4. vPortDefineHeapRegions() API 函数原型<br />每个单独的存储区域由 HeapRegion_t 类型的结构描述。 所有可用内存区域的描述都作为 HeapRegion_t 结构数组传递给 vPortDefineHeapRegions()。<br />​<br />
```verilog
typedef struct HeapRegion
{
    /* 将成为堆一部分的内存块的起始地址。 */
    uint8_t *pucStartAddress;

    /* 内存块的大小（字节）。 */
    size_t xSizeInBytes;
} HeapRegion_t;
```
清单 5. HeapRegion_t 结构<br />表 5. vPortDefineHeapRegions() 参数

| 参数名称/返回值 | 描述 |
| --- | --- |

| ​

​

​

​

​

pxHeapRegions | 指向 HeapRegion_t 结构数组开头的指针。 数组中的每个结构都描述了使用 heap_5 时将成为堆的一部分的内存区域的起始地址和长度。<br />数组中的 HeapRegion_t 结构必须按起始地址排序; 描述具有最低起始地址的存储区域的 HeapRegion_t 结构必须是数组中的第一个结构，并且描述具有最高起始地址的存储区域的 HeapRegion_t 结构必须是数组中的最后一个结构。<br />数组的末尾由 HeapRegion_t 结构标记，该结构的 pucStartAddress 成员设置为 NULL。 |
| --- | --- |

![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635933600926-a8612b64-b6cf-48eb-85cf-c66cbe5404b0.png#clientId=u80d8bf1b-4228-4&from=paste&id=u2c20fabf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=635&originWidth=1132&originalType=url&ratio=1&size=114233&status=done&style=none&taskId=u5ff55469-3aa1-45c2-a8ba-a5bdb097906)<br />图 8. 内存映射<br />清单 6 显示了一个 HeapRegion_t 结构数组，它们共同描述了三个 RAM 块。<br />​<br />
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
​

清单 6. 一个 HeapRegion_t 结构数组，它们共同描述了 RAM 的 3 个区域<br />​

虽然清单 6 正确描述了 RAM，但它没有演示一个可用的示例，因为它将所有 RAM 分配给堆，而没有 RAM 可供其他变量使用。<br />​

构建项目时，构建过程的链接阶段会为每个变量分配一个 RAM 地址。 可供链接器使用的 RAM 通常由链接器配置文件（如链接描述文件）描述。 在图 8 B 中，假设链接器脚本包含有关 RAM1 的信息，但不包括有关 RAM2 或 RAM3 的信息。 因此，链接器在 RAM1 中放置了变量，只留下了地址为 0x0001nnnn 的 RAM1 部分可供 heap_5 使用。 0x0001nnnn 的实际值将取决于所链接的应用程序中包含的所有变量的总大小。 链接器使所有 RAM2 和所有 RAM3 都未使用，使得整个 RAM2 和整个 RAM3 可供 heap_5 使用。<br />如果使用清单 6 中所示的代码，则分配给地址 0x0001nnnn下面的 heap_5 的 RAM 将与用于保存变量的 RAM 重叠。 为避免这种情况，xHeapRegions[] 数组中的第一个 HeapRegion_t 结构可以使用起始地址0x0001nnnn，而不是起始地址 0x00010000。 但是，这不是推荐的解决方案，因为：<br />​<br />

1. 起始地址可能不容易确定。
1. 链接器使用的 RAM 量可能会在将来的版本中发生变化，需要更新 HeapRegion_t结构中使用的起始地址。
1. 构建工具未知， 因此，如果链接器使用的 RAM 和 heap_5 使用的 RAM 重叠，则无法警告应用程序编写者。


<br />清单 7 演示了一个更方便和可维护的示例。 它声明了一个名为 ucHeap 的数组。 ucHeap 是一个普通变量，因此它成为链接器分配给 RAM1 的数据的一部分。 xHeapRegions 数组中的第一个 HeapRegion_t 结构描述了 ucHeap 的起始地址和大小，因此 ucHeap 成为 heap_5 管理的内存的一部分。 可以增加ucHeap 的大小，直到链接器使用的 RAM 消耗所有 RAM1，如图 8 C 所示。<br />​<br />
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

<br />清单 7. 一个 HeapRegion_t 结构数组，描述所有 RAM2，所有 RAM3，但只是 RAM1 的一部分<br />​

清单7中演示的技术的优点包括：<br />​<br />

1. 没有必要使用硬编码的起始地址。
1. ​HeapRegion_t 结构中使用的地址将由链接器自动设置，因此即使链接器使用的 RAM 量在将来的构建中发生更改，也始终是正确的。
1. 分配给 heap_5 的 RAM 不可能与链接器放入 RAM1 的数据重叠。
1. 如果 ucHeap 太大，应用程序将不会链接。

---

<a name="LAyjU"></a>
## 堆相关的实用函数
<a name="YR8RS"></a>
### xPortGetFreeHeapSize() API函数
xPortGetFreeHeapSize() API 函数在调用函数时返回堆中的空闲字节数。它可用于优化堆大小。 例如，如果 xPortGetFreeHeapSize() 在创建了所有内核对象后返回 2000，那么 configTOTAL_HEAP_SIZE 的值可以减少 2000。<br />​

使用 heap_3 时，xPortGetFreeHeapSize()不可用。
```dart
size_t xPortGetFreeHeapSize( void );
```
清单 8. xPortGetFreeHeapSize() API 函数原型<br />​

表 6. xPortGetFreeHeapSize() 返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| 返回值 | 调用 xPortGetFreeHeapSize() 时在堆中保持未分配的字节数。 |

<a name="lVkjQ"></a>
### xPortGetMinimumEverFreeHeapSize() API函数
xPortGetMinimumEverFreeHeapSize() API 函数返回自 FreeRTOS 应用程序开始执行以来堆中曾存在的最小未分配字节数。<br />xPortGetMinimumEverFreeHeapSize() 返回的值表明应用程序已经接近耗尽堆空间。例如，如果 xPortGetMinimumEverFreeHeapSize() 返回200，那么，在应用程序开始执行后的某个时候，它会在耗尽堆空间的 200 字节之内。<br />xPortGetMinimumEverFreeHeapSize() 仅在使用 heap_4 或 heap_5 时可用。
```dart
size_t xPortGetMinimumEverFreeHeapSize( void );
```
清单 9. xPortGetMinimumEverFreeHeapSize() API 函数原型<br />​

表 7. xPortGetMinimumEverFreeHeapSize()返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| 返回值 | 自 FreeRTOS 应用程序开始执行以来堆中已存在的最小未分配字节数。 |

<a name="JIfiu"></a>
### malloc 失败的钩子函数
可以直接从应用程序代码调用 pvPortMalloc()。 每次创建内核对象时，它也会在 FreeRTOS 源文件中调用。内核对象的示例包括任务，队列，信号量和事件组，所有这些都将在本书后面的章节中介绍。<br />​

就像标准库 malloc() 函数一样，如果 pvPortMalloc() 由于所请求大小的块不存在而无法返回 RAM 块，那么它将返回 NULL。 如果由于应用程序编写器正在创建一个内核对象而执行 pvPortMalloc()，并且对 pvPortMalloc() 的调用返回 NULL，则内核对象不会被创建。<br />​

如果对 pvPortMalloc() 的调用返回 NULL，则可以将所有示例堆配置方案配置为调用钩子（或回调）函数。<br />​

如果在 FreeRTOSConfig.h 中将 configUSE_MALLOC_FAILED_HOOK 设置为 1，则应用程序必须提供 malloc 失败的钩子函数，该函数具有由清单 10 显示的名称和原型。该函数可以以适合应用程序的任何方式实现。
```dart
void vApplicationMallocFailedHook( void );
```
清单 10. malloc 失败的钩子函数名和原型。

---

<a name="BetVo"></a>
# **任务管理**

---

<a name="X092w"></a>
## 引言与范围
<a name="G3hdm"></a>
### 范围
本章旨在让读者更好地理解：<br />​<br />

- FreeRTOS 如何为应用程序中的每个任务分配处理时间。
- FreeRTOS 如何选择在任何给定时间执行哪个任务。
- 每个任务的相对优先级如何影响系统行为。 
- 任何可以存在的任务。

​

读者也应该很好地理解：<br />​<br />

- 如何实现任务。
- 如何创建一个或多个任务实例。
- 如何使用任务参数。
- 如何更改已被创建任务的优先级。
- 如何删除任务。
- 如何使用任务实现定期处理（软件计时器将在后面的章节中讨论）。
- 空闲任务何时执行以及如何使用。

​

本章介绍的概念对于理解如何使用 FreeRTOS 以及 FreeRTOS 应用程序的行为至关重要。因此，这是书中最详细的一章。

---

<a name="NKZMR"></a>
## 任务函数
任务为 C 函数实现。 它们唯一的特殊之处是它们的原型，它必须返回 void 并接受一个 void 指针参数。 清单 11 展示了原型。
```dart
void ATaskFunction( void *pvParameters );
```
清单 11. 任务函数原型<br />​

每个任务本身都是一个小程序。 它有一个入口点，通常会在无限循环内永远运行，并且不会退出。 清单 12中显示了典型任务的结构。<br />​

FreeRTOS 任务不允许以任何方式从其实现的函数返回，即它们不能包含 return 语句，并且不能允许在函数结束之后执行。 如果不再需要任务，则应明确删除该任务。 这在清单 12 中也有说明。<br />单个任务函数，可被创建成任意数量的任务，每个创建的任务都是一个单独的执行实例，具有自己的堆栈以及在任务本身内定义的任何自动（堆栈）变量，都有自己的副本。<br />​<br />
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

---

<a name="O50j9"></a>
## 顶层任务状态
应用程序可以包含许多任务。 如果运行应用程序的处理器是单核的，那么在任何给定时间只能执行一个任务。 这意味着任务可以存在两种状态：运行和未运行。 首先考虑这种简单化的模型，但请记住，这是一种过度简化。 在本章的后面，显示 “未运行” 状态实际上包含许多子状态。<br />​

当任务处于运行状态时，则处理器正在执行该任务的代码。 当任务处于 “未运行” 状态时，任务处于休眠状态，其状态已准备好，以便在下次调度程序确定应进入 “运行” 状态时继续执行。 当一个任务恢复执行时，它会从它上次离开运行状态之前执行的指令执行。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635995518702-e2ed01de-83a9-45a8-a5df-fac1e30b3ba3.png#clientId=udb212963-d539-4&from=paste&id=ua4d0fb77&margin=%5Bobject%20Object%5D&name=image.png&originHeight=549&originWidth=1075&originalType=url&ratio=1&size=78852&status=done&style=none&taskId=u6dd71846-1a85-418f-9f56-39d1339aeb6)<br />图 9. 顶层任务状态和转换<br />从 “未运行” 状态转换为 “运行” 状态的任务被称为 “切入” 或 “换入”。 相反，从 “运行” 状态转换到 “未运行” 状态的任务据说已被 “切出” 或 “换出”。 FreeRTOS 的调度程序是唯一可以切换任务的实体。

---

<a name="HLOUs"></a>
## 创建任务
<a name="irKvW"></a>
### xTaskCreate() API 函数
FreeRTOS V9.0.0 还包含 xTaskCreateStatic() 函数，它分配了在编译时静态创建任务所需的内存：任务是使用 FreeRTOS xTaskCreate() API 函数创建的。这可能是所有 API 函数中最复杂的一个，因此很不幸，它是第一个遇到的函数，但是必须首先掌握任务，因为它们是多任务系统中最基本的组件。本书附带的所有示例都使用了 xTaskCreate() 函数，因此有很多示例可供参考。<br />​

第 1.5 节 “数据类型和编程样式指南” 描述了所使用的数据类型和命名约定。
```dart
BaseType_t xTaskCreate( TaskFunction_t pvTaskCode, 
                        const char* const pcName, 
                        uint16_t usStackDepth, 
                        void *pvParameters, 
                        UBaseType_t uxPriority, 
                        TaskHandle_t *pxCreatedTask );
```
清单 13. xTaskCreate() API 函数原型<br />表8. xTaskCreate() 参数和返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| pvTaskCode | 任务仅仅是从不退出的C函数，因此通常作为无限循环来实现。PvTaskCode参数只是指向实现任务的函数的指针(实际上，只是函数的名称)。 |
| ​

​

​

pcName | 任务的描述性名称。FreeRTOS 不会以任何方式使用它。它纯粹是作为调试辅助工具而包含的。通过人类可读的名称标识任务要比尝试通过其句柄标识任务简单得多。<br /> 应用程序定义的常量configMAX_TASK_NAME_LEN定义了任务名称可以包含的最大长度包括空终止符。提供长度超过此最大值的字符串将导致字符串被自动截断。 |
| ​

​

​

​

​

​

​

​

usStackDepth | 每个任务都有自己的唯一堆栈，该堆栈在创建任务时由内核分配给任务。 usStackDepth值告诉内核堆栈的大小。<br />该值指定堆栈可以容纳的字数，而不是字节数。 例如，如果堆栈是 32 位宽并且 usStackDepth 作为 100 传入，则将分配 400 字节的堆栈空间（100 * 4 字节）。 堆栈深度乘以堆栈宽度不得超过 uint16_t 类型的变量中可包含的最大值。 <br />空闲任务使用的堆栈大小由应用程序定义的常量 configMINIMAL_STACK_SIZE定义。在 FreeRTOS 演示应用程序中为正在使用的处理器体系结构分配给此常量的值是建议的最小任务。如果您的任务使用大量堆栈空间，则必须分配更大的值。 <br />没有简单的方法来确定任务所需的堆栈空间。可以计算，但是大多数用户只需分配他们认为合理的值，然后使用FreeRTOS 提供的功能来确保分配的空间确实足够，并且RAM 不会被浪费。 第 12.3 节“堆栈溢出” 包含有关如何查询任务实际使用的最大堆栈空间的信息。 |
| ​

pvParameters | 任务函数接受指向 void（void *）的类型指针的参数。分配给 pvParameters 的值是传递给任务的值。本书中的一些示例演示了如何使用该参数。 |
| ​

​

​

uxPriority | 定义任务执行的优先级。可以将优先级从 0（最低优先级）分配给（configMAX_PRIORITIES - 1），这是最高优先级。 configMAX_PRIORITIES 是用户定义的常量，在3.5 节中描述。 <br />传递上面的 uxPriority 值（configMAX_PRIORITIES - 1）将导致分配给任务的优先级被静默限制为最大合法值。 |
| ​

​

pxCreatedTask | pxCreatedTask 可用于传递正在创建的任务的句柄。然后，此句柄可用于引用 API 调用中的任务，例如，更改任务优先级或删除任务。 <br />如果您的应用程序没有使用任务句柄，那么 pxCreatedTask 可以设置为 NULL。 |
| ​

​

返回值 | 有两种可能的返回值：<br />pdPASS：这表明该任务已成功创建。<br />pdFAIL：这表明该任务尚未创建，因为 FreeRTOS 可用的堆内存不足，无法分配足够的 RAM 来保存任务数据结构和堆栈。第 2 章提供了有关堆内存管理的更多信息。 |

<a name="rgh0C"></a>
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
清单 15. 实现示例 1 中使用的第二个任务<br />​

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
清单 16. 启动示例 1 任务<br />​

执行该示例将生成图 10 中所示的输出。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635996495870-7a09af02-4122-4001-ab11-55e8dd15e680.png#clientId=udb212963-d539-4&from=paste&id=u47bf1be0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=657&originWidth=1319&originalType=url&ratio=1&size=307202&status=done&style=none&taskId=u94bd7315-6046-4d0d-adde-3af8fd44a8c)<br />图 10. 执行示例 1 时生成的输出<br />图 10 显示了两个似乎同时执行的任务; 但是，由于任务正在同一个处理器核上执行，但情况并非如此。 实际上，这两项任务都在迅速进入并退出运行状态。 这两个任务都以相同的优先级运行，因此在同一个处理器上运行共享时间。 实际执行模式如图 11 所示。<br />​

图 11 底部的箭头表示从时间 t1 开始经过的时间。 彩色线表示在每个时间点正在执行哪个任务，例如，Task1 在时间 t1 和时间 t2 之间执行。<br />​

任何时候只有一个任务可以处于运行状态。 因此，当一个任务进入运行状态（任务被切入）时，另一个任务进入非运行状态（任务被切出）。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635996495578-50f6ccc4-f552-4309-8f1c-3e3845833ba9.png#clientId=udb212963-d539-4&from=paste&id=u9fd4ef35&margin=%5Bobject%20Object%5D&name=image.png&originHeight=748&originWidth=1094&originalType=url&ratio=1&size=96917&status=done&style=none&taskId=ued2c029d-a2e8-4d6e-9f27-673149efc87)<br />图11.两个示例 1 任务的实际执行模式<br />示例 1 在启动调度程序之前，从 main() 内创建了两个任务。 从另一个任务中创建任务也是可能的。 例如，可以从 Task1 中创建 Task2，如清单 17 所示。
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
<a name="neC6C"></a>
### 示例 2. 使用任务参数
在示例 1 中创建的两个任务是字面上的，它们之间的唯一区别是它们打印出来的文本字符串。 相反，可以通过创建单个任务实现的两个实例来删除此重复。 然后可以使用任务参数将要打印出的字符串传递给每个任务。<br />​

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
清单 18. 用于在示例 2 中创建两个任务的单个任务函数<br />​

即使现在只有一个任务实现（vTaskFunction），也可以创建多个已定义任务的实例。每个创建的实例将在 FreeRTOS 调度程序的控制下独立执行。<br />​

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
清单 19. 示例 2 说使用的 main() 函数<br />​

示例 2 的输出完全与图 10 中的示例 1 所示的输出完全相同。

---

<a name="MTPdv"></a>
## 任务优先级

<br />xTaskCreate() API 函数的 uxPriority 参数为正在创建的任务分配初始优先级。 使用vTaskPrioritySet() API 函数可以在启动调度程序后，更改优先级。<br />​

可用优先级的最大数量由应用程序设置，在 FreeRTOSConfig.h 中定义的 configMAX_PRIORITIES 编译时配置常量。 低数字优先级值表示低优先级任务，优先级 0 是可能的最低优先级。 因此，可用优先级的范围是 0 到（configMAX_PRIORITIES - 1）。任意数量的任务可以共享相同的优先级，确保最大的设计灵活性。<br />​

FreeRTOS 调度程序可以使用以下两种方法之一来确定哪个任务将处于运行状态。 可以设置 configMAX_PRIORITIES 的最大值取决于使用的方法：<br />​<br />

1. 通用方法：通用方法在 C 中实现，可以与所有 FreeRTOS 架构移植一起使用。使用通用方法时，FreeRTOS 不会限制 configMAX_PRIORITIES 可以设置的最大值。 但是，始终建议将 configMAX_PRIORITIES 值保持在最小值，因为值越高，消耗的 RAM 越多，最坏情况下的执行时间就越长。如果在 FreeRTOSConfig.h 中将 configUSE_PORT_OPTIMISED_TASK_SELECTION 设置为 0，或者如果未定义 configUSE_PORT_OPTIMISED_TASK_SELECTION，或者如果通用方法是为使用的 FreeRTOS 移植提供的唯一方法，则将使用通用方法。
1. 架构优化方法：体系结构优化方法使用少量汇编代码，并且比通用方法更快。 configMAX_PRIORITIES 设置不会影响最坏情况执行时间。如果使用体系结构优化方法，那么configMAX_PRIORITIES 不能大于 32。与通用方法一样，建议将 configMAX_PRIORITIES 保持在必要的最小值，因为它的值越高，消耗的 RAM 就越多。如果在 FreeRTOSConfig.h 中将 configUSE_PORT_OPTIMISED_TASK_SELECTION 设置为 1，则将使用体系结构优化方法。并非所有 FreeRTOS 移植都提供了架构优化方法。


<br />FreeRTOS 调度程序将始终确保能够运行的最高优先级任务是选择进入运行状态的任务。 如果能够运行多个具有相同优先级的任务，则调度程序将依次将每个任务转换为运行状态和退出运行状态。

---

<a name="QGgXa"></a>
## 时间测量和嘀嗒中断
第 3.12 节 “调度算法” 描述了一个名为 “时间切片” 的可选功能。 在目前给出的示例中使用了时间切片，它是在输出中观察到的行为。在示例中，两个任务都以相同的优先级创建，并且两个任务始终能够运行。 因此，每个任务为 “时间片” 执行，在时间片开始时进入运行状态，并在时间片结束时退出运行状态。 在图 11 中，t1 和 t2 之间的时间等于单个时间片。<br />​

为了能够选择要运行的下一个任务，调度程序本身必须在每个时间片的末尾执行。 称为 “滴答中断” 的周期性中断用于此目的。 时间片的长度由滴答中断频率有效设置，滴答中断频率由应用程序在 FreeRTOSConfig.h 中定义的 configTICK_RATE_HZ 编译时配置常量配置。 例如，如果 configTICK_RATE_HZ 设置为 100（Hz），则时间片将为 10 毫秒。 两次滴答中断之间的时间称为 “滴答期”。 一个时间片等于一个滴答时间段。<br />​

图 11 可以展开来显示调度程序本身在执行序列中的执行情况。图 12 显示了这一点，其中顶部一行显示了调度程序何时执行，细箭头显示了从任务到滴答中断的执行顺序，然后从滴答中断返回到另一个任务。<br />configTICK_RATE_HZ 的最佳值取决于正在开发的应用程序，但值通常为 100。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635997524402-12ef733f-1066-4c7d-a98d-cf8ea78a2593.png#clientId=udb212963-d539-4&from=paste&id=u0c83a66a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=827&originWidth=1027&originalType=url&ratio=1&size=94826&status=done&style=none&taskId=u26a16709-7027-4e84-9d03-87575594f86)<br />图 12. 执行序列展开以显示滴答中断执行<br />FreeRTOS API 调用总是以滴答周期的倍数指定时间，这通常被简单地称为滴答。pdMS_TO_TICKS() 宏将以毫秒为单位的时间转换为以节拍为单位的时间。可用的分辨率取决于所定义的滴答频率，如果滴答频率高于 1 KHz (configTICK RATE HZ 大于 1000)，则不能使用 pdMS_TO_TICKS()。清单 20 显示了如何使用 pdMS_TO_TICKS() 将指定为 200 毫秒的时间转换为以滴答为单位指定的等效时间。
```groovy
/* pdMS_TO_TICKS() 将以毫秒为单位的时间作为其唯一参数，并计算滴答周期中的等效时间。 
此示例显示 xTimeInTicks 设置为等于 200 毫秒的滴答周期数。 */
TickType_t xTimeInTicks = pdMS_TO_TICKS( 200 );
```
清单 20. 使用pdMS TO TICKS()宏将200毫秒转换为滴答周期中的等效时间<br />​<br />
:::tips
不建议在应用程序中直接以滴答为单位指定时间，而是使用 pdMS_TO_TICKS()宏来指定以毫秒为单位的时间，这样做可以确保如果滴答频率发生变化，应用程序中指定的时间不会发生变化。
:::
​

滴答计数值是调度器启动以来发生的滴答中断总数，假设滴答计数没有溢出。在指定延迟周期时，用户应用程序不必考虑溢出，因为时间一致性是由 FreeRTOS 在内部管理的。<br />​

第 3.12 节 “调度算法” 描述了影响调度程序何时选择要运行的新任务以及何时执行勾选中断的配置常量。<br />​<br />
<a name="VqZHF"></a>
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
清单 21. 以不同的优先级创建两个任务<br />​

示例 3 产生的输出如图 13 所示。 调度程序将始终选择能够运行的最高优先级任务。 任务 2 的优先级高于任务 1，并且始终能够运行；因此，任务 2 是唯一进入运行状态的任务。 若任务 1 从未进入运行状态，它永远不会打印出它的字符串。 任务 1 被称为任务 2 的处理时间 “缺乏”。<br />​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998116184-6714d06a-2dcc-4cef-8c0e-a5d6d5745263.png#clientId=udb212963-d539-4&from=paste&id=u87ba14de&margin=%5Bobject%20Object%5D&name=image.png&originHeight=656&originWidth=1317&originalType=url&ratio=1&size=316516&status=done&style=none&taskId=u18469aa9-66f7-42e0-a38d-2e52c2a570c)<br />图 13. 以不同的优先级运行两个任务<br />任务 2 总是能够运行，因为它永远不必等待任何事情 —— 它要么循环一个空循环，要么打印到终端。<br />​

图 14 显示了示例 3 的执行顺序。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998115879-33726541-8bb3-4b85-934b-c698858aaee9.png#clientId=udb212963-d539-4&from=paste&id=ubff0d44b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=613&originWidth=924&originalType=url&ratio=1&size=79480&status=done&style=none&taskId=u10ec9c25-131c-445d-b2b7-83a698cfb4e)<br />图 14. 当一个任务的优先级高于另一个任务时的执行模式

---

<a name="UHTTV"></a>
## 展开说 “非运行” 状态
到目前为止，创建的任务总是有执行处理，从来没有等待任何事情，因为他们永远不必等待任何事情，他们总是能够进入运行状态。 这种类型的 “连续处理” 任务的有用性有限，因为它们只能以最低优先级创建。 如果它们以任何其他优先级运行，它们将阻止具有较低优先级的任务一直运行。<br />​

为了使任务有用，必须重写它们以使其成为事件驱动的。 事件驱动的任务只有在触发它的事件发生后才能执行工作（处理），并且在该事件发生之前无法进入运行状态。 调度程序始终选择能够运行的最高优先级任务。 无法运行的高优先级任务意味着调度程序无法选择它们，而必须选择能够运行的优先级较低的任务。 因此，使用事件驱动任务意味着可以在不同的优先级上创建任务，而不需要最高优先级的任务占用所有较低优先级的任务的处理时间。
<a name="Za8Ek"></a>
### 阻塞状态
正在等待事件的任务被称为处于阻塞状态，这是未运行状态的子状态。<br />​

任务可以进入阻塞状态，等待两种不同类型的事件：<br />​<br />

1. 时间（与时间相关）事件，即该事件是延时期到期或达到绝对时间。 例如，任务可以进入阻塞状态以等待10毫秒再通过。
1. 同步事件，即该事件源自另一个任务或中断。 例如，任务可以进入阻塞状态以等待数据到达队列。 同步事件涉及广泛的事件类型。


<br />FreeRTOS 的队列，二进制信号量，计数信号量，互斥量，递归互斥体，事件组和直接到任务通知都可用于创建同步事件。 所有这些功能都将在本书的后续章节中介绍。<br />​

任务可以在阻塞时设置带有超时的同步事件，同时有效地阻塞两种类型的事件。 例如，任务可以选择等待最多 10 毫秒的数据到达队列。 如果数据在 10 毫秒内到达，或者在没有数据到达的情况下传递 10 毫秒，任务都将离开阻塞状态。<br />​<br />
<a name="zt4hb"></a>
### 挂起状态
“挂起” 也是非运行状态的子状态。 处于挂起状态的任务对调度程序时不可用的。 进入挂起状态的唯一方法是通过调用 vTaskSuspend() API函数，调用 vTaskResume() 或 xTaskResumeFromISR() API 函数时退出挂起的唯一出路。 大多数应用程序不使用挂起状态。<br />​<br />
<a name="qaLCH"></a>
### 就绪状态
处于 “未运行” 状态但未被阻塞或挂起的任务被称为处于 “就绪” 状态。 它们能够运行，因此 “准备好” 运行，但当前不处于 “运行” 状态。<br />​<br />
<a name="myVNL"></a>
### 完成状态转换图
图 15 展开了先前过度简化的状态图，以包括本节中描述的所有非运行状态的子状态。 到目前为止，在示例中创建的任务没有使用阻塞或挂起状态；它们只在就绪状态和运行状态之间转换，由图 15 中的粗线突出显示。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998355516-0d90c553-da06-4c6c-80b1-b80e537ddfdd.png#clientId=udb212963-d539-4&from=paste&id=u91242f3f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=844&originWidth=674&originalType=url&ratio=1&size=75206&status=done&style=none&taskId=u5a41c47b-6dd2-4b06-b5c6-056167a7ea2)<br />图 15. 完整的任务状态机<br />

<a name="KJKj7"></a>
### 示例 4. 使用 “阻止” 状态创建延迟
到目前为止，在示例中创建的所有任务都是 “周期性的”，它们已经延迟了一段时间并打印出它们的字符串，然后再次延迟，依此类推。使用空循环非常粗略地生成延迟，该任务有效地轮询递增循环计数器，直到它达到固定值。实施例 3 清楚地证明了该方法的缺点。优先级较高的任务在执行空循环时保持运行状态，“饿死” 任何处理时间的优先级较低的任务。<br />​

任何形式的轮询还有其他几个缺点，其中最重要的是它的效率低下。在轮询期间，该任务实际上没有任何工作要做，但它仍然使用最长的处理时间，因此浪费了处理器的资源。示例 4 通过调用 vTaskDelay() API 函数替换轮询空循环来纠正此行为，其原型如清单 22 所示。新的任务定义如清单 23 所示。请注意，vTaskDelay() API 函数是仅当 FreeRTOSConfig.h 中的 INCLUDE_vTaskDelay 设置为 1 时才可用。<br />​

vTaskDelay() 将调用任务置于阻塞状态，以获得固定数量的滴答中断。任务在处于阻塞状态时不使用任何处理时间，因此任务仅在实际工作时使用处理时间。
```groovy
void vTaskDelay( TickType_txTicksToDelay );
```
清单 22. vTaskDelay() 的函数原型<br />表 9. vTaskDelay() 参数

| 参数名称 | 描述 |
| --- | --- |
| ​

​

​

xTicksToDelay | 调用任务在转换回就绪状态之前将保持在阻塞状态的滴答中断数。<br />例如，如果一个名为 vTaskDelay(100) 的任务在滴答计数为 10,000 时立即进入阻塞状态，并保持阻塞状态，直到滴答计数达到 10,100。可以使用宏 pdMS_TO_TICKS() 将以毫秒为单位的时间转换为以 int 为单位的时间。例如，调用 vTaskDelay(pdMS_TO_TICKS(100)) 将导致调用任务保持阻塞状态 100 毫秒。 |

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
清单 23. 空循环延迟的示例任务的源代码已被对 vTaskDelay()的调用所取代<br />​

尽管这两项任务仍在不同的优先级下创建，但现在都将运行。 示例 4 的输出（如图 16 所示）确认了预期的行为。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975518-5fca50bb-fe3a-4557-b451-5a8f1f374495.png#clientId=udb212963-d539-4&from=paste&id=u55337d24&margin=%5Bobject%20Object%5D&name=image.png&originHeight=546&originWidth=1089&originalType=url&ratio=1&size=228546&status=done&style=none&taskId=ua1682f1d-578b-412d-9724-4c9aaceab0c)<br />图16.执行示例4时产生的输出 <br />图 17 中显示的执行顺序解释了为什么两个任务都运行，即使它们是以不同的优先级创建的。 为简单起见，省略了调度程序本身的执行。<br />​

在启动调度程序时自动创建空闲任务，以确保始终有至少一个任务能够运行 (至少一个处于就绪状态的任务)。第 3.8 节，空闲任务和空闲任务钩子，更详细地描述了空闲任务。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975820-725d737a-27b4-486c-89ab-c7c758bb8c67.png#clientId=udb212963-d539-4&from=paste&id=u3f5551b4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=668&originWidth=1034&originalType=url&ratio=1&size=135478&status=done&style=none&taskId=u3508ac77-44dd-4827-a8d9-a26dcb2d884)<br />图17. 任务使用vTaskDelay()代替NULL 循环时的执行顺序<br />这两个任务的实现发生了变化，没有改变它们的功能。将图 17 与图 12 进行比较，可以清楚地看到，实现此功能的方式要有效得多。<br />​

图 12 显示了当任务使用空循环来创建延迟时的执行模式——始终运行，因此使用它们之间可用处理器时间的百分之百。 图 17 显示了任务在整个延迟期间进入阻塞状态时的执行模式，因此只有在实际需要执行工作时才使用处理器时间（在这种情况下只是要打印出来的消息），并且结果只使用可用处理时间的一小部分。<br />​

在图 17 的场景中，每当任务离开阻塞状态时，它们会在重新进入阻塞状态之前执行一小部分时间。 大多数情况下，没有能够运行的应用程序任务（就绪状态下没有应用程序任务），因此，没有可以选择进入运行状态的应用程序任务。 在这种情况下，空闲任务将运行。 分配给空闲的处理时间量是系统中备用处理能力的度量。使用 RTOS 可以简单地通过允许应用程序完全由事件驱动来显着增加备用处理能力。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1635998975351-cc37ae23-819c-4968-9d2a-44f926bfd9b0.png#clientId=udb212963-d539-4&from=paste&id=ue3a8f6a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=714&originWidth=620&originalType=url&ratio=1&size=72845&status=done&style=none&taskId=u5d8ed366-70e0-4661-bd89-17ea59a07c7)<br />图 18.粗线表示例4中任务所执行的状态转换<br />图 18 中的粗线显示了示例 4 中的任务执行的转换，每个转换现在都转换到阻塞状态，然后返回到就绪状态。
<a name="KrFd3"></a>
### vTaskDelayUntil() API 函数
vTaskDelayUntil() 类似于 vTaskDelay()。 正如刚才演示的，vTaskDelay() 参数指定了调用vTaskDelay() 的任务与再次从阻塞状态过渡到正常状态的任务之间应该发生的滴动中断的数量。任务处于阻塞状态的时间长度由 vTaskDelay() 参数指定，但是任务离开阻塞状态的时间与调用 vTaskDelay() 的时间相关。<br />相反，vTaskDelayUntil() 的参数指定调用任务应该从阻塞状态移动到就绪状态的精确计时计数值。vTaskDelayUntil() 是在需要固定执行期间（您希望任务以固定频率定期执行）时应使用的 API 函数，因为调用任务被解除阻塞的时间是绝对的，而不是相对于调用函数时（与 vTaskDelay() 的情况一样）。
```groovy
void vTaskDelayUntil( TickType_t* pxPreviousWakeTime, TickType_t xTimeIncrement );
```
清单 24. vTaskDelayUntil() API 函数原型<br />表 10. vTaskDelayUntil() 的参数

| 参数名称 | 描述 |
| --- | --- |

| ​

​

​

​

​

pxPreviousWakeTime | 此参数的命名是基于 vTaskDelayUntil() 用于实现定期执行且具有固定频率的任务。 在这种情况下，pxPreviousWakeTime 保持任务最后一次离开阻塞状态的时间（被 '唤醒'）。 此时间用作参考点，用于计算任务下次离开阻塞状态的时间。<br />pxPreviousWakeTime 指向的变量在 vTaskDelayUntil()函数中自动更新；它通常不会被应用程序代码修改，但必须在第一次使用之前初始化为当前的滴答计数。 清单 25 演示了如何执行初始化。 |
| --- | --- |
| ​

​

xTimeIncrement | 此参数的命名也是基于 vTaskDelayUntil() 用于实现定期执行且具有固定频率的任务，频率由 xTimeIncrement 值设置。<br />xTimeIncrement 在 'ticks' 中指定。 pdMS_TO_TICKS()宏可用于将毫秒指定的时间转换为刻度中指定的时间。 |

<a name="khd2x"></a>
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
清单 25. 使用 vTaskDelayUntil() 实现示例任务<br />​

示例 5 产生的输出与图 16 中的示例 4 所示的输出完全相同。<br />​<br />
<a name="TlZTt"></a>
### 示例 6. 组合阻塞和非阻塞任务
前面的示例已经单独检查了轮询和阻塞任务的行为。 该示例通过在组合两个方案时演示执行序列来重新强制执行所述的预期系统行为，如下所述。<br />​<br />

1. 在优先级 1 创建两个任务。除了连续打印字符串之外，它们什么也不做。这些任务从不执行任何可能导致它们进入阻塞状态的 API 函数调用，因此总是处于就绪状态或运行状态。这种性质的任务称为连续处理任务，因为它们总是有工作要做(尽管在本例中是相当琐碎的工作)。连续处理任务的源如清单 26 所示。



2. 然后在优先级 2 创建第三个任务，因此高于其他两个任务的优先级。第三个任务也只是打印出一个字符串，但这次是周期性的，所以使用 vTaskDelayUntil() API 函数在每次打印迭代之间将自己置于阻塞状态。


<br />这个周期性任务的源代码如清单 27 所示。
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
清单 27. 示例 6 中使用的周期性任务<br />​

图 19 显示了示例 6 生成的输出，并通过图 20 中所示的执行顺序解释了观察到的行为。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636003548129-7c94f574-f7fc-4bbd-8a62-bab00cf22fa0.png#clientId=udb212963-d539-4&from=paste&id=u339239cb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=738&originWidth=1469&originalType=url&ratio=1&size=259619&status=done&style=none&taskId=ucb1b0763-9752-4437-8d6d-46475593251)<br />图 19. 执行示例 6 时生成的输出<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636003547829-6ab4f336-8e9c-465b-be34-3828bb8c9d77.png#clientId=udb212963-d539-4&from=paste&id=ud7294821&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1000&originWidth=1175&originalType=url&ratio=1&size=272294&status=done&style=none&taskId=u26f19121-4989-4479-ba6a-80eb704ba31)<br />图 20. 示例 6 的执行模式

---

<a name="pVWGv"></a>
## 空闲任务和空闲任务钩子
示例 4 中创建的任务将大部分时间用于阻塞状态。在此状态下，它们无法运行，因此调度程序也无法选择它们。<br />​

但必须始终至少有一个任务可以进入运行状态。 为确保这种情况，调用 vTaskStartScheduler() 时，调度程序会自动创建一个空闲任务。 空闲任务只是坐在一个循环中 —— 所以，就像原始第一个例子中的任务一样，它始终能够运行。<br />​

空闲任务具有可能的最低优先级（优先级为零），以确保它永远不会阻止更高优先级的应用程序任务进入运行状态 —— 尽管没有什么可以阻止应用程序设计者创建任务，从而共享空闲任务优先级，如果在FreeRTOSConfig.h 中，configIDLE_SHOULD_YIELD 编译时间配置常量可用于防止空闲任务消耗更高效地分配给应用程序任务的处理时间。 configIDLE_SHOULD_YIELD 在第 3.12 节 “调度算法” 中描述。<br />​

以最低优先级运行可确保只要优先级较高的任务进入就绪状态，空闲任务就会从运行状态转移出来。 这可以在图 17 中的时间点看到，其中空闲任务被立即换出以允许任务 2 在任务 2 离开被阻止状态的瞬间执行。 据说任务 2 已经抢占了空闲任务。 抢先自动发生，并且不知道任务被抢占。<br />​<br />
:::tips
如果应用程序使用 vTaskDelete() API 函数，那么空闲任务就不会浪费处理时间。这是因为空闲任务负责在删除任务之后清理内核资源。
:::
​<br />
<a name="lpxfi"></a>
### 空闲任务钩子函数
可以通过使用空闲钩子（或空闲回调）函数将应用程序特定功能直接添加到空闲任务中 —— 空闲任务循环每次迭代时由空闲任务自动调用的函数。<br />​

空闲任务钩子的常见用途包括：<br />​<br />

- 执行低优先级，后台或连续处理功能。
- 测量备用处理能力的数量。（当所有优先级较高的应用程序任务无法执行时，空闲任务将会运行；因此，测量分配给空闲任务的处理时间量可清楚地指示多少处理时间。）
- 将处理器置于低功耗模式，在没有应用处理的情况下提供简单且自动的省电方法（虽然使用这种方法所能达到的省电比使用第 10 章低功耗支持中描述的无时钟空闲模式所能达到的省电要少）。



<a name="TeBlZ"></a>
### 对空闲任务钩子函数实现的限制
空闲任务钩子函数必须遵循以下规则。<br />​<br />

1. 一个空闲的任务钩子函数决不能试图阻塞或挂起。
1. 如果应用程序使用了 vTaskDelete() API函数，那么空闲任务钩子必须总是在合理的时间段内返回给调用者。这是因为空闲任务负责在删除任务之后清理内核资源。如果空闲任务永久地保留在空闲钩子函数中，则无法进行清理。

​<br />
:::tips
以任何方式阻塞空闲任务都可能导致无法使用任何任务进入运行状态。
:::
​

空闲任务钩子函数必须具有清单 28 所示的名称和原型。
```groovy
void vApplicationIdleHook( void );
```
清单 28. 空闲任务钩子函数名称和原型
<a name="yIQ3W"></a>
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
清单 29. 一个非常简单的空闲钩子函数<br />​

config_IDLE_HOOK 必须在 FreeRTOSConfig.h 设置为 1 ，以便调用空闲钩子函数。<br />​

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
清单 30. 示例任务的源代码现在打印出 ulIdleCycleCount 值<br />示例 7 产生的输出如图 21 所示。 因此，在应用任务的每次迭代之间，空闲任务钩子函数被调用大约 400 万次（迭代次数取决于演示执行的硬件的速度）。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636004122657-a8018e6b-03d7-4323-9f8e-55587af3fb20.png#clientId=udb212963-d539-4&from=paste&id=u7d1971d3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=1458&originalType=url&ratio=1&size=468817&status=done&style=none&taskId=u494b6b63-0d3e-4b0d-987a-e15488a60e6)<br />图 21. 执行示例 7 时生成的输出

---

<a name="bAT1y"></a>
## 更改任务的优先级
<a name="l66Vl"></a>
### vTaskPrioritySet() API 函数
vTaskPrioritySet() API 函数可用于在调度程序启动后更改任何任务的优先级。请注意，当 FreeRTOSConfig.h 中的 INCLUDE_vTaskPrioritySet 设置为 1 时，vTaskPrioritySet() API 函数是可用的。
```groovy
void vTaskPrioritySet( TaskHandle_t pxTask, UBaseType_t uxNewPriority );
```
清单 31. vTaskPrioritySet() API 函数原型<br />表 11. vTaskPrioritySet() 参数

| 参数名称 | 描述 |
| --- | --- |
| ​

​

pxTask | 正在修改其优先级的任务的句柄（主题任务）—— 有关获取任务句柄的信息，请参阅xTaskCreate() API函数的 pxCreatedTask 参数。任务可以通过传递 NULL 代替有效的任务句柄来更改自己的优先级。 |
| ​

​

uxNewPriority | 要设置主题任务的优先级。 这自动限制为（configMAX_PRIORITIES - 1）的最大可用优先级，其中 configMAX_PRIORITIES 是在 FreeRTOSConfig.h 头文件中设置编译时间常量。 |

<a name="K59Yw"></a>
### uxTaskPriorityGet() API 函数
uxTaskPriorityGet() API 函数可用于查询任务的优先级。请注意，只有在 FreeRTOSConfig.h 中将INCLUDE_uxTaskPriorityGet 设置为 1 时，uxTaskPriorityGet() API 函数才可用。
```groovy
UBaseType_t uxTaskPriorityGet( TaskHandle_t pxTask );
```
清单 32. uxTaskPriorityGet() API 函数原型<br />表 12. uxTaskPriorityGet()参数和返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| ​

​

pxTask | 正在查询其优先级的任务的句柄（主题任务）— 请参阅 xTaskCreate()API 函数的pxCreatedTask参数，以获取有关获取任务句柄的信息。<br />任务可以通过传递 NULL 代替有效的任务句柄来查询自己的优先级。 |
| 返回值 | 当前分配给正在查询的任务的优先级。 |

<a name="rPVIX"></a>
### 示例 8. 改变任务优先级
调度程序将始终选择最高就绪状态任务作为进入运行状态的任务。示例 8 通过使用 vTaskPrioritySet() API 函数来更改两个任务相对于彼此的优先级来证明这一点。<br />​

示例 8 以两个不同的优先级创建两个任务。 这两个任务都没有任何 API 函数调用可能导致它进入阻塞状态，因此无法处于就绪状态或运行状态。 因此，具有最高相对优先级的任务将始终是调度程序选择处于 “正在运行” 状态的任务。<br />​

示例 8 行为如下：<br />​<br />

1. 创建任务 1（清单 33）具有最高优先级，因此保证首先运行。 任务 1 在将任务 2（清单 34）的优先级提高到高于其自己的优先级之前打印出几个字符串。
1. 任务 2 一旦具有最高相对优先级就开始运行（进入运行状态）。 任何时候只有一个任务可以处于运行状态，因此当任务 2 处于运行状态时，任务 1 处于就绪状态。
1. 任务 2 在将自己的优先级设置回低于任务 1 的优先级之前打印出一条消息。
1. 任务 2 将其优先级降低意味着任务 1 再次成为优先级最高的任务，因此任务 1 进入运行状态，强制将任务 2 回到就绪状态。
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
清单 35. 示例 8 的 main() 的实现<br />​

图 22 显示了示例 8 任务执行的顺序，结果输出如图 23 所示。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005149753-00f92ede-12e4-4676-9c1f-c1033c91be31.png#clientId=udb212963-d539-4&from=paste&id=u1df990f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=853&originWidth=746&originalType=url&ratio=1&size=123504&status=done&style=none&taskId=u86e3ec02-1db5-4e27-ac96-ac489f0ce15)<br />图 22. 运行示例 8 时的任务执行顺序<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005149758-ae7dbbb5-5b42-498d-a9f2-657426972854.png#clientId=udb212963-d539-4&from=paste&id=ue6f622ca&margin=%5Bobject%20Object%5D&name=image.png&originHeight=708&originWidth=1416&originalType=url&ratio=1&size=255334&status=done&style=none&taskId=u08290988-4229-449d-9b5e-e0e9eb2534e)<br />图 23. 执行示例 8 时产生的输出

---

<a name="EY6Kn"></a>
## 删除任务
<a name="xQuW6"></a>
### vTaskDelete() API 函数
任务可以使用 vTaskDelete() API 函数来删除自身或任何其他任务。 请注意，只有在 FreeRTOSConfig.h 中将 INCLUDE_vTaskDelete 设置为 1 时，vTaskDelete() API 函数才可用。<br />删除的任务将不再存在，且不能再进入运行模式。<br />释放已删除的任务所分配的内存是空闲任务的职责。因此，重要的是，使用 vTaskDelete() API 函数的应用程序不占用所有处理时间，而饿死空闲任务。<br />​<br />
:::tips
只有内核本身分配给任务的内存在删除任务时才会自动释放。任何分配的任务的实现必须明确释放的内存或其他资源。
:::
```groovy
void vTaskDelete( TaskHandle_t pxTaskToDelete );
```
清单 36. vTaskDelete() 函数原型<br />表 13. vTaskDelete() 参数

| 参数名称/返回值 | 描述 |
| --- | --- |
| ​

​

pxTaskToDelete | 要删除的任务的句柄（主题任务）—— 请参阅 xTaskCreate() API 函数的pxCreatedTask 参数，以获取有关获取任务句柄的信息。<br />任务可以通过在有效的任务句柄处传递 NULL 来删除自己。 |

<a name="WH3vu"></a>
### 示例9. 删除任务
这是一个非常简单的示例，其行为如下。<br />​<br />

1. 任务 1 由 main() 创建，优先级为 1。当它运行时，它以优先级 2 创建任务 2。任务 2 现在是最高优先级的任务，因此它立即开始执行。main() 的源代码如清单 37 所示，任务 1 的源代码如清单 38 所示。

​<br />

2. 任务 2 除了删除自己之外，什么也不会做。它可以通过向 vTaskDelete() 传递 NULL 来删除自己，但是处于演示目的，它会使用自己的任务句柄。任务 2 的源代码如清单 39 所示。

​<br />

3. 当任务 2 被删除后，任务 1 再次成为优先级最高的任务，所以它将继续执行 —— 此时它会调用 vTaskDelay() 来阻塞一小段时间。

​<br />

4. 空闲任务会在任务 1 阻塞状态的时候执行，并释放之前分配给先已删除了的任务 2 的内存。

​<br />

5. 当任务 1 离开阻塞状态的时候，它将再次变为最高优先级的就绪状态任务，从而抢占空闲任务。当进入运行状态后，它将再次创建任务 2， 并且如此这样继续下去。
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
清单 39. 示例 9 任务 2 的实现<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005679119-d75bf7e7-93ac-4db3-a9cb-b7f58677a559.png#clientId=udb212963-d539-4&from=paste&id=ue55ccab1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=638&originWidth=1271&originalType=url&ratio=1&size=475465&status=done&style=none&taskId=u4a966b29-2636-4b19-a6b9-09b87dfc713)<br />图 24. 执行示例 9 的输出结果<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636005678926-6cd45eaf-06f1-4961-89ac-b416728bc6a0.png#clientId=udb212963-d539-4&from=paste&id=ub3e559c3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=868&originWidth=1125&originalType=url&ratio=1&size=130846&status=done&style=none&taskId=u9bfc5f90-b5a0-412a-836f-18cf35f8aed)<br />图 25. 示例 9 的执行顺序

---

<a name="ydWhh"></a>
## 线程本地存储
待完成。本节将在最终发布之前编写。<br />​<br />
:::tips
译者注：原文就是待完成。
:::
​<br />

---

<a name="ksR6e"></a>
## 调度算法
<a name="piFxt"></a>
### 任务状态和事件的回顾
实际运行的任务（使用处理时间）处于运行状态。 在单核处理器上，在任何给定时间只能有一个任务处于运行状态。<br />实际上未运行的任务，既没有处于阻塞状态，也没有处于挂起状态，那就是处于就绪状态。调度程序可以选择处于就绪状态的任务作为进入运行状态的任务。调度程序总是选择优先级最高的就绪状态任务来进入运行状态。<br />任务可以在阻塞状态中等待事件，并在事件发生时自动移动回就绪状态。时间事件发生在特定的时间，例如，当阻塞时间过期时，通常用于实现周期性或超时行为。同步事件发生在任务或中断服务例程使用任务通知、队列、事件组或多种信号量类型之一发送信息时。它们通常用于发出异步活动的信号，例如有数据到达外设。
<a name="h3UIA"></a>
### 配置调度算法
调度算法是决定哪个就绪状态的任务变成运行任务的程序。<br />到目前为止，所有示例都使用了相同的调度算法，但可以通过配置 configUSE_PREEMPTION 和 configUSE_TIME_SLICING 常量更改算法。 两个常量都在 FreeRTOSConfig.h 中定义。<br />第三个配置常量 configUSE_TICKLESS_IDLE 也会影响调度算法，因为它的使用会导致滴答中断在较长时间内完全关闭。configUSE_TICKLESS_IDLE 是一个高级选项，专门用于必须最小化其功耗的应用程序。 configUSE_TICKLESS_IDLE 在第10章，低功耗支持中描述。 本节中提供的描述假设configUSE_TICKLESS_IDLE 设置为 0，如果常量未定义，则这是默认设置。<br />在所有可能的配置中，FreeRTOS 调度程序将确保选择共享优先级的任务以依次进入运行状态。 这种 “反过来” 政策通常被称为 “循环调度”。 循环调度算法不保证在相同优先级的任务之间平均分配时间，只有优先级相同的就绪状态任务依次进入运行状态。
<a name="Otd83"></a>
### 具有时间切片的优先抢占式调度
表 14 中所示的配置将 FreeRTOS 调度程序设置为使用称为“带时间切片的固定优先级预先调度” 的调度算法，这是大多数小型 RTOS 应用程序使用的调度算法，以及所有示例中使用的算法。 到目前为止这本书。 表 15 中提供了算法名称中使用的术语的描述。<br />表 14. FreeRTOSConfig.h 用于配置内核以使用带时间片的优先级预先调度的设置

| 常量 | 值 |
| --- | --- |
| configUSE_PREEMPTION | 1 |
| configUSE_TIME_SLICING | 1 |

表 15. 用于描述调度策略的术语的解释<br />术语

| 术语 | 定义 |
| --- | --- |
| 固定优先级 | 描述为 “固定优先级” 的调度算法不会更改分配给正在调度的任务的优先级，但也不会阻止任务本身更改自己的优先级或其他任务的优先级。 |
| 抢占 | 如果优先级高于运行状态任务的任务进入就绪状态，则先发制人调度算法将立即“抢占”运行状态任务。 被抢占意味着不自觉地（不明显地屈服或阻塞）移出运行状态并进入就绪状态以允许不同的任务进入运行状态。 |
| 时间分片 | 时间分片用于在相同优先级的任务之间共享处理时间，即使任务确实明确地产生或进入阻塞状态。 如果存在与运行任务具有相同优先级的其他就绪状态任务，则使用时间分片描述的调度算法将选择新任务以在每个时间片结束时进入运行状态。时间片等于两次 RTOS 滴答中断之间的时间。 |

图 26 和图 27 说明了当使用具有时间分片算法的固定优先级抢占式调度时如何调度任务。 图 26 显示了当应用程序中的所有任务具有唯一优先级时，选择任务以进入“运行”状态的顺序。 图 27 显示了当应用程序中的两个任务共享优先级时，选择任务以进入“运行”状态的顺序。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163237-1d4e45b5-6947-4f62-abed-8c5843fc6685.png#clientId=udb212963-d539-4&from=paste&id=ub6e78c49&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1420&originalType=url&ratio=1&size=147345&status=done&style=none&taskId=u1ed72536-a84a-47bf-a166-a6150a3d7b1)<br />图 26. 执行模式在一个假设的应用程序中突出显示任务优先级和优先级，其中每个任务都被赋予了唯一的优先级<br />
<br />图 26 参考：<br />​<br />

1. ​**空闲任务**：空闲任务以最低优先级运行，因此每次较高优先级任务进入就绪状态时都会被抢占 —— 例如，在时间 t3，t5 和 t9。
1. ​**任务 3**：任务 3 是一个事件驱动的任务，它以相对较低的优先级执行，但高于空闲优先级。 它大部分时间都处于阻塞状态，等待其感兴趣的事件，每次事件发生时从阻塞状态转换到就绪状态。 所有 FreeRTOS 任务间通信机制（任务通知，队列，信号量，事件组等）都可用于以这种方式发送事件和解除阻塞任务。事件发生在时间 t3 和 t5，也发生在 t9 和 t12 之间。 在时间 t3 和 t5 发生的事件被立即处理，因为在这些时间，任务 3 是能够运行的最高优先级任务。 在时间 t9 和 t12 之间某处发生的事件直到 t12 才被处理，因为在此之前，优先级较高的任务任务 1 和任务 2 仍在执行。 仅在时间 t12，任务 1和任务 2 都处于阻塞状态，使得任务 3 成为最高优先级就绪状态任务。
1. ​**任务 2**：任务 2 是周期性任务，其执行优先于任务 3 的优先级，但低于任务 1 的优先级。任务的周期间隔意味着任务 2 想要在时间 t1，t6 和 t9 执行。在时间 t6，任务 3 处于运行状态，但任务 2 具有较高的相对优先级，因此预先占用任务 3 并立即开始执行。 任务 2 完成其处理并在时间 t7 重新进入阻塞状态，此时任务 3 可以重新进入运行状态以完成其处理。 任务 3 本身在时间 t8 阻塞。
1. **任务 1**： 任务 1 也是一个事件驱动任务。 它以所有的最高优先级执行，因此可以抢占系统中的任何其他任务。 所示的唯一任务 1 事件发生在时间 t10，此时任务 1 抢占任务 2。任务 2 仅在任务 1 在时间t11 重新进入阻止状态之后才能完成其处理。


<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163243-98b7aa16-3853-47e8-9cad-6ede003b0e10.png#clientId=udb212963-d539-4&from=paste&id=u98f897a0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=415&originWidth=1477&originalType=url&ratio=1&size=79519&status=done&style=none&taskId=u8ab06b99-ab3a-400e-b9bb-2e7ddbc3695)<br />图 27. 执行模式在一个假设的应用程序中突出显示任务优先级和时间切片，其中两个任务以相同的优先级运行<br />
<br />图 27 参考：

1. ​**空闲任务与任务 2**：空闲任务和任务 2 都是连续处理任务，两者的优先级均为 0（可能的最低优先级）。 当没有能够运行的优先级较高的任务时，调度器会将处理时间分配给优先级为 0 的任务，并通过时间分片共享分配给优先级 0 任务的时间。 每个嘀嗒中断开始一个新的时间片，在图 27 中，时间为 t1，t2，t3，t4，t5，t8，t9，t10 和 t11。空闲任务和任务 2 依次进入运行状态，这可导致两个任务在同一时间片的一部分处于运行状态，如在时间 t5 和时间 t8 之间发生。
1. **任务 1**：任务 1 的优先级高于空闲优先级。 任务 1 是一个事件驱动的任务，它将大部分时间都置于阻塞状态，等待它感兴趣的对象，每次事件发生时从阻塞状态转换到就绪状态。感兴趣的事件发生在时间 t6，因此在 t6，任务 1 成为能够运行的最高优先级任务，因此任务 1 在时间片中部分地抢占空闲任务。 事件的处理在时间 t7 完成，此时任务 1 重新进入阻止状态。


<br />图 27 显示了由应用程序编写者创建的任务的空闲任务共享处理时间。 如果应用程序编写者创建的空闲优先级任务有工作要做，但空闲任务没有，则可能不需要将大量处理时间分配给空闲任务。configIDLE_SHOULD_YIELD 编译时配置常量可用于更改空闲任务的调度方式：<br />​<br />

- 如果 configIDLE_SHOULD_YIELD 设置为 0，那么空闲任务将保持其整个时间片的运行状态，除非它被更高优先级的任务抢占。
- 如果 configIDLE_SHOULD_YIELD 设置为 1，那么空闲任务将在其循环的每次迭代中产生（自愿放弃其分配的时间片的剩余部分），并且在就绪状态下还有其他空闲优先级任务。


<br />图 27 中显示的执行模式是当 configIDLE_SHOULD_YIELD 设置为 0 时将观察到的内容。图 28 中显示的执行模式是在 configIDLE_SHOULD_YIELD 设置为 1 时在同一场景中观察到的内容。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006163221-747cec7a-77d0-4b45-901f-6312954d9970.png#clientId=udb212963-d539-4&from=paste&id=uddec8b93&margin=%5Bobject%20Object%5D&name=image.png&originHeight=470&originWidth=1321&originalType=url&ratio=1&size=104740&status=done&style=none&taskId=u4e2a45d0-68ba-4e20-82e7-25dc2041605)<br />图 28. 同一场景的执行模式如图 27 所示，但这次将 configIDLE_SHOULD_YIELD 设置为1<br />图 28 还显示，当 configIDLE_SHOULD_YIELD 设置为 1 时，在空闲任务之后选择进入运行状态的任务不会对整个时间片执行，而是执行转发空闲任务所产生的时间片的剩余时间。
<a name="UWcUr"></a>
### 优先级抢占式调度(没有时间分片)
没有时间分片的优先级抢先式调度保持与前一节中描述的相同的任务选择和抢占算法，但不使用时间分片来共享相同优先级的任务之间的处理时间。<br />​

FreeRTOSConfig.h 设置将 FreeRTOS 调度程序配置为使用优先级抢占式调度而不进行时间切片，如表 16所示。<br />​

表 16. FreeRTOSConfig.h 设置，用于配置内核以使用优先级预先调度而无需时间切片

| 常量 | 值 |
| --- | --- |
| configUSE_PREEMPTION | 1 |
| configUSE_TIME_SLICING | 0 |

如图 27 所示，如果使用时间分片，并且有多个能够运行的最高优先级的就绪状态任务，那么调度程序将选择一个新任务以在每个 RTOS 节拍中断期间进入运行状态 （标记时间片结束的滴答中断）。 如果未使用时间分片，则调度程序将仅选择新任务以在以下任一情况下进入运行状态：

- 优先级较高的任务进入就绪状态。
- 处于运行状态的任务进入阻塞或挂起状态。

当不使用时间分片时，与使用时间分片时相比，任务上下文切换更少。 因此，关闭切换时间会导致调度程序的处理开销减少。但是，关闭时间切换也会导致相同优先级的任务接收到大大不同的处理时间量，如图 29 所示。出于这个原因，运行调度程序没有时间切片被认为是一种只有经验丰富的用户才能使用的高级技术。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006430382-395d6c69-99e1-4193-84f9-1a7f47caa650.png#clientId=udb212963-d539-4&from=paste&id=ub1021141&margin=%5Bobject%20Object%5D&name=image.png&originHeight=388&originWidth=1313&originalType=url&ratio=1&size=91755&status=done&style=none&taskId=u3843cb4b-6343-4efb-bc7c-01d1fefafcc)<br />图 29. 执行模式，演示当不使用时间切片时，具有相同优先级的任务如何可以获得非常不同的处理时间量<br />图 29 参考，假设 configIDLE_SHOULD_YIELD 设置为 0：<br />​<br />

1. **滴答中断**：滴答中断发生在 t1、t2、t3、t4、t5、t8、t11、t12 和 t13 时刻。
1. ​**任务 1**：任务 1 是一个高优先级事件驱动的任务，大部分时间都处于阻塞状态，等待它感兴趣的对象发生。 每次事件发生时，任务 1 从阻塞状态转换到就绪状态（并且随后，因为它是最高优先级的就绪状态任务，进入运行状态）。图 29 显示在时间 t6 和 t7 之间处理事件的任务 1， 然后在时间 t9 和 t10 之间再次出现。
1. ​**空闲任务与任务 2**：空闲任务和任务 2 都是连续处理任务，并且两者都具有优先级 0（空闲优先级）。连续处理任务不进入阻塞状态。未使用时间分片，因此处于“正在运行”状态的空闲优先级任务将保持“正在运行” 状态，直到它被优先级较高的任务 1 抢占为止。在图 29 中，空闲任务在时间 t1 开始运行，并且保持在运行状态，直到它被任务 1 在时间 t6 抢占 —— 这在进入运行状态之后超过四个完整的滴答时段。任务 2 在时间 t7 开始运行，这时任务 1 重新进入阻塞状态以等待另一个事件。 任务 2 保持在运行状态，直到任务 1 在时间 t9 之前被抢占 —— 在进入运行状态之后小于一个滴答周期。在时间 t10，空闲任务重新进入运行状态，尽管已经接收到比任务 2 多四倍以上的处理时间。
<a name="iDmUW"></a>
### 协作式调度
本书主要介绍抢占式调度，但 FreeRTOS 也可以使用协作式调度。配置 FreeRTOS 调度程序使用协作式调度的 FreeRTOSConfig.h 设置如表 17 所示。<br />表 17. FreeRTOSConfig.h 设置，配置内核使用协作式调度

| 常量 | 值 |
| --- | --- |
| configUSE_PREEMPTION | 0 |
| configUSE_TIME_SLICING | 任何值 |

当使用协作式调度程序时，只有在运行状态任务进入阻塞状态，或者运行状态任务通过调用 taskYIELD() 显式地产生（手动请求重新调度）时，才会发生上下文的切换。 任务永远不会被抢占，因此不能使用时间切片。<br />​

图 30 展示了协作式调度程序的行为。图 30 中的水平虚线显示任务何时处于就绪状态。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636006700649-8ef52549-2f45-48ad-89c1-0111cca7958b.png#clientId=udb212963-d539-4&from=paste&id=u6518d0ac&margin=%5Bobject%20Object%5D&name=image.png&originHeight=450&originWidth=1334&originalType=url&ratio=1&size=99771&status=done&style=none&taskId=ueaf9a6a4-5cd6-4acb-a5cc-553103a5666)<br />图 30. 显示协作式调度程序行为的执行模式<br />图 30 参考：<br />​<br />

1. **任务 1**：任务 1 有最高优先级，它开始于阻塞状态，等待一个信号量。在 t3 时刻，一个中断给予了信号量，引起任务 1 离开阻塞状态并进入就绪状态（第 6 章涵盖了来自中断给与的信号量的相关内容）。在 t3 时刻，任务 1 是最高优先级的任务，如果是抢占模式调度器会将任务 1 直接变成运行态任务，然而在协作式调度下，任务 1 任然是就绪态，直到 t4 ——当运行态的任务调用 taskYIELD() 时。
1. **任务 2**：任务 2 的优先级介于任务 1 与任务 3。它同样开始于阻塞状态，等待一个在 t2 时刻由任务 3 发出的消息。在 t2 时刻，任务 2 是最高优先级的就绪态任务，如果是抢占式调度，会将任务 2 直接变成运行态任务，然而在协作式调度下，任务 2 任然是就绪态，知道运行态任务调用 taskYIELD() 。运行态任务调用 taskYIELD() 在 t4 时刻，但是任务 1 是最高优先级的就绪态任务，所以任务 2 实际没有成为运行态任务，直到 t5 时刻任务 1 重新进入阻塞状态。在 t6 时刻，任务 2 重新进入阻塞状态来等待下一个消息，在这个点任务 3 再一次成为最高优先级的就绪态任务。

在一个多任务应用程序中，应用程序编写者必须注意同一个资源会被多个任务同时访问，因为同时访问可能会对该资源造成破坏。例如，考虑下面的场景，其中访问的时 UART 接口。两个任务是将字符串写入 UART， 任务 1 写的是 "abcdefghijklmnop" ，任务 2 写的是 "123456789" ：<br />​<br />

1. 任务 1 是运行态，并且开始写字符串。当写到 "abcdefg" 时离开运行态。
1. 任务 2 进入运行态，同样开始写字符串 "123456789" 到 UART，并离开运行态。
1. 任务 1 重新进入，并将剩余的字符串写入 UART。


<br />在这种情况下实际写入 UART 的是 "abcdefg123456789hijklmnop" 。任务 1 编写的字符串没有按照预期的顺序完整的写入 UART，而是被破坏了，因为任务 2 写入 UART 的字符串有人出现在其中。<br />​

通常，使用协作式调度比抢占式调度更容易避免由于时访问而造成的问题：<br />​<br />

- 当使用抢占式调度时，可以在任何时刻抢占运行态的任务，包括与另一个任务共享资源的处于不一致的状态时。正如 UART 的示例所示，让资源处于不一致的状态可能会导致资源损坏。
- 当使用协作式调度时，应用程序编写者控制着任务合适切换到另一个任务。因此，应用程序编写者能够确保资源处于不一致状态时不会被切换到另一个任务。
- 在上面的 UART 示例中，应用程序编写者可以确保任务 1 在整个字符串写入 UART 之前不会离开运行状态，这样做可以消除字符串被另一个任务激活所破坏的可能性。


<br />如图 30 所示，使用协作式调度的系统响应速度将低于抢占式调度的响应速度：<br />​<br />

- 当使用抢占式调度时，调度程序将立即开始运行任务，该任务将成为最高优先级的就绪态任务。这在实时系统中，通常是必不可少的，因为实时系统必须在规定的时间内响应最高优先级事件。
- 当使用协作式调度时，只有正在运行的任务进入阻塞状态或者调用 taskYIELD() 时，才会切换到已成为最高优先级的就绪态任务。

---