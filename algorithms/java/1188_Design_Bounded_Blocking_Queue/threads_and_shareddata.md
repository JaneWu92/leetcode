|memory type|stored data type|shared or not|
|----|----|----|
|stack|primitive types and object references|not shared by threads|
|heap|objects|shared by threads|
|method area|class variables(compared to object variables)|shared by threads|

### lock and monitor


reference:
https://www.javaworld.com/article/2076971/how-the-java-virtual-machine-performs-thread-synchronization.html