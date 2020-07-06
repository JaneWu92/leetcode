Book:  
Computer Systems. A Programmer’s Perspective [3rd ed.]  
page 59
### binary vs ASCII
All files are at heart binary files -- that is, a collection of 1s and 0s.  
And ASCII is one of the way that use numbers to represent the number and letters.  
And it's all about how we view it.
Let's say we have different files which all contains "hello" text only.  
like: plain text file, rich text format file, word document, unicode plain text file.  
We can use a hex editor to see what is exactly the "binary collection" in these files.  
![alt text](pic_computer_systems/ASCII-Binary Text File.PNG)

### source code to executable binary file
![alt text](pic_computer_systems/souce-executable.PNG)

### Processors Read and Interpret Instructions Stored in Memory
souce code
```c
#include <stdio.h>
int main()
{
printf("hello, world\n");
return 0;
}
```
assembly code
```aidl
1 main:
2 subq $8, %rsp
3 movl $.LC0, %edi
4 call puts
5 movl $0, %eax
6 addq $8, %rsp
7 ret
```
![alt text](pic_computer_systems/hardware organization.PNG)

let's say we have a hello file which is compiled as executable file.  
and when we type "./hello" in bash and then click enter key, it will be executed and print out "hello world".  

### Process
Process it the abstraction of OS on CPU, main memory and IO devices.  
So it can provide the illusion that each program is the only one running on the system.  
And the context swith is the mechanism to concurrently run multiple processes.  
So the context involve, content of main memory, PC, register files.  

### OS heap & stack
stack: 
1. function variable(primitive values or reference to the objects in heap); function parameters; function return address
2. not need to allocate them yourself
3. LIFO(last in first out)

heap: other than stack, need to allocate.

### Process and thread
Difference:  
process is the smallest unit for OS to allocate resource.  
thread is the smallest unit for OS to execute tasks.  
One is for resource allocating and one is for tasks executing.  

what is the resource in process in allocating:  
registers, program counter, virtual memory address(stack, heap)

### Thread status
status: new, runnable, blocked, wait, timed_wait, terminated  
new -> runnable: thread.start() is called
runnable -> blocked: synchronized, lock.lock
runnable -> wait: object.wait 
runnable -> timed_wait: object.wait(time), sleep()

**blocked vs wait**
blocked is waiting for other thread to release the lock  
wait is waiting for other thread to notify it

**ReentrantLock doesn't use 'this' as object monitor**
```aidl
ReentrantLock lockA = new ReentrantLock();
lockA.lock();
lockA.wait(); //ERRPR. Will throw illegalMonitorStatException. Need synchronized here
```

**difference between object.wait vs unsafe.park**
no idea

### countdownlatch vs cyclicbarrier
countdownlatch.await: all threads blocks at count down to become 0 
cyclicbarrier.await: all threads blocks at count down to become 0, and then the cycic barrier will reset to original nunmber again

### cpu, core, hyper-thread
cpu can have several cores  
core: its own ALU, registers, PC  
hyper-thread: one ALU and several registers + PC(so that don't need the context switch)

### @? JVM本地变量表和CPU的寄存器类似
### @? What is DMA
direct memory access

### @? 缓存一致性
什么时候有使用到？如果有的话，还有多线程的问题吗？

### @？intel lock 汇编指令
hotspot的volatile和synchronize都用的lock来实现  
为什么可以实现
volatile怎么实现可见性和指令不可重排的

###
因为有缓存一致性机制，所以其实在cpu层面，是不存在共享数据竞争的。
多线程下的共享数据竞争，是因为指令重排，在编程习惯上，我们依赖于高层语言的代码执行顺序。却不知道他们在CPU层面是会被重排序的。
Java里面的volatile，其实就是禁止指令重排。用了内存屏障，前面的指令都会被执行完毕，再执行接下里的指令。防止乱序。
多线程下的共享资源竞争，更多的是因为没有原子性。比如说对同一个数的a++。他们都读到a，都加1，都往回set，结果就是1而不是2。这个是因为a的读取和plus和写入不是原子性的。所以要加同步锁。  
而volatile跟同步的关系是，


### 缓存一致性保证了内存可见性吗
不。有两方面的原因：
1. 在寄存器里待太久
2. 在store buffer(cpu和L1 cache之间的另一层缓存)待太久。
以上两者，都是还没有到缓存，所以被缓存一致性照顾不到的。
**内存屏障**可以解决这个问题
内存屏障是用来保证屏障之前的指令要在屏障之后的指令前执行结束。  
所以他意味着，load都要load完，store也要store完。注意这里目标说的都是到主存。而不是寄存器或者store buffer或者cache。  
java里面的volatile就是通过内存屏障来实现的。  


### 线程与纤程
线程要用户态和内核态切换，所以是重量级的。  
纤程只在用户态，是轻量级的。

## @? 中断，软中断，硬中断

### class loading
loading, linking, initializing
it's all about class, not about object. 
in linking, static field will be set to default value, in initializing stat, the static field will be initialized. 
if you want to hack the parent-delegation loading, you need to override the loadClass method.

### @? 热加载，热部署

 

















