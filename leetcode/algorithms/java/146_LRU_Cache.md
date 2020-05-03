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