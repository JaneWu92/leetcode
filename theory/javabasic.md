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

### final, finally, finalize
final can be used in fields or method or class.  
in fields: this field can't be modified once initialized.
in method: this method can't be override by child class.
in class: this class can't be extends.  
finally: the finally block always executes when the try block exists.  
So it ensures that the resource close can be handled even there's exception in try block.
finalize: when an object is garbage collected, the finalize method will be called. It's for performing some resource close. 
It's recommend not to override finalize method because it may have many overhead to garbage collection.

### java reference: soft, weak, phantom reference
**why we need different type of reference**
Java is known of it's dynamic garbage collection.  
And the key point of garbage collection is to release the objects that we think it's okay to release to make some room.  
The straightforward way is, when there's no reference to an object it will be marked as okay-to-release.  
And then it will be 

**what are they exactly**

### override vs overload
method signature: method name + parameter type/length 
return type is not included here, means that if 2 methods only differs at return type, compiler will output error
override: subclass has same signature as super class  
overload: in one class, there are 2 methods has same name but different parameter type/length

### static and dynamic binding
binding: link a method to a class/instance  
static binding: compile bind. at compile time it's decided which class/instance this method is linked  
dynamic binding: only at runtime it will be figured out which class/instance this method is bound to  
why there are dynamic binding: it's in the java polymorphism.  
static binding: overloaded, private, static, final methods. type of reference  
dynamic binding: overriden methods. actual object

### java virtual function
In java, except for static function, final function, private function, all others are virtual function.  
virtual function this concept in fact are in the polymorphism. so if a function can be override, in this extend it's a virtual function.  
**java virtual method table**
in jvm, each class will have a virtial method table. which is an array has an index and the method itself.  
here's 2 concept, reference type and object type.  
Father o = new Children();  // reference type is father and object type is children 
o.talk(); // talk method is overriden by children, and is public non-static non final method  
because JVM will see that talk is virtual method, it then note down this method index in Father class's virtual table  
then go to the stack to see the o's real object type.  
when it find that it's children, then it will go to the same index of the children's virtual method table.  
This is how dynamic binding work.  

### Java GC
**how to decide when an object is dead**
1. reference counting
2. root set of reference
reference counting:  can not prevent isolated island  
if a and b reference each other, they should be all dead but they are all alive.  
root set of reference: from top to bottom, and if you are not find by the very root reference, you are dead.  
root set:
1. active thread
2. local variables
3. static variables

### happens-before
This is a relation to denote partial order.  
If an action A has the relation of "happens-before" B,  
then I don't care how you physically execute the action, but you need to make sure that the result guarantee the "happens beore" relation.  


### assembler & assembly language
assembler is bound to specific computer architechture.  
and each assembler has its own assembly language.  
compiler: high level source code to machine code  
assember: assembly code(symbolic machine code) to machine code  

### machine instruction vs machine code
machine code: 10101001  
machine instruction: same with machine code
assembly code: symbolic naming added for machine code (mnemonic like MOV, LOAD) 

### what does JVM do exactly
JVM is java virtual machine for short, so to answer this question we should answer first:  
what does machine do exactly when execute machine code.  
