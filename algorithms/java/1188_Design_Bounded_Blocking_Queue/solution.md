Solution 1: Use synchronized and wait and notify<br/>
Result: invalid solution with time limitation<br/>
<br/>
Question:<br/>
What is synchronized and wait and notify indeed?<br/>


```
import java.util.LinkedList;
import java.util.List;

class BoundedBlockingQueue {
	
	private int capacity;
	private int size = 0;
	private List list;
	
    public BoundedBlockingQueue(int capacity) {
    	this.capacity = capacity;
        list = new LinkedList<Integer>();
    }
    
    public synchronized void enqueue(int element) throws InterruptedException {
        while(size == capacity) {
        	wait();
        }
        list.add(0, element);
        size ++;
        if(size == 1) {
        	notifyAll();
        }
    }
    
    public synchronized int dequeue() throws InterruptedException {
        while(size == 0) {
        	wait();
        }
        int result;
        result = (int) list.get(size - 1);
        list.remove(size - 1);
        size--;
        if(size == capacity - 1 ) {
        	notifyAll();
        }
        
        return result;
    }
    
    public int size() {
        return size;
    }
}
```