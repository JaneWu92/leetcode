Solution 1:  
```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }
    Semaphore fooSm = new Semaphore(1);
    Semaphore barSm = new Semaphore(0);

    public void foo(Runnable printFoo) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
            fooSm.acquire();
        	// printFoo.run() outputs "foo". Do not change or remove this line.
        	printFoo.run();
            barSm.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
            barSm.acquire();
            // printBar.run() outputs "bar". Do not change or remove this line.
        	printBar.run();
            fooSm.release();
        }
    }
}
```