<a name="YO8Tl"></a>
# **软件定时器管理**

---

<a name="gjZCu"></a>
## 章节介绍和范围
软件计时器用于调度功能在将来的设定时间执行，或以固定频率定期执行。由软件定时器执行的函数称为软件定时器的回调函数。<br />软件计时器由FreeRTOS内核实现，并受FreeRTOS内核的控制。它们不需要硬件支持，也与硬件计时器或硬件计数器无关。<br />请注意，根据FreeRTOS使用创新设计以确保最高效率的理念，软件计时器不会使用任何处理时间，除非软件计时器回调函数实际正在执行。<br />软件计时器功能是可选的。要包括软件计时器功能，请执行以下操作:

1. 将FreeRTOS源文件FreeRTOS/Source/timers.c构建为项目的一部分 
1. 在FreeRTOSConfig.h中将configUSE_TIMERS设置为1。
<a name="lVjwM"></a>
### 范围
本章旨在让读者更好地了解以下内容：<br />​<br />

- 软件定时器的特性与任务特性的比较。
- RTOS后台任务。
- 计时器命令队列。
- 单次软件定时器和周期性软件定时器之间的区别。
- 如何创建、启动、重置和更改软件计时器的周期。

---

<a name="XwWDH"></a>
## 软件定时器回调函数
软件计时器回调函数被实现为C函数。它们唯一的特别之处是它们的原型，它必须返回void，并将软件计时器的句柄作为其唯一的参数。清单72演示了回调函数原型。
```verilog
void ATimerCallback( TimerHandle_t xTimer );
```
清单72.软件计时器回调函数原型<br />​

软件计时器回调函数自始至终执行，并以正常方式退出。它们应该保持简短，并且不能进入阻塞状态。<br />​<br />
:::tips
注意：正如将看到的，软件计时器回调函数在启动FreeRTOS调度程序时自动创建的任务的上下文中执行。 因此，软件计时器回调函数决不能调用会导致调用任务进入阻塞状态的FreeRTOS API函数，这一点至关重要。 可以调用xQueueReceive()之类的函数，但前提是该函数的xTicksToWait参数(指定函数的阻塞时间)设置为0。 调用vTaskDelay()之类的函数是不对的，因为调用vTaskDelay()会始终将调用任务置于阻塞状态。
:::

---

<a name="AGQ9Y"></a>
## 软件计时器的属性和状态
<a name="s5ITX"></a>
### 软件计时器的周期
软件计时器的‘周期’是软件计时器启动和软件计时器的回调函数执行之间的时间
<a name="dsXSq"></a>
### 单次计时器和自动重新加载计时器
有两种类型的软件计时器：<br />​<br />

1. 单次计时器 一旦启动，一次性定时器将只执行其回调函数一次。一次性计时器可以手动重新启动，但不会自行重新启动。
1. 自动重新加载计时器 一旦启动，自动重新加载计时器将在每次到期时重新启动，从而定期执行其回调函数。


<br />图38显示了单次定时器和自动重新加载定时器之间的行为差异。虚线垂直线标记计时中断发生的时间。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705888-f6634080-ebab-4eda-aaca-bf13cc26a399.png#clientId=udb212963-d539-4&from=paste&id=u55d2f1b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=744&originalType=url&ratio=1&size=83139&status=done&style=none&taskId=u9c025a60-9629-4223-83bc-68fe1bc28fa)<br />图38一次性软件计时器和自动重新加载软件计时器之间的行为差异<br />参考图38：

- 计时器1 定时器1是具有6个滴答周期的一次性定时器。它在时间t1启动，因此它的回调函数在6个刻度之后，即时间t7执行。由于定时器1是一次性定时器，其回调函数不会再次执行。
- 计时器2 定时器2是具有5个滴答周期的自动重新加载定时器。它在时间t1启动，因此它的回调函数在时间t1之后每5个节拍执行一次。在图38中，这是时间t6、t11和t16。
<a name="r8lbk"></a>
### 软件计时器状态
软件计时器可以处于以下两种状态之一：

- 休眠：存在休眠的软件计时器，可以由其句柄引用，但不在运行，因此其回调函数将不会执行
- 运行：正在运行的软件定时器，将在自该软件定时器进入运行状态，或自该软件定时器上次被重置以来经过与其周期相等的时间之后执行其回调功能。

图39和图40分别显示了自动重新加载定时器和单次定时器在休眠和运行状态之间可能的转换。这两个图的关键区别在于定时器到期后进入的状态；自动重新加载定时器执行其回调函数，然后重新进入运行状态，一次性定时器执行其回调函数，然后进入休眠状态。<br />xTimerDelete()接口函数的作用是：删除计时器。可以随时删除计时器。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705902-99800f84-ae67-440b-90c9-3310cac40bce.png#clientId=udb212963-d539-4&from=paste&id=uef367f6a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=397&originWidth=829&originalType=url&ratio=1&size=72281&status=done&style=none&taskId=uc68bcfd4-0298-4ceb-b294-8ca03b03473)<br />图39自动重新加载软件计时器状态和转换<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013705551-9f9c00ea-ec01-4ccb-be7d-55253a1a817f.png#clientId=udb212963-d539-4&from=paste&id=u9fcd7fe3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=368&originWidth=766&originalType=url&ratio=1&size=72037&status=done&style=none&taskId=uf8dcfb00-f482-4327-bdde-046959723f1)<br />图40一次性软件定时器状态和转换

---

<a name="nMNiJ"></a>
## 软件定时器的上下文
<a name="dbUH3"></a>
### RTOS守护(计时器服务)任务
所有软件计时器回调函数都在同一RTOS守护进程(或“计时器服务”)任务的上下文中执行[1]。<br />_[1]. 该任务过去被称为“计时器服务任务”，因为最初它只用于执行软件计时器回调函数。现在同一任务也用于其他目的，因此它被称为“RTOS守护程序任务”的更一般的名称。_<br />守护程序任务，是在启动调度程序时，自动创建的标准FreeRTOS任务。其优先级和堆栈大小分别由configTIMER_TASK_PRIORITY和configTIMER_TASK_STACK_DEPTH编译时间配置常量设置。这两个常量都在FreeRTOSConfig.h中定义。<br />软件计时器回调函数不得调用会导致调用任务进入阻塞状态的FreeRTOS API函数，否则将导致守护程序任务进入阻塞状态。
<a name="Ur2QN"></a>
### 计时器命令队列
软件计时器API函数将命令从调用任务发送到称为“计时器命令队列”的队列上的守护程序任务。这如图41所示。命令的例子包括“启动定时器”、“停止定时器”和“重置定时器”。<br />计时器命令队列是在启动调度程序时自动创建的标准FreeRTOS队列。定时器命令队列的长度由FreeRTOSConfig.h中的configTIMER_QUEUE_LENGTH编译时间配置常量设置。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974388-3c1f11a9-dd13-4d5e-b740-fd915ce0b6a0.png#clientId=udb212963-d539-4&from=paste&id=u07a448be&margin=%5Bobject%20Object%5D&name=image.png&originHeight=406&originWidth=707&originalType=url&ratio=1&size=142166&status=done&style=none&taskId=u161deb5a-be29-477e-8d12-eb36b9d460a)<br />图41 软件定时器API函数使用定时器命令队列与RTOS守护程序任务通信
<a name="WpnrZ"></a>
### 守护进程任务调度
守护程序任务与任何其他FreeRTOS任务一样进行调度；当守护程序任务是能够运行的最高优先级任务时，它只会处理命令或执行计时器回调函数。图42和图43演示了configTIMER_TASK_PRIORITY设置如何影响执行模式<br />​

图42显示了当守护程序任务的优先级低于调用 xTimerStart() API函数的任务的优先级时的执行模式<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974334-7e306f7b-1f83-4107-85be-dee306722f04.png#clientId=udb212963-d539-4&from=paste&id=ubf105cdd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=278&originWidth=571&originalType=url&ratio=1&size=56585&status=done&style=none&taskId=ubf4dace8-5a45-428f-b390-2c996b332ac)<br />图42 调用xTimerStart()的任务的优先级高于守护程序任务的优先级时的执行模式<br />参照图42，其中任务1的优先级高于守护程序任务的优先级，并且守护程序任务的优先级高于空闲任务的优先级：<br />​<br />

1. t1时刻 ：任务1处于RUNNING状态，守护程序任务处于BLOCKED状态。 守护程序任务将脱离阻塞状态，如果一个命令被发送到计时器命令队列，在这种情况下，它将处理命令。或者如果软件计时器超时，在这种情况下，它将执行软件计时器的回调函数。
1. t2时刻：任务1调用xTimerStart()。 xTimerStart()向计时器命令队列发送命令，使守护程序任务离开阻塞状态。 任务1的优先级高于守护程序任务的优先级，因此守护程序任务不会抢占任务1。 任务1仍处于Running状态，守护程序任务已离开BLOCKED状态，进入READY状态。
1. t3时刻：任务1完成 xTimerStart() API函数的执行。 任务1从函数开始到函数结束执行xTimerStart()，而不离开运行状态。
1. t4时刻：任务1调用导致其进入阻塞状态的API函数。守护程序任务现在是处于就绪状态的最高优先级任务，因此调度程序选择守护程序任务作为进入运行状态的任务。然后，守护程序任务开始处理任务1发送到计时器命令队列的命令。 _注意：正在启动的软件计时器将到期的时间，是从向计时器命令队列发送“启动计时器”命令开始计算的，而不是从守护程序任务从计时器命令队列接收到“启动计时器”命令的时间计算的。_
1. t5时刻：守护程序任务已完成对任务1发送给它的命令的处理，并尝试从计时器命令队列接收更多数据。计时器命令队列为空，因此守护程序任务重新进入阻塞状态。如果将命令发送到计时器命令队列，或者如果软件计时器超时，则守护程序任务将再次离开阻塞状态。空闲任务现在是处于就绪状态的最高优先级任务，因此调度程序选择空闲任务作为要进入运行状态的任务。

图43显示了类似于图42所示的场景，但是这一次守护程序任务的优先级高于调用xTimerStart()的任务的优先级。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636013974569-c3bbbfc2-83ff-4d93-9db7-9feb390934af.png#clientId=udb212963-d539-4&from=paste&id=uaf26d638&margin=%5Bobject%20Object%5D&name=image.png&originHeight=268&originWidth=401&originalType=url&ratio=1&size=53893&status=done&style=none&taskId=ub36444eb-2aa1-4a7b-baea-dfc11c1a224)<br />图43 当调用xTimerStart()的任务的优先级低于守护程序任务的优先级时的执行模式<br />
<br />参照图43，其中守护任务的优先级高于任务1的优先级，任务1的优先级高于空闲任务的优先级： 1. t1时刻 和之前一样，任务1正在运行态，守护任务在阻塞态。<br />​<br />

1. t2时刻 任务1调用xTimerStart()。 xTimerStart() 向计时器命令队列发送命令，使守护程序任务离开阻塞状态。守护程序任务的优先级高于任务1的优先级，因此调度器选择守护程序任务作为进入运行状态的任务。 任务1在完成执行 xTimerStart() 函数之前被守护程序任务抢占，现在处于就绪状态。守护程序任务开始处理任务1发送到定时器命令队列的命令。
1. t3时刻 守护程序任务已完成对任务1发送给它的命令的处理，并尝试从计时器命令队列接收更多数据。 计时器命令队列为空，因此守护程序任务重新进入阻塞状态。 任务1现在是处于就绪状态的最高优先级任务，因此调度程序选择任务1作为要进入运行状态的任务。
1. t4时刻 任务1在完成执行 xTimerStart() 函数之前被守护程序任务抢占，并且只有在重新进入运行状态后才退出(从)xTimerStart()。
1. t5时刻 任务1调用导致其进入阻塞状态的API函数。空闲任务现在是处于就绪状态的最高优先级任务，因此调度程序选择空闲任务作为要进入运行状态的任务。


<br />在图42所示的场景中，任务1向计时器命令队列发送命令与守护进程任务接收和处理命令之间经过了一段时间。在图43所示的场景中，在Task1从发送命令的函数返回之前，守护进程任务已经接收并处理了Task1发送给它的命令。<br />​

发送到计时器命令队列的命令包含时间戳。时间戳用于说明从应用程序任务发送的命令到守护程序任务正在处理的同一命令之间经过的任何时间。例如，如果发送“启动计时器”命令来启动周期为10个滴答的计时器，则时间戳用于确保计时器是在命令发送后10个滴答超时，而不是在命令被守护进程处理之后10个滴答超时。

---

<a name="D5OAl"></a>
## 创建和开始一个软件定时器
<a name="sSbb9"></a>
### xTimerCreate()API函数
FreeRTOS V9.0.0还包括xTimerCreateStatic()函数，该函数分配在编译时静态创建计时器所需的内存：软件计时器必须先显式创建，然后才能使用。<br />​

软件计时器由TimerHandle_t类型的变量引用。xTimerCreate()用于创建软件计时器，并返回TimerHandle_t以引用其创建的软件计时器。软件计时器在休眠状态下创建。<br />​

可以在调度器运行之前创建软件计时器，也可以在调度器启动后从任务创建软件计时器。第0节介绍了使用的数据类型和命名约定。
```verilog
TimerHandle_t xTimerCreate( const char * const pcTimerName, 
                                TickType_t xTimerPeriodInTicks, 
                                UBaseType_t uxAutoReload, 
                                void * pvTimerID, 
                                TimerCallbackFunction_t pxCallbackFunction );
```
清单73. xTimerCreate()API函数原型<br />​

表27. xTimerCreate()参数和返回值

| 参数/返回值 | 描述 |
| --- | --- |
| ​

pcTimerName | 计时器的描述性名称。FreeRTOS不会以任何方式使用它。它纯粹是作为调试辅助工具而包含的。使用人类可读的名称标识计时器比尝试通过其句柄标识要简单得多。 |
| ​

xTimerPeriodInTicks | 以刻度为单位指定的计时器周期。pdMS_TO_TICKS()宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| ​

uxAutoReload | 将uxAutoReload设置为pdTRUE以创建自动重新加载计时器。将uxAutoReload设置为pdFALSE以创建一次性计时器 |
| ​

​

pvTimerID | 每个软件定时器都有一个ID值。ID是一个空指针，应用程序编写器可以将其用于任何目的。当多个软件计时器使用相同的回调函数时，ID特别有用，因为它可用于提供特定于计时器的存储。<br />本章中的一个示例演示了计时器ID的使用。 pvTimerID设置正在创建的任务的ID的初始值。 |
| ​

pxCallbackFunction | 软件计时器回调函数仅仅是符合清单72中所示原型的C函数。pxCallbackFunction参数是指向要用作正在创建的软件计时器的回调函数的函数指针(实际上就是函数名)。 |
| ​

​

返回值 | 如果返回NULL，则无法创建软件计时器，因为FreeRTOS没有足够的堆内存来分配必要的数据结构。<br />返回的非NULL值表示软件计时器已成功创建。返回值是创建的计时器的句柄。 第2章提供了有关堆内存管理的更多信息。 |

<a name="nLpru"></a>
### xTimerStart()API函数
xTimerStart()用于启动处于休眠状态的软件定时器，或重置(重新启动)处于运行状态的软件定时器。xTimerStop()用于停止处于运行状态的软件计时器。停止软件计时器与将计时器转换到休眠状态相同。<br />可以在调度程序启动之前调用xTimerStart()，但是当这样做时，软件计时器直到调度程序启动时才会实际启动。
:::tips
注意：切勿从中断服务例程调用xTimerStart()。应该使用中断安全版本xTimerStartFromISR()来代替它。
:::
```verilog
BaseType_t xTimerStart( TimerHandle_t xTimer, TickType_t xTicksToWait );
```
清单74. xTimerStart() API函数原型<br />​

表28. xTimerStart() 参数和返回值

| 参数/返回值 | 描述 |
| --- | --- |
| xTimer | 正在启动或重置的软件计时器的句柄。句柄将从用于创建软件计时器的 xTimerCreate() 调用中返回。 |
| ​

​

​

​

​

​

​

xTicksToWait | xTimerStart() 使用计时器命令队列向守护程序任务发送“启动计时器”命令。<br />xTicksToWait 指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。<br />​

如果xTicksToWait为零且计时器命令队列已满，则xTimerStart()将立即返回。<br />​

 阻塞时间以滴答周期指定，因此它表示的绝对时间取决于滴答频率。宏pdMS_TO_TICKS()可用于将以毫秒为单位的时间转换为以滴答为单位的时间。 <br />​

如果FreeRTOSConfig.h中的include_vTaskSuspend设置为1，则。将xTicksToWait设置为portMAX_DELAY将导致调用任务无限期地保持在阻塞状态(没有超时)，以等待定时器命令队列中的空间变得可用。<br /> <br />如果在调度程序启动之前调用xTimerStart()，则会忽略xTicksToWait的值，并且xTimerStart()的行为就像xTicksToWait已设置为零一样。 |
| ​

​

​

​

​

​

​

​

返回值 | 有两个可能的返回值<br /> 1. pdPASS<br /> 只有当“启动定时器”命令成功发送到定时器命令队列时，才会返回pdPASS。 如果守护程序任务的优先级高于调用xTimerStart()的任务的优先级，那么调度程序将确保在xTimerStart()返回之前处理启动命令。这是因为一旦计时器命令队列中有数据，守护进程任务就会抢占调用xTimerStart()的任务。 <br />如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在块时间到期之前，数据已成功写入计时器命令队列。<br />2.pdFALSE 如果由于队列已满而无法将“启动定时器”命令写入定时器命令队列，则将返回pdFALSE。 <br />如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在计时器命令队列中腾出空间，但指定的阻塞时间在此之前已过期。 |

<a name="MQXQv"></a>
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
清单75. 创建并启动示例13中使用的计时器<br />​

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
清单77. 示例13中的自动重新加载计时器使用的回调函数<br />​

执行此示例将生成如图44所示的输出。图44显示了自动重新加载计时器的回调函数以500个滴答的固定周期执行(清单75中的mainAUTO_RELOAD_TIMER_PERIOD设置为500)，当滴答计数为3333时，一次性计时器的回调函数只执行一次(清单75中的MainOne_Shot_Timer_Period设置为3333)。<br />​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636016723478-770313b8-648a-490e-8a2d-10d63175c4eb.png#clientId=u77807233-20e3-4&from=paste&id=uea4f153f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=276&originWidth=557&originalType=url&ratio=1&size=74325&status=done&style=none&taskId=u0d70eacf-30dd-46bb-831f-bcfea2a3800)<br />图44 执行示例13时产生的输出

---

<a name="SxNcs"></a>
## 定时器ID
每个软件计时器都有一个ID，它是应用程序编写器可以出于任何目的使用的标记值。ID存储在空指针(void*)中，因此可以直接存储整数值、指向任何其他对象或用作函数指针。<br />创建软件计时器时会为ID分配初始值-之后可以使用vTimerSetTimerID() API函数更新ID，并使用pvTimerGetTimerID() API函数进行查询。<br />与其他软件计时器API函数不同，vTimerSetTimerID()和pvTimerGetTimerID()直接访问软件计时器-它们不向计时器命令队列发送命令。
<a name="B78k2"></a>
### vTimerSetTimerID()API函数
```verilog
void vTimerSetTimerID( const TimerHandle_t xTimer, void *pvNewID );
```
清单78. vTimerSetTimerID() API函数原型<br />​

表 29. vTimerSetTimerID() 参数

| 参数/返回值 | 描述 |
| --- | --- |
| xTimer | 使用新ID值更新的软件计时器的句柄。 句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| pvNewID | 将设置软件计时器ID的值。 |

<a name="nW7Rt"></a>
### pvTimerGetTimerID()API函数
```verilog
void *pvTimerGetTimerID( TimerHandle_t xTimer );
```
清单79 pvTimerGetTimerID() API函数原型<br />​

表30. pvTimerGetTimerID()参数和返回值

| 参数/返回值 | 描述 |
| --- | --- |
| xTimer | 正在查询的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| 返回值 | 正在查询的软件计时器的ID。 |

<a name="Xfm3t"></a>
### 示例14.使用回调函数参数和软件定时器ID
可以将相同的回调函数分配给多个软件计时器。完成后，回调函数参数用于确定哪个软件计时器过期。<br />​

示例13使用了两个单独的回调函数；一个回调函数由OneShot计时器使用，另一个回调函数由自动重新加载计时器使用。示例14创建与示例13创建的功能类似的功能，但将单个回调函数分配给两个软件计时器。<br />​

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
清单80. 创建示例14中使用的计时器<br />​

prvTimerCallback()将在任一计时器超时时执行。prvTimerCallback()的实现使用函数的参数来确定调用它是因为一次性计时器过期，还是因为自动重新加载计时器过期。<br />​

prvTimerCallback()还演示了如何将软件计时器ID用作特定于计时器的存储；每个软件计时器在其自己的ID中保存其过期次数的计数，并且自动重新加载计时器在第五次执行时使用该计数停止自身。<br />​

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
清单81. 示例14中使用的计时器回调函数<br />​

示例14产生的输出如图45所示。可以看到，自动重新加载计时器只执行五次。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636018371894-d9394688-5f72-4013-a729-fe5b572f9b11.png#clientId=u77807233-20e3-4&from=paste&id=u843a9c1b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=274&originWidth=550&originalType=url&ratio=1&size=42995&status=done&style=none&taskId=u3dee1b40-7b4f-46a9-baa4-984bade5482)<br />图45执行示例14时产生的输出

---

<a name="AbmYq"></a>
## 更改定时器的周期
每个官方FreeRTOS端口都提供了一个或多个示例项目。大多数示例项目都是在运行中不断自检，LED用于提供项目状态的可视反馈；如果自检总是通过，则LED缓慢闪烁，如果自检失败，则LED快速闪烁。<br />一些示例项目在任务中执行自检，并使用vTaskDelay()函数控制LED的切换速率。其他示例项目在软件计时器回调函数中执行自检，并使用计时器的周期来控制LED的切换速率。
<a name="qPnZ8"></a>
### xTimerChangePeriod()API函数
使用xTimerChangePeriod()函数更改软件计时器的周期。<br />​

如果xTimerChangePeriod()用于更改已在运行的计时器的周期，则该计时器将使用新的周期值重新计算其到期时间。重新计算的过期时间是相对于调用xTimerChangePeriod()的时间，而不是相对于最初启动计时器的时间。<br />​

如果使用xTimerChangePeriod()来更改处于休眠状态的计时器(未运行的计时器)的周期，则计时器将计算到期时间，并转换到运行状态(计时器将开始运行)。<br />​<br />
:::tips
注意：切勿从中断服务例程调用xTimerChangePeriod()。应该使用中断安全版本xTimerChangePerodFromISR()来代替它。
:::
```verilog
BaseType_t xTimerChangePeriod( TimerHandle_t xTimer,  
                               TickType_t xNewTimerPeriodInTicks, 
                               TickType_t xTicksToWait );
```
清单82. xTimerChangePeriod() API函数原型<br />​

表31 xTimerChangePeriod()参数和返回值

| 参数/返回值 | 描述 |
| --- | --- |
| xTimer | 需要更新的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| ​

xTimerPeriodInTicks | 软件计时器的新周期，以刻度为单位指定。pdMS_TO_TICKS() c宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| ​

​

xTicksToWait | xTimerChangePeriod()使用计时器命令队列向守护程序任务发送“change Period”命令。<br />xTicksToWait指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。如果xTicksToWait为零且计时器命令队列已满，则xTimerChangePeriod()将立即返回。 |
| ​

​

​

​

​

​

返回值 | 有两个可能的返回值<br /> 1.pdPASS <br />只有当数据成功发送到计时器命令队列时，才会返回pdPASS。<br />如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在阻塞时间到期之前，数据已成功写入计时器命令队列。<br />2.pdFALSE<br />如果由于队列已满而无法将‘Change Period’命令写入计时器命令队列，则将返回pdFALSE。<br />如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在队列中腾出空间，但指定的阻塞时间在此之前已过期。 |

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

---

<a name="UaEGk"></a>
## 重置一个定时器
重置软件计时器意味着重新启动计时器；计时器的超时时间相对于计时器重置的时间被重新计算，而不是根据计时器最初启动的时间。图46演示了这一点，它显示了一个以6为周期的计时器，在最终到期并执行他的回调函数之前，重启两次的过程。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636019663078-040714ed-3f00-4f97-b1c8-1b9b9a3fd5cf.png#clientId=u77807233-20e3-4&from=paste&id=ub3e74f1a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=378&originWidth=998&originalType=url&ratio=1&size=99144&status=done&style=none&taskId=ucdcd4cb5-043c-437e-9d4c-87dc49b3865)<br />图46. 启动和重置周期为6个滴答的软件计时器<br />参考图46：

- 定时器1在时间t1启动。它的周期为6，因此它执行回调函数的时间最初计算为T7，即启动后的6个滴答。
- 定时器1在到达时间T7之前，也就是在它到期并执行其回调函数之前被重置。定时器1在时间t5被重置，因此它将执行其回调函数的时间被重新计算为t11，即它被重置后的6个滴答。
- 定时器1在时间t11之前再次重置，因此在其到期并执行其回调函数之前再次复位。定时器1在时间t9被重置，因此它将执行其回调函数的时间被重新计算为t15，这是它上次被重置后的6个滴答。
- 定时器1不会再次复位，因此它在时间t15到期，并且相应地执行其回调函数。
<a name="mhUfL"></a>
### xTimerReset() API函数
使用xTimerReset()API函数重置计时器。<br />xTimerReset()还可用于启动处于休眠状态的计时器。
:::tips
注意：切勿从中断服务例程调用xTimerReset()。应该使用中断安全版本xTimerResetFromISR()来代替它。
:::
```verilog
BaseType_t xTimerReset( TimerHandle_t xTimer, TickType_t xTicksToWait );
```
清单84. xTimerReset() API函数原型<br />​

表32. xTimerReset()参数和返回值

| 参数/返回值 | 描述 |
| --- | --- |
| xTimer | 需要更新的软件计时器的句柄。句柄将从用于创建软件计时器的xTimerCreate()调用中返回。 |
| ​

xTimerPeriodInTicks | 软件计时器的新周期，以刻度为单位指定。pdms_to_ticks()宏可用于将以毫秒为单位指定的时间转换为以计时单位指定的时间。 |
| ​

​

​

​

xTicksToWait | xTimerChangePeriod()使用计时器命令队列向守护程序任务发送“重置”命令。xTicksToWait指定如果队列已满，调用任务应保持在阻塞状态以等待计时器命令队列上的空间变为可用的最长时间。<br />如果xTicksToWait为零且计时器命令队列已满，则xTimerReset()将立即返回。<br />如果FreeRTOSConfig.h中的include_vTaskSuspend设置为1，则将xTicksToWait设置为portMAX_DELAY将导致调用任务无限期地保持阻塞状态(没有超时)，以等待计时器命令队列中的空间变得可用。 |
| ​

​

​

​

​

​

返回值 | 有两种可能的返回值：<br />1. pdPASS<br />只有当数据成功发送到计时器命令队列时，才会返回pdPASS。 如果指定了阻塞时间(xTicksToWait不是零)，则在函数返回之前，调用任务可能被置于阻塞状态，以等待计时器命令队列中的空间变为可用，但在阻塞时间到期之前，数据已成功写入计时器命令队列。<br />1. pdFALSE<br />如果由于队列已满而无法将‘RESET’命令写入定时器命令队列，则将返回pdFALSE。 如果指定了阻塞时间(xTicksToWait不是零)，则调用任务将被置于阻塞状态，以等待守护进程任务在队列中腾出空间，但指定的阻塞时间在此之前已过期。<br /> |

<a name="wan0q"></a>
### 示例15.重置软件计时器
此示例模拟手机上的背光行为。背光：<br />​<br />

- 按下某个键时打开。
- 如果在特定时间段内按下更多键，则保持打开状态。
- 如果在特定时间段内没有按下键，则自动关闭。

​

使用一次性软件计时器来实现此行为：<br />​<br />

- 按下按键时打开[模拟]背光，在软件计时器的回调函数中关闭[模拟]背光。
- 每次按键按下，软件定时器重置。
- 因此，需要按下按键防止背光熄灭的时间等于软件定时器的周期；如果在计时器到期之前没有通过按键重置软件计时器，则执行计时器的回调功能，并且关闭背光。

​

xSimulatedBacklightOn变量保存背光状态。xSimulatedBacklightOn设置为pdTRUE表示背光打开，设置为pdFALSE表示背光关闭。<br />​

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
清单85. 示例15中使用的一次性计时器的回调函数<br />​

示例15创建一个任务来轮询键盘[1]。清单86显示了该任务，但是出于下一段中描述的原因，清单86并不是最佳设计的代表。<br />​

_[1].打印到Windows控制台和从Windows控制台读取密钥都会导致执行Windows系统调用。Windows系统调用，包括使用Windows控制台、磁盘或TCP/IP堆栈，可能会对FreeRTOS Windows端口的行为产生不利影响，通常应该避免。_<br />_​_

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
清单86. 示例15中用于重置软件计时器的任务<br />​

执行示例15时产生的输出如图47所示。参考图47：<br />​<br />

- 第一次按键发生在滴答计数为812的时候。当时打开了背光，启动了一次计时器。
- 当滴答计数为1813、3114、4015和5016时，出现了进一步的按键。所有这些按键都会导致计时器在计时器到期之前被重置。
- 计时器在滴答计数为10016时超时。当时背光是关着的。


<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/23129867/1636020021564-3b9b3a22-9894-4fdd-a45e-7891522494d2.png#clientId=u77807233-20e3-4&from=paste&id=u4bd3faea&margin=%5Bobject%20Object%5D&name=image.png&originHeight=538&originWidth=1080&originalType=url&ratio=1&size=138764&status=done&style=none&taskId=u8530c2e6-99c0-45d9-b6ff-e797a7e974d)<br />图47执行示例15时产生的输出<br />在图47中可以看到，计时器有5000个滴答的周期；在上次按下一个键之后恰好5000个滴答地关闭了背光，所以在最后一次重置计时器之后有5000个滴答。

---