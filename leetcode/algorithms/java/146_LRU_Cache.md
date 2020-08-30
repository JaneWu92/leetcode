Solution 1: 
<br/>Not Pass
<br/>Didn't read the requirement carefully and try to achieve a Least Totally Used version
```java
import java.util.*;

class LRUCache {
    //<key, value>
    Map<Integer, Integer> valueMap = new HashMap<Integer, Integer>();
    //<count, keyList>
    Map<Integer, Set<Integer>> countAsKeyMap = new HashMap<Integer, Set<Integer>>();
    //<key, count>
    Map<Integer, Integer> countMap = new HashMap<Integer, Integer>();

    int capacity = 0;
    int size = 0;

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key) {
        if (!valueMap.containsKey(key)) {
            return -1;
        }
        int v = valueMap.get(key);
        int oldC = -1;
        int newC = -1;
        if (countMap.containsKey(key)) {
            oldC = countMap.get(key);

        } else {
            oldC = 0;
        }
        newC = oldC + 1;
        countMap.put(key, newC);
        if (countAsKeyMap.containsKey(oldC)) {
            countAsKeyMap.get(oldC).remove(key);
        }
        if (countAsKeyMap.containsKey(newC)) {
            countAsKeyMap.get(newC).add(key);
        } else {
            Set<Integer> set = new HashSet<Integer>();
            set.add(key);
            countAsKeyMap.put(newC, set);
        }
        return v;
    }

    public void put(int key, int value) {
        if (valueMap.containsKey(key)) {
            return;
        } else {
            if (size == capacity) {
                //need to evict old one
                Collection<Integer> vs = countMap.values();
                Object[] ori = vs.toArray();
                Arrays.sort(ori);
                int leastCount = (int) ori[0];
                Set<Integer> keysWithLeastCount = countAsKeyMap.get(leastCount);
                int leastKey = keysWithLeastCount.iterator().next();
                valueMap.remove(leastKey);
                countAsKeyMap.get(leastCount).remove(leastKey);
                countMap.remove(leastKey);
                valueMap.put(key, value);
            } else {
                //just put
                valueMap.put(key, value);
                size++;

                countMap.put(key, 0);
                if (countAsKeyMap.containsKey(0)) {
                    countAsKeyMap.get(0).add(key);
                } else {
                    Set<Integer> set = new HashSet<Integer>();
                    set.add(key);
                    countAsKeyMap.put(0, set);
                }

            }
        }


    }
}

```

Solution 2:
<br/>Not Pass
<br/>Time Limit Exceeded

```java
import java.util.*;

class LRUCache {
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    //<key, weight>
    Map<Integer, Integer> weightMap = new HashMap<Integer, Integer>();

    int capacity = 0;
    int size = 0;
    int maxWeight = 0;

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key){
        if(!map.containsKey(key)){
            return -1;
        }
        weightMap.put(key, ++maxWeight);
        return map.get(key);
    }

    public void put(int key, int value){
        if(map.containsKey(key)){
            map.put(key, value);
            weightMap.put(key, ++maxWeight);
            return;
        }
        if(size == capacity){
            //evict least used keys
            Object[] ori = weightMap.values().toArray();
            Arrays.sort(ori);
            int leastWeight = (int)ori[0];
            int leastUsedKey = -1;
            for(int k: weightMap.keySet()){
                if(weightMap.get(k) == leastWeight){
                    leastUsedKey = k;
                }
            }
            map.remove(leastUsedKey);
            weightMap.remove(leastUsedKey);
            //put new keys
            map.put(key, value);
            weightMap.put(key, ++maxWeight);
        }else{
            map.put(key, value);
            size++;
            weightMap.put(key, ++maxWeight);
        }
    }
}
```
Solution 3:
<br/>Pass
<br/>time complexity: O(n)
```java
import java.util.*;

class LRUCache {
    /**
     * use queue.push, queue.pop to maintain the order
     * get/put both need queue.push
     * one problem is that there will be same key updated.
     * so need an extra countMap to maintain the count.
     * 1)pop 2)countMap value -1 by key.
     * only if value = 0, can treat it as least used.
     * if not, keep popping next elememt.
     */
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    Queue<Integer> queue = new LinkedList<Integer>();
    Map<Integer, Integer> countMap = new HashMap<Integer, Integer>();

    int capacity = 0;
    int size = 0;
    int maxWeight = 0;

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key) {
        if (!map.containsKey(key)) {
            return -1;
        }
        incValueByKey(key, countMap);
        queue.add(key);
        return map.get(key);
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            map.put(key, value);
            queue.add(key);
            incValueByKey(key, countMap);
            return;
        }
        if (size == capacity) {
            //evict least used keys
            int leastUsedKey = -1;
            while(true){
                int tmpKey = queue.poll();
                boolean flag = decValueByKeyAndSeeIfZero(tmpKey, countMap);
                if(flag){
                    leastUsedKey = tmpKey;
                    break;
                }
            }
            map.remove(leastUsedKey);
            countMap.remove(leastUsedKey);
            //put new keys
            map.put(key, value);
            queue.add(key);
            incValueByKey(key, countMap);
        } else {
            map.put(key, value);
            size++;
            queue.add(key);
            incValueByKey(key, countMap);
        }
    }

    private void incValueByKey(int key, Map<Integer, Integer> map) {
        if (!map.containsKey(key)) {
            map.put(key, 1);
        } else {
            int oriV = map.get(key);
            map.put(key, oriV + 1);
        }
    }

    private boolean decValueByKeyAndSeeIfZero(int key, Map<Integer, Integer> map) {
        int oriV = map.get(key);
        map.put(key, oriV - 1);
        return (oriV - 1 == 0) ? true : false;
    }
}
```

Solution4:
beat: 51%, 80%.
```java
class Node{
    int key;
    int val;
    Node pre;
    Node next;
    public Node(int k, int v){
        this.key = k;
        this.val = v;
    }
}


class LRUCache {
    int capacity;
    HashMap<Integer, Node> map = null;
    Node head = null;
    Node tail = null;
    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>(capacity);
        head = new Node(-1, -1);
        tail = new Node(-1, -1);
        head.next = tail;
        tail.pre = head;
    }
    
    public int get(int key) {
        if(!map.containsKey(key)){
            return -1;
        }else{
            Node n = map.get(key);
            n.pre.next = n.next;
            n.next.pre = n.pre;
            n.next = head.next;
            n.pre = head;
            head.next = n;
            n.next.pre = n;
            return n.val;
        }
    }
    
    public void put(int key, int value) {
        if(!map.containsKey(key)){
            // add element
            if(map.size() < capacity){
                // add directly
                addToMapAndUpdateList(key, new Node(key, value));
            }else{
                // remove oldest; add directly
                Node old = tail.pre;
                old.pre.next = tail;
                tail.pre = old.pre;
                map.remove(old.key);
                addToMapAndUpdateList(key, new Node(key, value));
            }
        }else{
            // repleace element and refresh order
            get(key);
            Node n = map.get(key);
            n.val = value;
        }        
    }

    private void addToMapAndUpdateList(int key, Node n){
        map.put(key, n);
        n.next = head.next;
        head.next.pre = n;
        head.next = n;
        n.pre = head;
    }
    public static void main(String[] args){
        LRUCache obj = new LRUCache(2);
        obj.put(1,1);
        obj.put(2,2);
        obj.get(1);
        obj.put(3,3);
        obj.get(2);


    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```