<a name="CGoT9"></a>
# **队列管理**

---

<a name="XEGUV"></a>
## 章节介绍与范围
“队列” 提供了一个**任务到任务**，**任务到中断**和**中断到任务**的通讯机制。
<a name="O4M5l"></a>
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

---

<a name="kuDO5"></a>
## 队列的特征
<a name="lBAo3"></a>
### 数据存储
一个队列能保存有限数量的固定大小的数据单元。一个队列能保存单元的最大数量叫做 “长度”。每个队列数据单元的长度与大小是在创建队列时设置的。<br />​

队列通常是一个先入先出（FIFO）的缓冲区，即数据在队列末尾（tail）被写入，在队列前部（head）移出。图 31 展示了数据被写入和移出作为 FIFO 使用的队列。也可以写入队列的前端，并覆盖已位于队列前端的数据。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636007539867-7cde74d6-3c85-4ca7-8964-91adf229f12a.png#clientId=udb212963-d539-4&from=paste&id=uf93179e0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1143&originWidth=886&originalType=url&ratio=1&size=190569&status=done&style=none&taskId=u15852ce1-975f-43c2-a67a-e01e4accc2b)<br />图 31. 写入队列和从队列读取的示例序列<br />有两种方法可以实现队列的行为：<br />​<br />

1. 通过复制实现队列：复制队列是指将发送到队列的数据一个字节一个字节地复制到队列中。
1. 通过引用实现队列：引用队列意味着队列只持有指向发送到队列的数据的指针，而不是数据本身。

​

FreeRTOS 是通过使用复制方法实现队列。这是考虑到复制队列比引用队列更强大，更容易使用，因为：<br />​<br />

- 堆栈变量可以直接发送到队列，即使该变量将在声明它的函数退出后，不再存在。
- 可以将数据发送到队列，而无需先分配缓冲区来保存数据，然后将数据复制到分配的缓冲区中。
- 发送任务可以立即重用发送到队列的变量或缓冲区。
- 发送任务和接收任务是完全解耦的，应用程序设计人员不需要关心哪个任务拥有数据，或者哪个任务负责发布数据。
- 复制队列并不会阻止队列也被用于引用队列。例如，当正在排队的数据的大小使得将数据复制到队列不切实际时，可以将指向数据的指针复制到队列中。
- RTOS 完全负责分配用于存储数据的内存。
- 在受内存保护的系统中，任务可以访问的 RAM 将受到限制。在这种情况下，只有当发送和接收任务都可以访问存储数据的 RAM 时，才可以使用引用排队。按复制排队不受此限制；内核总是以完全特权运行，允许使用队列跨内存保护边界传递数据。
<a name="egPKw"></a>
### 多任务访问
队列本身就是对象，任何知道它们存在的任务或 ISR 都可以访问它们。任意数量的任务可以写入同一个队列，任意数量的任务也可以从同一个队列读取。在实践中，队列有多个写入者是非常常见的，但是队列有多个读取者就不那么常见了。
<a name="fMPiu"></a>
### 阻塞队列读取
当任务尝试从队列中读取时，它可以选择指定 “阻塞” 时间。 如果队列已经为空，则这是任务将保持在阻塞状态以等待队列中的数据可用的时间。 当另一个任务或中断将数据放入队列时，处于阻塞状态且等待数据从队列中变为可用的任务将自动移至就绪状态。 如果指定的阻塞时间在数据可用之前到期，则任务也将自动从 “阻塞” 状态移动到 “就绪” 状态。<br />队列可以有多个读取者，因此单个队列可能会由多个在其上阻塞等待数据的任务。 在这种情况下，只有一个任务在数据可用时将被解除阻塞。 取消阻塞的任务始终是等待数据的最高优先级任务。 如果被阻塞的任务具有相同的优先级，那么等待数据最长的任务将被阻塞。
<a name="Hx9T6"></a>
### 阻塞队列写入
与从队列读取数据时一样，任务也可以在向队列写入数据时指定阻塞时间。在这种情况下，如果队列已经满了，则阻塞时间是任务应该保持在阻塞状态以等待队列上可用空间的最长时间。<br />队列可以有多个写入者，因此对于一个完整的队列，可能有多个任务阻塞在队列上，等待完成发送操作。在这种情况下，当队列上的空间可用时，只有一个任务将被解除阻塞。未阻塞的任务总是等待空间的最高优先级任务。如果阻塞的任务具有相同的优先级，那么等待空间最长的任务将被解除阻塞。
<a name="az69I"></a>
### 阻塞多个队列
队列可被分组到集合中，允许任务进入阻塞状态来等待数据在集合的任何队列中变为可用。队列集合在第 4.6 章节 “从多个队列接收” 中展示。

---

<a name="vg6iq"></a>
## 使用队列
<a name="YtO9x"></a>
### xQueueCreate() API 函数
一个队列在使用前必须被显式的创建。<br />队列被句柄引用，句柄是类型为 QueueHandle_t 类型的变量。xQueueCreate() API 函数会创建一个队列，并给一个 QueueHandle_t 的变量来引用这个被创建的队列。<br />FreeRTOS V9.0.0 也包含了 xQueueCreateStatic() 函数，它创建队列是在编译时静态地分配内存。当一个队列创建时，FreeRTOS 是从 FreeRTOS 堆中分配所需 RAM。这一段 RAM 被用来保存队列数据结构和队列所含的各个单元。xQueueCreate() 在创建队列所需 RAM 不足时会返回 NULL 。第 2 章提供了 FreeRTOS 堆的更多信息。
```groovy
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength, UBaseType_t uxItemSize);
```
清单 40. xQueueCreate() API 函数原型<br />表 18. xQueueCreate() 参数和返回值

| 参数名 | 描述 |
| --- | --- |
| uxQueueLength | 正在创建的队列一次可以容纳的最大项数。 |
| uxItemSize | 可以存储在队列中的每个数据项的字节大小。 |
| 返回值 | 如果返回 NULL，则无法创建队列，因为 FreeRTOS 没有足够的堆内存来分配队列数据结构和存储区域。返回的非空值表示队列已成功创建。返回的值应该存储为已创建队列的句柄。 |


创建队列后，可以使用 xQueueReset() API 函数将队列返回到其原始的空状态。
<a name="S9i2Z"></a>
### xQueueSendToBack() 与 xQueueSendToFront() API 函数
正如所料，xQueueSendToBack() 用于将数据发送到队列的后端（尾部），xQueueSendToFront() 用于将数据发送到队列的前端（头部）。<br />​

xQueueSend() 与 xQueueSendToBack() 等价，并且完全相同。创建队列后，可以使用 xQueueReset() API 函数将队列返回到其原始的空状态。<br />​<br />
>
永远不要从中断服务例程调用 xQueueSendToFront() 或 xQueueSendToBack()。应该使用中断安全转换 xQueueSendToFrontFromISR() 和 xQueueSendToBackFromISR() 。这些将在第 6 章中描述。

​<br />
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
清单 42. xQueueSendToBack() API 函数原型<br />​

表 19. xQueueSendToFront() 和 xQueueSendToSendToBack() 函数参数和返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| xQueue | 发送(写入)数据的队列的句柄。队列句柄将从用于创建队列的 xQueueCreate() 调用中返回。 |
| pvItemToQueue | 指向要复制到队列中的数据的指针。<br />在创建队列时，将设置队列可以容纳的每个项目的大小，因此这多个字节将从 pvItemToQueue 复制到队列存储区域中。 |
| xTicksToWait | 如果队列已经满了，任务应该保持阻塞状态以等待队列上可用空间的最大时间量。<br />如果 xTicksToWait 为零且队列已满，则 xQueueSendToFront() 和 xQueueSendToBack() 都将立即返回。<br />阻塞时间以滴答周期指定，因此它所表示的绝对时间依赖于滴答频率。宏 pdMS TO TICKS() 可用于将以毫秒为单位的时间转换为以节拍为单位的时间。<br />如果在 FreeRTOSConfig.h 中将 INCLUDE_vTaskSuspend 设置为 1，则将 xTicksToWait 设置为 portMAX_DELAY 将导致任务无限期地等待（没有超时）。 |
| 返回值 | 有两种可能的返回值：<br />1. 1.pdPASS：仅当数据成功发送到队列时，才会返回 pdPASS。如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务可能被置于Blocked状态，等待空间在队列中变为可用，在函数返回之前，但数据已成功写入队列 在阻止时间到期之前。<br />1. 2.errQUEUE_FULL：如果由于队列已满，无法将数据写入队列，将返回errQUEUE_FULL 。如果指定了阻塞时间（xTicksToWait 不为零），则调用任务将被置于阻塞状态以等待另一个任务或中断在队列中腾出空间，但指定的阻塞时间在该状态之前到期。<br /> |

xQueueReceive() 是用来从队列中接收（读取）一个元素。收到的元素将从队列中删除。<br />​<br />
>
切勿从中断服务程序调用 xQueueReceive()。 中断安全 xQueueReceiveFromISR() API 函数在第 6 章中描述。

​<br />
```verilog
BaseType_t xQueueReceive( QueueHandle_t xQueue,
                          void *const pvBuffer,
                          TickType_t xTicksToWait );
```
清单 43. xQueueReceive() API 函数原型<br />表 20. xQueueReceive() 函数参数和返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| xQueue | 正在接收（读取）数据的队列句柄。<br />将从用于创建队列的 xQueueCreate() 调用返回队列句柄。 |
| pvBuffer | 指向要将接收到的数据复制到的内存的指针。<br />在创建队列时设置队列保存的每个数据项的大小。 pvBuffer 指向的内存必须至少足以容纳那么多字节。 |
| xTicksToWait | 如果队列已经为空，则任务应保持在阻塞状态以等待数据的最长时间在队列中可用。<br />如果 xTicksToWait 为零，那么如果队列已经为空，则 xQueueReceive() 将立即返回。<br />阻塞时间在滴答周期中指定，因此它表示的绝对时间取决于滴答频率。 宏 pdMS_TO_TICKS() 可用于将以毫秒为单位指定的时间转换为刻度中指定的时间。<br />将 xTicksToWait 设置为 portMAX_DELAY 会导致任务无限期地等待（没有超时），前提是 FreeRTOSConfig.h 中的 INCLUDE_vTaskSuspend 设置为 1。 |
| 返回值 | 有两种可能的返回值：<br />1. 1.pdPASS：仅当从队列中成功读取数据时才会返回 pdPASS。 如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务可能被置于阻塞状态，等待数据在队列中可用，但是在阻塞时间到期之前已成功从队列中读取数据。<br />1. 2.errQUEUE_EMPTY：如果由于队列已经为空而无法从队列中读取数据，则将返回errQUEUE_EMPTY。如果指定了阻塞时间（xTicksToWait 不为零），那么调用任务将被置于阻塞状态以等待另一个任务或中断将数据发送到队列，但阻塞时间在该时间之前到期。<br /> |


uxQueueMessagesWaiting() 用于查询当前队列中的项目数。<br />​<br />
>
切勿从中断服务程序调用 uxQueueMessagesWaiting()。 应该在其位置使用中断安全 uxQueueMessagesWaitingFromISR()。

​<br />
```verilog
UBaseType_t uxQueueMessagesWaiting( QueueHandle_t xQueue );
```
清单 44. uxQueueMessagesWaiting() API 函数原型<br />表 21. uxQueueMessagesWaiting() 函数参数或返回值

| 参数名称/返回值 | 描述 |
| --- | --- |
| xQueue | 正在查询队列的句柄。 将从用于创建队列的 xQueueCreate() 调用返回队列句柄。 |
| 返回值 | 正在查询的队列当前持有的项目数。 如果返回零，则队列为空。 |

<a name="FBMro"></a>
### 示例 10. 从队列接收时阻塞
此示例演示了正在创建的队列，从多个任务发送到队列的数据以及从队列中接收的数据。 创建队列以保存 int32_t 类型的数据项。 发送到队列的任务不指定阻塞时间，从队列接收的任务执行。<br />​

发送到队列的任务的优先级低于从队列接收的任务的优先级。 这意味着队列永远不应包含多个项目，因为只要数据被发送到队列，接收任务就会解锁，抢占发送任务，并删除数据 - 再次将队列留空。<br />​

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
清单 45. 示例 10 中使用的发送任务的实现。<br />​

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
清单 46. 示例 10 接受任务的实现<br />​

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
清单 47. 例 10 中 main() 的实现<br />​

发送到队列的两个任务都具有相同的优先级。 这导致两个发送任务依次将数据发送到队列。 例 10 中产生的输出如图 32 所示。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009217986-4d4b1766-a395-47f3-b5ea-874aa265b6ed.png#clientId=udb212963-d539-4&from=paste&id=u2552b462&margin=%5Bobject%20Object%5D&name=image.png&originHeight=818&originWidth=1643&originalType=url&ratio=1&size=519227&status=done&style=none&taskId=u5fd79088-29d7-47b3-8205-9a6fb632a7e)<br />图 32. 执行示例 10 产生的输出<br />图 33 展示了执行的顺序<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009217849-e08f6a41-477f-4e96-b904-7f536a748b7f.png#clientId=udb212963-d539-4&from=paste&id=u03f07381&margin=%5Bobject%20Object%5D&name=image.png&originHeight=747&originWidth=1346&originalType=url&ratio=1&size=167332&status=done&style=none&taskId=ud24955ff-5990-4457-82eb-4f3581e80e9)<br />图 33 .示例 10 执行顺序

---

<a name="W93M2"></a>
## 从多个源接收数据
FreeRTOS 设计中常见的任务是从多个源接收数据，所以接收任务需要知道数据来自何处以确定如何处理数据。 一个简单的设计解决方案是使用单个队列来传输具有数据值和结构域中包含的数据源的结构。 该方案如图 34 所示。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636009291854-192cb58f-4eef-407d-85e5-e542e10dc802.png#clientId=udb212963-d539-4&from=paste&id=u6735f044&margin=%5Bobject%20Object%5D&name=image.png&originHeight=606&originWidth=1301&originalType=url&ratio=1&size=88444&status=done&style=none&taskId=u77dd374e-17b1-4d01-b8e4-8b958f236a2)<br />图 34. 发送结构到队列的示例场景<br />图 34 参考：<br />​<br />

- 一个以 Data_t 结构创建的队列。这个结构成员允许包含数据值和一个枚举类型来指示一个消息发送到队列。
- 中央的 Controller 任务用于执行主系统功能。 这必须对输入和对队列中与其通信的系统状态的更改作出反应。
- 一个 CAN 总线任务用于封装 CAN 总线接口功能。当 CAN 总线任务接收并解码消息时，它将已解码的消息发送到 Data_t 结构中的 Controller 任务。 传输结构的 eDataID 成员用于让 Controller 任务知道数据是什么 —— 在描述中它是电机速度值。 传输结构的 lDataValue 成员用于让 Controller 任务知道实际的电机速度值。
- 人机界面（HMI）任务用于封装所有 HMI 功能。机器操作员可能以多种方式输入命令和查询值，这些方式必须在 HMI 任务中检测和解释。输入新命令时，HMI 任务将命令以一个 Data_t 的结构发送到 Controller 任务。传输结构的 eDataID 成员用于让 Controller 任务知道数据是什么 —— 在描述中它是一个新的调定点的值。 传递结构的 lDataValue 成员用于让 Controller 任务知道实际调定点的值。



<a name="dSD8j"></a>
### 示例 11. 发送到队列和发送队列结构时的阻塞
示例 11 与示例 10 类似，但任务优先级相反，因此接收任务的优先级低于发送任务。 此外，队列用于传递结构，而不是整数。<br />​

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
清单 48. 要在队列上传递的结构的定义，以及由示例使用的两个变量的声明<br />​

在示例 10 中，接收任务具有最高优先级，因此队列中永远不会存在多个元素。 这是因为一旦数据被放入队列中，接收任务就会抢占发送任务。 在示例 11 中，发送任务具有更高的优先级，因此队列通常是满的。 这是因为，一旦接收任务从队列中删除了一个项目，它就会被其中一个发送任务抢占，然后立即重新填充队列。 然后，发送任务重新进入阻塞状态，等待空间再次在队列中可用。<br />​

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
清单 49. 示例 11 发送任务的实现<br />​

接收任务的优先级最低，所以只有当两个发送任务都处于阻塞状态时，接收任务才会运行。发送任务仅在队列满时才进入阻塞状态，因此接收任务仅在队列满时才会执行。因此，即使没有指定阻塞时间，它也总是期望接收数据。<br />​

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
清单 50. 示例 11 接收任务的定义<br />main() 仅比前一个示例略有变化。 创建队列以容纳三个 Data_t 结构，并且发送和接收任务的优先级相反。 main() 的实现如清单 51 所示。
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
清单 51. 示例 11 main() 的实现<br />​

示例 11 生成的输出如图 35 所示。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636010272533-30942d14-dede-422f-a845-deb1f85342e5.png#clientId=udb212963-d539-4&from=paste&id=u90cd2a3d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=661&originWidth=1323&originalType=url&ratio=1&size=198564&status=done&style=none&taskId=u1a9e1471-feb9-4c27-b46c-04eee45bfa1)<br />图 35. 示例 11 产生的输出<br />图 36 显示了由于发送任务的优先级高于接收任务的优先级而导致的执行顺序。 表 22 提供了对图 36 的进一步说明，并描述了前四个消息是否来自同一任务。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636010257309-6a9aeb4f-b4f2-4080-b8b5-9f3140764004.png#clientId=udb212963-d539-4&from=paste&id=u814191d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=1108&originalType=url&ratio=1&size=72733&status=done&style=none&taskId=ufded997f-3b44-47ab-bb76-adc1d07e906)<br />图 36. 示例 11 的执行顺序<br />
<br />表 22. 图 36 的关键点

| 时刻 | 描述 |
| --- | --- |
| t1 | 任务发送方 1 执行并向队列发送 3 个数据项。 |
| t2 | 队列已满，因此发送方 1 进入阻塞状态，等待下一次发送完成。任务发送方 2 现在是能够运行的最高优先级任务，因此进入运行状态。 |
| t3 | 任务发送者 2 发现队列已经满了，因此进入阻塞状态，等待第一次发送完成。任务接收者现在是能够运行的最高优先级任务，因此进入运行状态。 |
| t4 | 优先级高于接收任务优先级的两个任务正在等待队列中的空间可用，从而导致任务接收者在从队列中删除一个项目后立即被抢占。 任务发送者 1 和发送者 2 具有相同的优先级，因此调度程序选择等待时间最长的任务作为将进入运行状态的任务 —— 在这种情况下是任务发送者 1。 |
| t5 | 任务发送者 1 将另一个数据项发送到队列。 队列中只有一个空间，因此任务发送者 1 进入阻塞状态以等待下一次发送完成。 任务接收器再次是能够运行的最高优先级任务，因此进入运行状态。<br />任务发送者 1 现在已向队列发送了四个项目，任务发送者 2 仍在等待将其第一个项目发送到队列。 |
| t6 | 优先级高于接收任务优先级的两个任务正在等待队列中的空间可用，因此任务接收者一旦从队列中删除了一个项目就会被抢占。 此时发送者 2 等待的时间比发送者 1 长，因此发送者 2 进入运行状态。 |
| t7 | 任务发送者 2 将数据项发送到队列。 队列中只有一个空格，因此发件人 2 进入阻止状态以等待下一次发送完成。 发送者 1 和发送者 2 都在等待队列中的空间可用，因此任务接收者是唯一可以进入运行状态的任务。 |

<a name="g2tXT"></a>
### 排队指针
如果存储在队列中的数据大小很大，则最好使用队列将指针传输到数据，而不是将数据本身逐字节地复制到队列中。 传输指针在处理时间和创建队列所需的 RAM 量方面都更有效。 但是，在队列指针时，必须特别注意确保：<br />​<br />

1. 指向的 RAM 的所有者是明确定义的。通过指针在任务之间共享内存时，必须确保两个任务不会同时修改内存内容，或采取任何其他可能导致内存内容无效或不一致的操作。 理想情况下，只允许发送任务访问存储器，直到指向存储器的指针已经排队，并且在从队列接收到指针之后，只允许接收任务访问存储器。
1. 指向的 RAM 仍然有效。如果指向的内存是动态分配的，或者是从预先分配的缓冲区池中获取的，则完全一个任务应该负责释放内存。 任务完成后，任何任务都不应尝试访问内存。永远不应该使用指针来访问已在任务堆栈上分配的数据。 堆栈帧更改后，数据无效。


<br />举例来说，清单 52，清单 53 和清单 54 演示了如何使用队列从一个任务向另一个任务发送指向缓冲区的指针：<br />​<br />

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
<a name="PvZNa"></a>
### 使用队列发送不同类型和长度的数据
前面几节已经证明了两个强大的设计模式：发送结构到一个队列，发送指针到一个队列。 组合这两个技术就可以允许一个任务使用一个队列接收来自任何数据源的任何数据类型。 FreeRTOS+TCP TCP/IP 协议栈的实现提供了如何这样实现的实际例子。<br />在自己的任务中运行的 TCP/IP 协议栈必须处理许多来自不同源的事件。 不同的事件类型与不同类型和长度的数据相关联。 在 TCP/IP 任务之外发生的所有事件都由 IPStackEvent_t 类型的结构描述，并发送到队列上的 TCP/IP 任务。 IPStackEvent_t 结构如清单 55 所示，IPStackEvent_t 结构的 pvData 成员是一个指针，可用于直接保存值或指向缓冲区。
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
清单 55. 用于将事件发送到 FreeRTOS+TCP上的 TCP/IP 协议栈任务结构<br />​

TCP/IP 事件及其相关数据的示例包括:<br />​<br />

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
清单 58. 伪代码，显示如何使用 IPStackEvent_t 结构向 TCP 发送网络关闭事件<br />​

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

---

<a name="LXGfG"></a>
## 从多个队列接收
<a name="z9pIP"></a>
### 队列集
应用程序设计通常需要单个任务来接收不同大小的数据、不同含义的数据以及来自不同来源的数据。上一节演示了如何使用接收结构的单个队列以简洁高效的方式实现这一点。然而，有时应用程序的设计者正在使用限制其设计选择的约束，这就需要对某些数据源使用单独的队列。例如，集成到设计中的第三方代码可能假设存在专用队列。在这种情况下，可以使用 “队列集”。<br />​

队列设置允许所有任务从多个队列接收数据，而无需任务依次轮询每个队列以确定哪个队列(如果有)包含数据。<br />​

与使用接收结构的单个队列实现相同功能的设计相比，使用队列集从多个源接收数据的设计不那么整洁，效率也较低。因此，建议仅在设计约束绝对必要时才使用队列集。<br />​

以下部分描述了如何使用由以下人员设置的队列：<br />​<br />

1. 创建队列集。
1. 向集合中添加队列。信号量也可以添加到队列集中。信号量将在本书后面描述。
1. 从队列集中读取数据，以确定队列集中哪些队列包含数据。当作为队列集合成员的队列接收数据时，接收队列的句柄被发送到队列集合，当任务调用从队列集合读取的函数时返回。因此，如果从队列集中返回队列句柄，那么句柄引用的队列就知道包含数据，然后任务就可以直接从队列中读取。

​<br />
>
注意：如果队列是队列集的成员，那么不要从队列中读取数据，除非队列的句柄已经首先从队列集中读取。

​

队列集功能是通过在 FreeRTOSConfig.h 中将 configUSE_QUEUE_SETS 编译时配置常数设置为 1 来启用的。
<a name="uByGb"></a>
### xQueueCreateSet() API函数
必须先显式创建队列集，然后才能使用它。<br />​

队列集由句柄引用，句柄是 QueueSetHandle_t 类型的变量。xQueueCreateSet() API 函数创建一个队列集，并返回一个引用它创建的队列集的 QueueSetHandle_t。
```verilog
QueueSetHandle_t xQueueCreateSet(const UBaseType_t uxEventQueueLength);
```
清单 60. xQueueCreateSet() 函数原型<br />​

表 23. xQueueCreateSet() 参数与返回值

| 参数名 | 描述 |
| --- | --- |
| uxEventQueueLength | 当队列集合中的一个队列接收数据时，接收队列的句柄被发送到队列集合。 uxEventQueueLength 定义了正在创建的队列集在任何时候可以容纳的队列句柄的最大数量。<br />队列句柄仅在队列集中的队列接收数据时发送给队列集。如果队列已满，则无法接收数据，因此如果队列集中的所有队列都已满，则无法向队列集发送队列句柄。因此，队列集中一次必须容纳的最大项目数是该组中每个队列长度的总和。<br />例如，如果集合中有三个空队列，每个队列的长度为五，那么集合中的队列总共可以在集合中的所有队列都满之前接收十五个项目(三个队列乘以五个项目)。在该示例中，uxEventQueueLength 必须设置为 15，以保证队列集可以接收发送给它的每个项目。<br />信号量也可以添加到队列集中。二进制和计数信号量将在本书后面介绍。为了计算必要的uxEventQueueLength，二进制信号量的长度为 1，计数信号量的长度由信号量的最大计数值给出。<br />作为另一个例子，如果队列集将包含长度为 3 的队列和长度为 1 的二进制信号量，uxEventQueueLength 必须设置为 4 (3+1)。 |
| 返回值 | 如果返回空值，则无法创建队列集，因为空闲操作系统没有足够的堆内存来分配队列集数据结构和存储区域。<br />返回的非空值表示队列集已成功创建。返回值应该存储为创建的队列集的句柄。 |

xQueueAddToSet() 将队列或信号量添加到队列集中。信号量将在本书后面描述。
```verilog
BaseType_t xQueueAddToSet( QueueSetMemberHandle_t xQueueOrSemaphore, 
                           QueueSetHandle_t xQueueSet );
```
清单 61. xQueueAddToSet() API 函数原型<br />​

表 24. xQueueAddToSet() 参数与返回值

| 参数名 | 描述 |
| --- | --- |
| xQueueOrSemaphore | 正在添加到队列集中的队列或信号量的句柄。<br />队列句柄和信号量句柄都可以转换为 QueueSetMemberHandle_t 类型。 |
| xQueueSet | 要添加队列或信号量的队列集的句柄。 |
| 返回值 | 有两种可能的返回值:<br />1. 1.pdPASS: 只有当队列或信号量成功添加到队列集中时，才会返回 pdPASS。<br />1. 2.pdFAIL: 如果队列或信号量无法添加到队列集中，将返回 pdFAIL。队列和二进制信号量只有在为空时才能添加到集合中。计数信号量只能在其计数为零时添加到集合中。队列和信号量一次只能是一个集合的成员。<br /> |

xQueueSelectFromSet() 从队列集中读取队列句柄。<br />​

当作为集合成员的队列或信号量接收数据时，接收队列或信号量的句柄被发送到队列集合，并在任务调用 xQueueSelectFromSet() 时返回。如果从对 xQueueSelectFromSet() 的调用中返回句柄，则句柄引用的队列或信号量已知包含数据，然后调用任务必须直接从队列或信号量中读取。<br />​<br />
>
注意: 除非队列或信号量的句柄是从对 xQueueSelectFromSet() 的调用中首先返回的，否则不要从属于集合成员的队列或信号量中读取数据。每次调用 xQueueSelectFromSet() 返回队列句柄或信号量句柄时，只从队列或信号量中读取一个项目。

```verilog
QueueSetMemberHandle_t xQueueSelectFromSet( QueueSetHandle_t xQueueSet,
                                            const TickType_t xTicksToWait );
```
清单 62. xQueueSelectFromSet() API 函数原型<br />​

表 25. xQueueSelectFromSet() 参数与返回值

| 参数名 | 描述 |
| --- | --- |
| xQueueSet | 队列集的句柄，从中接收(读取)队列句柄或信号量句柄。对用于创建队列集的 xQueueCreateSet() 的调用将返回队列集句柄。 |
| xTicksToWait | 如果队列集中的所有队列和信号量都为空，则调用任务应保持在阻塞状态以等待从队列集中接收队列或信号量句柄的最长时间。如果 xTicksToWait 为零，那么如果集合中的所有队列和信号量都为空，xQueueSelectFromSet() 将立即返回。<br />阻塞时间以刻度周期指定，因此它表示的绝对时间取决于刻度频率。宏 pdMS_TO_TICKS() 可用于将以毫秒为单位指定的时间转换为以刻度为单位指定的时间。<br />将 xTicksToWait 设置为 portMAXDELAY 将导致任务无限期等待(无超时)，前提是在 FreeRTOSConfig.h 中将 INCLUDE_vTaskSuspend 设置为 1。 |
| 返回值 | 非空的返回值将是已知包含数据的队列或信号量的句柄。如果指定了阻塞时间(xTicksToWait 不为零)，那么调用任务可能被置于阻塞状态，以等待数据从集合中的队列或信号量变得可用，但是在阻塞时间到期之前，从队列集合中成功读取了句柄。句柄以 QueueSetMemberHandle_t 类型返回，可以转换为 QueueHandle_t 类型或 SemaphoreHandle_t 类型。<br />如果返回值为空，则无法从队列集中读取句柄。如果指定了阻塞时间(xTicksToWait 不为零)，则调用任务将被置于阻塞状态，以等待另一个任务或中断向集合中的一个或多个信号量发送数据，但阻塞时间在此之前已经过期。 |

<a name="jQsHe"></a>
### 示例12. 使用一个队列集
本示例创建两个发送任务和一个接收任务。发送任务通过两个独立的队列向接收任务发送数据，每个任务一个队列。这两个队列被添加到队列集中，接收任务从队列集中读取以确定这两个队列中的哪一个包含数据。<br />​

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
清单 63. 示例 12 main() 实现<br />​

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
清单 64. 示例 12 中使用的发送任务<br />发送任务写入的队列是同一队列集的成员。每次任务发送到其中一个队列时，队列的句柄都会被发送到队列集中。接收任务调用 xQueueSelectFromSet() 从队列集中读取队列句柄。接收任务从集合中接收到队列句柄后，它知道接收句柄引用的队列包含数据，因此直接从队列中读取数据。它从队列中读取的数据是指向字符串的指针，接收任务将打印出该字符串。<br />​

如果对 xQueueSelectFromSet() 的调用超时，则它将返回空值。在示例 12 中，xQueueSelectFromSet() 是以不确定的块时间调用的，因此永远不会超时，并且只能返回有效的队列句柄。因此，在使用返回值之前，接收任务不需要检查 xQueueSelectFromSet() 是否返回空值。<br />​

xQueueSelectFromSet() 只有在句柄引用的队列包含数据时才会返回队列句柄，因此从队列中读取时没有必要使用块时间。<br />​

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
清单 65. 示例 12 使用的接受任务<br />​

图 37 显示了示例 12 产生的输出。可以看出，接收任务从两个发送任务接收字符串。vSenderTask1() 使用的阻塞时间是 vSenderTask2() 使用的块时间的一半，导致 vSenderTask1() 发送的字符串打印频率是 vSenderTask2() 发送的字符串的两倍。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636012137223-8409cebc-07e8-4a46-ab17-ccbebd5462e6.png#clientId=udb212963-d539-4&from=paste&id=ud2b6bd14&margin=%5Bobject%20Object%5D&name=image.png&originHeight=656&originWidth=1316&originalType=url&ratio=1&size=240782&status=done&style=none&taskId=ub25bbef3-0ffb-4ef9-9263-55f8fa06769)<br />图 37. 执行示例 12 时产生的输出
<a name="eW6X9"></a>
### 更实际的队列集用例
例 12 展示了一个非常简单的例子；队列集只包含队列，它包含的两个队列都用于发送字符指针。在真实的应用程序中，队列集可能包含队列和信号量，并且队列可能不都包含相同的数据类型。在这种情况下，在使用返回值之前，有必要测试 xQueueSelectFromSet() 返回的值。清单 66 演示了当集合具有以下成员时，如何使用从 xQueueSelectFromSet() 返回的值:<br />​<br />

1. 二进制信号量。
1. 从中读取字符指针的队列。
1. 从中读取 uint32_t 值的队列。


<br />清单 66 假设队列和信号量已经被创建并添加到队列集中。
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

---

<a name="D4BdF"></a>
## 使用队列创建邮箱
嵌入式社区内部对术语没有共识，“邮箱”在不同的实时操作系统中将有不同的含义。在这本书里，术语邮箱是指一个长度为1的队列。队列之所以被描述为邮箱，是因为它在应用程序中的使用方式，而不是因为它与队列的功能不同:<br />​<br />

- 队列用于将数据从一个任务发送到另一个任务，或者从中断服务例程发送到一个任务。发送方在队列中放置一个条目，接收方从队列中移除该条目。数据通过队列从发送方传递到接收方。
- 邮箱用于保存任何任务或任何中断服务例程都可以读取的数据。数据不会通过邮箱，而是保留在邮箱中，直到被覆盖。发件人会覆盖邮箱中的值。接收器从邮箱中读取该值，但不从邮箱中删除该值。


<br />本章描述了允许队列用作邮箱的两个队列应用编程接口函数。<br />​

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
<a name="Za7Zf"></a>
### xQueueOverwrite() API 函数
像 xQueueSendToBack() 应用编程接口函数一样，xQueueOverwrite() 应用编程接口函数向队列发送数据。与 xQueueSendToBack() 不同，如果队列已经满了，那么 xQueueOverwrite() 将覆盖队列中已经存在的数据。<br />xQueueOverwrite() 只能用于长度为1的队列。这种限制避免了函数的实现在队列已满时任意决定要覆盖队列中的哪个项目。<br />​<br />
>
注意: 永远不要从中断服务例程调用 xQueueOverwrite()。应该使用中断安全版本xQueueOverwriteFromISR() 来代替它。

```verilog
BaseType_t xQueueOverwrite( QueueHandle_t xQueue, const void * pvItemToQueue );
```
清单 68. xQueueOverwrite() API 函数原型<br />​

表 26. xQueueOverwrite() 参数和返回值

| 参数名/返回值 | 描述 |
| --- | --- |
| xQueue | 数据被发送(写入)到的队列的句柄。队列句柄将从用于创建队列的 xQueueCreate() 调用中返回。 |
| pvItemToQueue | 指向要复制到队列中的数据的指针。<br />队列可以容纳的每个项目的大小是在创建队列时设置的，因此这些字节将从 pvItemToQueue 复制到队列存储区域。 |
| 返回值 | xQueueOverwrite() 将写入队列，即使队列已满，因此 pdPASS 是唯一可能的返回值。 |

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
<a name="HvS1A"></a>
### xQueuePeek() API 函数
xQueuePeek() 用于从队列中接收(读取)项目，而不从队列中移除项目。xQueuePeek() 从队列头接收数据，而不修改存储在队列中的数据或数据在队列中的存储顺序。<br />​<br />
>
注意: 永远不要从中断服务例程调用 xQueuePeek()。应该使用中断安全版本 xQueuePeekFromISR() 来代替它。

xQueuePeek() 具有与 xQueueReceive() 相同的函数参数和返回值。
```verilog
BaseType_txQueuePeek( QueueHandle_t xQueue,
                      void *const pvBuffer,
                      TickType_t xTicksToWait );
```
清单 70. xQueuePeek() API 函数原型<br />​

清单 71 显示了 xQueuePeek() 用于接收发布到清单 69 中邮箱(队列)的项目<br />​<br />
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

---
