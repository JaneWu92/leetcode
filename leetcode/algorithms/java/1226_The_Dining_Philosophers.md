Solution 1:  
Pass  
Interesting. I think this will definitely cause dead lock but the test case is pass!  
Deadlock happens when all philosophers pick up their left fork.  
In order to prevent this situation, we may need a semaphor to control only 4 philosophers can take the forks.
```java
class DiningPhilosophers {

    public DiningPhilosophers() {

    }
    Object[] forks = {"","","","",""};

    // call the run() method of any runnable to execute its code
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
        Object leftFork = forks[(philosopher-1+5)%5];
        Object rightFork = forks[philosopher];
        synchronized (leftFork) {
            pickLeftFork.run();
            synchronized (rightFork){
                pickRightFork.run();
                eat.run();
                putRightFork.run();
            }
            putLeftFork.run();
        }

    }


}
```

Solution 2:  
Pass  
```java
import java.util.concurrent.Semaphore;

class DiningPhilosophers {

    public DiningPhilosophers() {

    }
    Object[] forks = {"", "", "", "", ""};
    Semaphore semaphore = new Semaphore(4);
    // call the run() method of any runnable to execute its code
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
        Object leftFork = forks[(philosopher - 1) % 5];
        Object rightFork = forks[philosopher];
        semaphore.acquire();
        synchronized (leftFork) {
            pickLeftFork.run();
            synchronized (rightFork) {
                pickRightFork.run();
                eat.run();
                putRightFork.run();
            }
            putLeftFork.run();
        }
        semaphore.release();
    }


}
```