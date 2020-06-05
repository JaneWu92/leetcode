### equals & ==
**Integer**
```java
Integer a = new Integer(3); //new instance in heap
Integer b = 3; // Integer.valueOf(3); user IntegerCache
Integer c = 128; 
//Integer.valueOf(128); 
//while 128 is out of default IntegerCache limit. so it's new instance in heap
//by increasing the limit to 1000, then it can use the cache.
//use cache it means that it will == (Integer c = new Integer(128);)
Integer d = new Instance(5000);
int e = 5000;
d == e; //true, because there's an unboxing of d.

```
**String**
```java
String a = "1"; // String pool
String c = new String("2"); // new instance in heap
String e = new String("3").intern(); // String pool
//note: if use too many times of intern(), then string pool contains too many string. the performance will be bad.
```

**how to override an equals**
1. compare reference to see if it's same object
2. if it's null, return false
3. compare class type, if it's different type, return false
4. foreach field, compare ...

**what if we only override equals?**  
Equals denotes that how we threat these 2 objects as equal.  
And let's say when we come to some hashmap, and use this "object" as key.  
The hashmap use hashcode to decide which bucket this object should sit.  
To some extent, hashmap here treat hashcode as an mechanism to see if these 2 objects are same.  
It means that in hashmap's idea, if 2 object are same, their hashcode should be same.  
And if we only override equals, but leave alone hashcode, then hashcode and equals will behave differently.  
So that 2 equal object may go to different bucket of hashmap.

**equals, hashcode, compareTo need to behave same**
hashcode: hashset, hashmap  
compareTo: Collection.binarySearch(list, targetObject);

### What do you think of Java platform
Java is a very popular and widely-used language.  
It's well-known because of its "write once and run everywhere".  
This feature is because of its JVM.  
So in fact java code is running first in the virtual machine and then the machine will translate the  "java byte code" into native machine code.  
This is why java is called the interpreted language which corresponds to compiled language.  
However, the JIT(Just in time compiler) is in fact more and more used now in Java which can recognize hot spot code and compile it to native machine code.  
And here comes the question, if "write once and run everywhere" is based on the "java byte code", what will happen if there some "JIT" machine code.  
Here's the thing, the JIT is in the progress of "running" bytecode. If not JIT, then it needs to interprete each bytecode.  
And with JIT, JVM knows what bytecode needs to be complied to machine code first. And then it can be directly used.  

**javac vs JIT**
javac: compile java source code to java bytecode  
JIT: java just in time compiler. compile java bytecode to native machine code at runtime before executing(JVM interpretes  the bytecode into native machine code) it.

### Exception vs Error
Exception is something that can be recovered while error not.  
So exception needs to be taken care very carefully.  
Exception like nullpointer exception, index out of bound exception. Error like OOM error.  
Compiler exception vs runtime exception  
I think it may be different because of one is checked exception and one is unchecked.  
And unchecked exception is runtime exception.  
They both implement Throwable.  
1. Linage error: NoClassDefFoundError
2. vitual machine error: OOM Error, StackOverflow Error
3. checked exception: IO exception
4. runtime exception: NullPointerException, IndexOutOfBoundException, ClassCastException









