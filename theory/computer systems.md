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







