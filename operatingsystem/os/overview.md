> [操作系统原理](https://item.jd.com/12397357.html)

## 主要功能

为了高效地使用计算机软、硬件资源，提高计算机系统资源的利用率和方便用户使用，在计算机系统中都采用多道程序设计技术。多道程序设计技术是指内存中同时放入多道程序（进程）交替运行并共享系统资源，当一道程序（进程）由于某种原因（如输入/输出请求）而暂停执行时，CPU立即转去执行另一道程序（进程）；这样，不仅使CPU得到充分利用，而且也提高了 I/O设备和内存的利用率。在引入了多道程序设计技术后，使得操作系统具有多道程序同时（进程）运行且宏观上并行、微观上串行的特点，而操作系统也正是随着多道程序设计技术的出现而逐步发展起来的。

1、处理器管理（进程管理）

处理器管理主要是指对计算机系统中央处理器（CPU）的管理，其主要任务是对CPU进行分配，并对其运行进行有效地控制与管理。

为了提高计算机的利用率，操作系统采用了多道程序技术。为了描述多道程序的并发执行引入了进程的概念，进程可看作正在执行的程序，通过进程管理来协调多道程序之间的关系，以使 CPU资源得到最充分的利用。在多道程序环境下，CPU的分配与运行是以进程为基本单位的。随着并行处理技术的发展，为了进一步提高系统并行性，使并发执行单位的粒度更细以降低并发执行的代价，操作系统又引入了线程（Thread）的概念。对CPU的管理和调度最终归结为对进程和线程的管理和调度，它的主要功能包括进程控制和管理、进程同步与互斥、进程通信、进程死锁、线程控制和管理以及CPU调度。

2、存储管理（内存管理）

存储管理是指对内存空间的管理。程序要运行就必须由外存装入内存，当多道程序被装入内存共享有限的内存资源时，存储管理的主要任务就是为每道程序分配内存空间，使它们彼此隔离互不干扰，尤其是当内存不够用时，要通过虚拟技术来扩充物理内存，把当前不运行的程序和数据及时调出内存，需要运行时再将其由外存调入内存。存储管理的主要功能包括内存分配、内存保护、地址变换和内存扩充。

3、设备管理

设备管理是指计算机中除CPU和内存之外的所有输入输出设备（I/O设备，也称外部设备，简称外设）的管理。其首要任务是为这些设备提供驱动程序或控制程序，以便用户不必详细了解设备及接口的细节就可以方便地对设备进行操作。设备管理的另一个任务就是通过中断技术、通道技术和缓冲技术使外部设备尽可能与 CPU并行工作，以提高设备的使用效率。为了完成这些任务，设备管理的主要功能包括外部设备的分配与释放、缓冲区管理、共享型外部设备的驱动调度、虚拟设备等。

4、文件管理

文件是计算机系统中除CPU、内存、外部设备等硬件设备之外的另一类资源，即软件资源。程序和数据以文件的形式存放在外存储器（如磁盘、光盘、磁带、优盘）上，需要时再把它们装入内存。文件管理系统的主要任务是有效地组织、存储和保护文件，使用户方便、安全地访问它们。文件管理的主要功能包括文件存储空间管理、文件目录管理、文件存取控制和文件操作等。

5、用户接口

为了方便用户使用，操作系统向用户提供了使用接口。接口通常以命令、图形和系统调用等形式呈现给用户，前两种形式供用户通过键盘、鼠标或屏幕操作，后一种形式供用户在编程时使用。用户接口的主要功能包括命令接口管理、图形接口管理（图形实际上是命令的图形化表现形式）和程序接口管理。用户通过这些接口能方便地调用操作系统功能。

6、网络与通信管理

随着计算机网络的迅速发展，网络功能已成为操作系统的重要组成部分。现代操作系统都注重为用户提供便捷、可靠的网络通信服务。为此，网络操作系统至少应具有以下3种管理功能。

（1）网络资源管理。计算机联网的主要目的之一是共享资源，网络操作系统应能够实现网上资源共享，管理用户程序对资源的访问，保证网络信息资源的安全性和完整性。

（2）数据通信管理。计算机联网后，站点之间可以互相传送数据。数据通信管理为网络应用提供必要的网络通信协议，处理网络信息传输过程中的物理细节，同时通过通信软件，按照网络通信协议完成网络上计算机之间的信息传输。

（3）网络管理。包括网络性能管理、网络安全管理、网络故障管理、网络配置管理和日志管理等。

## 基本特征

1、并发性

并发性（Concurrence）是操作系统最重要的特征。并发性是指两个或两个以上的事件或活动在同一时间间隔内发生（注意，不是同一时刻）。也就是说，在计算机系统中同时存在多个进程，从宏观上看，这些进程是同时运行并向前推进的；从微观上讲，任何时刻只能有一个进程执行，如果在单CPU条件下，那么这些进程就是在CPU上交替执行的。

操作系统的并发性能够有效地改善系统资源的利用率，提高系统的效率。例如，一个进程等待 I/O时，就让出 CPU，并由系统调度另一个进程占用 CPU运行。这样，在一个进程等待I/O时，CPU就不会空闲，这就是并发技术。

操作系统的并发性会使操作系统的设计和实现变得更加复杂。例如，以何种策略选择下一个可执行的进程？怎样从正在执行的进程切换到另一个等待执行的进程？如何将内存中各交替执行的进程隔离开来，使之互不干扰？怎样让多个交替执行的进程互通消息并协作完成任务？如何协调多个交替执行的进程对资源的竞争？多个交替执行的进程共享文件数据时，如何保证数据的一致性？为了更好地解决这些问题，操作系统必须具有控制和管理进程并发执行的能力，必须提供某种机制和策略进行协调，从而使各并发进程能够顺利推进并获得正确的运行结果。此外，操作系统要充分发挥系统的并行性，就要合理地组织计算机的工作流程，协调各类软、硬件资源的工作，充分提高资源的利用率。

2、共享性

共享性（Sharing）是操作系统的另一个重要特征。在内存中并发执行的多个进程可以共同使用系统中的资源（包括硬件资源和信息资源）。资源共享的方式可以分为以下两种。

（1）互斥使用方式

互斥使用方式是指当一个进程正在使用某种资源时，其他欲使用该资源的进程必须等待，仅当这个进程使用完该资源并释放后，才允许另一个进程使用这个资源，即它们只能互斥地共享该资源，因此这类资源也称互斥资源。系统中的有些资源，如打印机、磁带机的使用就只允许互斥使用。

（2）同时使用方式

系统中有些资源允许在同一段时间内被多个进程同时使用，这里的“同时”是宏观意义上的。典型的可供多个进程同时使用的资源是磁盘，可重入程序（可以供多个用户同时运行的程序）也是可被同时使用的，如编译程序。

共享性和并发性是操作系统两个最基本的特征，它们互为依存。一方面，资源的共享是因为程序的并发执行而引起的，若系统不允许程序并发执行，自然也就不存在资源共享的问题。另一方面，如果系统不能对资源共享实施有效的管理，则必然会影响到程序的并发执行，甚至程序无法并发执行，操作系统也就失去了并发性，导致整个系统的效率低下。

3、虚拟性

虚拟性（Virtual）的本质含义是指将一个物理实体映射为多个逻辑实体。前者是实际存在的；后者是虚拟的，是一种感觉性的存在。例如，在单 CPU系统中虽然只有一个CPU存在，且每一时刻只能执行一道程序，但操作系统采用了多道程序技术后，在一段时间间隔内，从宏观上看有多个程序在运行，给人的感觉好像是有多个 CPU在支持每一道程序运行。这种情况就是将一个物理的CPU虚拟为多个逻辑的CPU。

4、不确定性

在多道程序设计环境下，不确定性（Nondeterminacy）主要表现在以下三个方面。

（1）在多道程序环境中，允许多个进程（程序）并发执行，但由于资源等因素的限制，每个进程的运行并不是“一气呵成”的，而是以“走走停停”的方式执行。内存中的每个进程何时开始执行，何时暂停，以什么速度向前推进，每个进程需要多少时间才能完成都是不可预知的。

（2）并发程序的执行结果也可能不确定，即对同一程序和同样的初始数据，其多次执行的结果可能是不同的。因此，操作系统必须解决这个问题，即保证在相同初始条件下，重复执行同一个程序时都不受运行环境的影响，而得到完全相同的结果。

（3）外部设备中断、I/O请求、程序运行时发生中断的时间等都是不可预测的。

## CPU工作模式

为了防止用户异常操作损坏操作系统功能，CPU工作状态分为用户态和内核态。

内核态是指操作系统程序运行的状态，在该状态下可以执行操作系统的所有指令（包括特权指令），并能够使用系统的全部资源。用户态是指用户程序运行的状态，在该状态下所能执行的指令和访问的资源都将受到限制。

操作系统通过程序状态字（PSW）来区分CPU的不同状态

中断和异常是 CPU 状态从用户态转换到内核态的唯一途径。用户程序在执行中发生中断或异常时，CPU响应中断或异常并交换程序状态字（暂停用户程序的执行，准备执行相应的中断处理程序），此时会导致 CPU的状态从用户态转换为内核态。而中断或异常处理的程序（中断处理程序）则运行在内核态下。

在操作系统层面上，通常关心的是内核态和用户态的软件实现和切换。当 CPU处于内核态时，可以通过修改程序状态字（PSW）直接进入用户态运行。当 CPU处于用户态时，如果需要切换到内核态，则一般是通过访管指令或系统调用来实现。访管指令或系统调用是一条具有中断性质的特殊机器指令，用户程序使用它们通过中断进入内核态来运行操作系统程序完成指定的操作。访管指令或系统调用的主要功能表现在以下三个方面。

① 通过中断实现从用户态到内核态的改变。

② 在内核态下由操作系统代替用户完成其请求（用户指定的操作）。

③ 操作系统完成指定操作后再通过修改程序状态字（PSW）由内核态切换回用户态。

## 中断技术

中断（Interrupt）机制是实现多道程序并发执行的重要条件，也是现代计算机系统的重要组成部分。中断是指程序在执行过程中 CPU对系统发生的某个事件做出的一种反应。中断具有以下三个特点。

（1）中断是随机的。

（2）中断是可恢复的。

（3）中断是自动处理的。

中断发生时，CPU暂停正在执行的程序并保存其运行现场信息，然后自动转去执行相应的中断处理程序，中断处理完成后返回被中断程序的断点处（同时恢复为被中断程序所保存的现场信息）继续执行或者重新调度其他程序执行。在计算机系统中引入中断的目的主要有两个：一是解决CPU与I/O设备的并行工作问题；二是实现实时控制。中断在计算机系统中的应用越来越广泛，它可以用来处理系统中的任何突发事件，如请求系统服务、程序出错、硬件故障和网络通信等。用户程序执行系统调用，或者出现 I/O通道和设备产生的内部和外部事件时，都需要通过中断机制产生中断信号来启动系统内核工作。

**分类**

![image-20201028224308574](https://gitee.com/p8t/picbed/raw/master/imgs/20201028224309.png)

（1）外中断。通常指硬中断，指来自 CPU之外的外部设备通过硬件请求方式产生的中断，如被 I/O设备触发的中断。外中断是一种强迫性中断，主要包括外部中断、I/O中断等。由于外中断是由随机发生的异步事件引起的，它与 CPU当前正在执行的任务无关，因此通常也称为异步中断。
外中断可以发生在用户态，也可以发生在内核态。不同的外中断通常具有不同的优先级，当 CPU处理高一级的中断时，会部分或全部屏蔽较低优先级的中断，外中断一般在两条机器指令之间响应。外中断又可分为不可屏蔽中断和可屏蔽中断。不可屏蔽中断在当前指令执行结束后，就会无条件立即予以响应；可屏蔽中断则根据中断允许标志位是否允许来决定CPU是否响应中断。可屏蔽中断通常用于CPU与外设之间的数据交换。

（2）内中断。通常也称为异常（Exception），是由 CPU在执行指令过程中检测到一个或多个错误或异常条件引起的，用于表示CPU执行指令时本身出现算术溢出、0作除数、取数时的奇偶校验错、访存（访问内存）指令越界或执行了一条所谓的“异常指令”（用于实现系统调用）等情况。这时中断当前执行的程序，转到相应的错误处理程序或异常处理程序。最早的中断和异常并没有区分，随着中断发生原因和处理方式差别的越发明显，才有了中断和异常的区分。内中断由 CPU中控制器里的硬件中断装置产生，中断处理程序所提供的服务是当前运行程序所需要的，如内存访问错误、单步调试和被 0除等。内中断处理程序在当前运行进程（程序）的上下文中执行。

内中断不可屏蔽，一旦发生必须立即响应处理。内中断通常发生在用户态，允许在指令执行期间进行响应并且允许多次响应。

根据中断事件的性质，可以将内中断划分为两大类：强迫性中断和自愿性中断。

① 强迫性中断。与当前运行程序的执行完全异步，中断是由随机事件和外部请求所引发的，引起强迫性中断的事件不是当前运行程序所期待的。强迫性中断事件主要包括硬件故障中断、程序性中断、外部中断和I/O中断等。

② 自愿性中断。用户程序在运行过程中请求操作系统为其提供某种功能服务，通过执行一条访管指令而引起的中断，称为“访管中断”或“陷阱”（Trap）。常见的自愿性中断有创建进程、分配内存、打开文件、信号量操作、发送或接收消息等。自愿性中断事件是当前运行程序所期待的，是用户在程序中有意安排的中断，它表示用户程序对操作系统的某种需求。当用户需要操作系统提供某种服务时，就使用访管指令或系统调用产生自愿性中断来达到需求的目的。

访管中断是通过执行访管指令而产生的中断。用户程序执行时，如果执行的是访管指令就产生一个访管中断，并通过陷入指令将 CPU的状态从用户态转换为内核态，然后将处理权移交给操作系统中的一段特殊代码（系统调用子程序），这一过程称为陷入。访管指令在用户态下执行，它包括两个部分：操作码和访管参数。操作码指出该指令是访管指令；访管参数则描述具体的访管要求。不同的访管参数对应不同的中断服务要求，就像机器指令的操作码一样，操作系统通过分析访管指令中的访管参数，调用相应的系统调用子程序为用户服务，系统调用功能完成后再返回用户态继续运行用户程序。

不同类型的中断具有不同的优先级。一般情况下，上述各种中断优先级从高到低的顺序依次为：硬件故障中断、自愿性中断、程序性中断、外部中断和I/O中断。

大部分内中断是由软件方法产生的，通过软件方法来模拟硬件中断，以实现宏观上异步执行效果的中断称为软中断，如“信号量”是一种软中断机制，是操作系统内核（或其他进程）对某个进程的中断。Windows系统中由内核发出用于启动线程调度、延迟过程调用的Dispatch/DPC和APC等中断也是软中断的典型应用。

## 系统调用

程序的运行空间分为用户空间和内核空间，在逻辑上它们之间是相互隔离的。因此在用户空间（用户态）运行的用户程序不能直接访问内核空间（内核态）的内核数据，也无法直接调用执行在内核空间的内核函数；否则，系统的安全性和可靠性就得不到保证。系统调用是操作系统提供给用户程序的特殊接口，用户程序通过系统调用通知系统自己需要执行内核空间的内核函数。此时由系统实现用户态到内核态的切换，并代替用户程序在内核空间执行内核函数及访问内核数据，待执行完内核函数后再由系统完成内核态到用户态的切换，从而继续执行用户程序。为保护系统内核数据不被破坏，同时又为用户程序提供一个良好的运行环境，操作系统内核提供了一组具有特定功能的内核函数，并通过一组称为系统调用的接口呈现给用户使用。

系统调用通常也称为“陷阱”，它的执行过程与普通函数的调用很相似，但用户态运行的程序只有通过系统调用才能进入操作系统内核，而普通函数调用则由函数库或用户自己提供且只能运行于用户态。