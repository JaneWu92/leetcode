Book:  
Computer Systems. A Programmer’s Perspective [3rd ed.] 
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

















