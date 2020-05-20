Solution 1:  
Pass  
```java
import java.util.*;

// start at: 19:15
// end at: 21:53
// what a shame...
class Solution {
    public int pathSum(int[] nums) {
        int[][] fineNums = new int[nums.length][2];
        Map<Tuple, Integer> mapAll = new HashMap<Tuple, Integer>();
        Map<Tuple, Integer> mapLeaf = new HashMap<Tuple, Integer>();
        for(int num: nums){
            int a = num/100;
            int b = num/10%10;
            int c = num%10;
            mapAll.put(new Tuple(a,b), c);
        }
        // a,b,c => a+1, 2b-1, && a+1, 2b
        // find all leaf nodes and keep them
        for(Map.Entry e: mapAll.entrySet()){
            Tuple k = (Tuple)e.getKey();
            int v = (int)e.getValue();
            Tuple left = new Tuple(k.first + 1, 2 * k.second -1);
            Tuple right = new Tuple(k.first + 1, 2 * k.second);
            if(!mapAll.containsKey(left) &&  !mapAll.containsKey(right)){
                mapLeaf.put(k, v);
            }
        }
        int sum = 0;
        for(Map.Entry e: mapLeaf.entrySet()){
            Tuple k = (Tuple)e.getKey();
            Integer v = (Integer)e.getValue();

            while(true){
                sum += v;
                k = new Tuple(k.first - 1, (k.second + 1)/2);;
                v = mapAll.get(k);
//                System.out.println(v);
                if(mapAll.get(k) == null){
                    break;
                }
            }
        }
        return sum;
    }
}

class Tuple{
    public int first;
    public int second;
    public Tuple(int f, int s){
        first = f;
        second = s;
    }
    @Override
    public boolean equals(Object o){
        Tuple other = (Tuple)o;
        return other.first == first && other.second == second;
    }
    @Override
    public int hashCode(){
        return first + second;
    }
}
```

Solution 2: 
Pass  
```java
import java.util.*;

// start at: 21:54
// end at:
// 1. Doesn't need a Tuple structure, we can just maintain first 2 digit and compute to seperate them
// 2. use a depth first search to sum all results would be good.
// 3. use a total_sum and a seperate sum to represent different scope of sum
class Solution {
    private Map map;
    int total_sum = 0;

    public int pathSum(int[] nums) {
        map = new HashMap<Integer, Integer>();
        for (int num : nums) {
            map.put(num / 10, num % 10);
        }
        dfs(nums[0] / 10, 0);
        return total_sum;
    }

    private void dfs(int key, int sum) {
        int first = key / 10;
        int second = key % 10;
        int val = (int) map.get(key);
        sum += val;
        int f_left = first + 1;
        int s_left = second * 2 - 1;
        int left = f_left * 10 + s_left;
        int f_right = first + 1;
        int s_right = second * 2;
        int right = f_right * 10 + s_right;
        if (!map.containsKey(left) && !map.containsKey(right)) {
            total_sum += sum;
            return;
        }
        if (map.containsKey(left)) {
            dfs(left, sum);
        }
        if (map.containsKey(right)) {
            dfs(right, sum);
        }

    }
}
```