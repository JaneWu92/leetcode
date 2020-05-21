Solution 1:  
Pass  
Interesting. I think this will definitely cause deadlock but the test case is pass!
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