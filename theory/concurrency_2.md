### synchronized & Reentranlock  
Basically they are same. Different is that Reentranlock has more ability. It can provide:  
1. trylock  
2. reenter mechanism  
3. fair lock  
4. lock interruptibly mechanism.(synchronized needs to wait all the time)  

### volatile
1. visibility
2. happens-before(prevent instruction reordering)
3. can't prevent race condition since it's not atomic instruction  

**Since volatile can't prevnet race condition, when is it enough?**  
When only one thread is write and others are all read. It's enough then.  
Once there's multiple threads to write for a same data, you should use synchronized.

