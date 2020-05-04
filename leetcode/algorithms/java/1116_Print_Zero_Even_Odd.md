Solution 1:  
Not Pass.  
This will have problem. Because shared data i is not thread safe.
```java
import java.util.concurrent.Semaphore;

class ZeroEvenOdd {
    private int n;
    //n = 7
    //01020304050607
    public ZeroEvenOdd(int n) {
        this.n = n;
    }
    Semaphore zeroSm = new Semaphore(1);
    Semaphore evenSm = new Semaphore(0);
    Semaphore oddSm = new Semaphore(0);
    private int i = 1;
    // printNumber.accept(x) outputs "x", where x is an integer.
    public void zero(IntConsumer printNumber) throws InterruptedException {
        while(true){
            if(i > n){
                break;
            }
            zeroSm.acquire();
            printNumber.accept(0);
            if(i % 2 == 1){
                oddSm.release();
            }else{
                evenSm.release();
            }
        }
    }

    public void even(IntConsumer printNumber) throws InterruptedException {
        while(true){
            if(i > n){
                break;
            }
            evenSm.acquire();
            printNumber.accept(i);
            i++;
            zeroSm.release();
        }
    }

    public void odd(IntConsumer printNumber) throws InterruptedException {
        while(true){
            if(i > n){
                break;
            }
            oddSm.acquire();
            printNumber.accept(i);
            i++;
            zeroSm.release();
        }
    }
}
```
Solution 2:  
Pass.  
Time beats 27.43% users.  
Memory beats 5.41% users.
```java
import java.util.concurrent.Semaphore;

class ZeroEvenOdd {
    private int n;
    //n = 7
    //01020304050607
    public ZeroEvenOdd(int n) {
        this.n = n;
    }
    Semaphore zeroSm = new Semaphore(1);
    Semaphore evenSm = new Semaphore(0);
    Semaphore oddSm = new Semaphore(0);
    // printNumber.accept(x) outputs "x", where x is an integer.
    public void zero(IntConsumer printNumber) throws InterruptedException {
        for(int i = 1; i < n +1; i++){
            zeroSm.acquire();
            printNumber.accept(0);
            if(i % 2 == 1){
                oddSm.release();
            }else{
                evenSm.release();
            }
        }
    }

    public void even(IntConsumer printNumber) throws InterruptedException {
        for(int i = 2; i < n +1; i+=2){
            evenSm.acquire();
            printNumber.accept(i);
            zeroSm.release();
        }
    }

    public void odd(IntConsumer printNumber) throws InterruptedException {
        for(int i = 1; i < n +1; i+=2){
            oddSm.acquire();
            printNumber.accept(i);
            zeroSm.release();
        }
    }
}
```