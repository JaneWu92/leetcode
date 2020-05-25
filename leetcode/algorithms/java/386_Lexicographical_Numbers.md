Solution 1:  
Pass  
Time complexity: nlogn  
Space complexity: n
```java
import java.util.*;

class Solution {
    public List<Integer> lexicalOrder(int n) {
        List<Integer> list = new ArrayList();
        for (int i = 1; i <= n; i++) {
            list.add(i);
        }
        list.sort(((a, b) -> lexicalCompare(a, b)));
        return list;
    }

    private int lexicalCompare(int a, int b) {
        if (a == b) {
            return 0;
        }
        String aS = String.valueOf(a);
        String bS = String.valueOf(b);
        int index = 0;

        while (index < aS.length() && index < bS.length()) {
            if (index == aS.length()) {
                return -1;
            } else if (index == bS.length()) {
                return 1;
            }
            int aT = aS.charAt(index) - '0';
            int bT = bS.charAt(index) - '0';
            if (aT == bT) {
                index++;
                continue;
            }
            return aT > bT ? 1 : -1;
        }
        return 0;
    }
}
```

Solution 2:  
Pass  
```java
import java.util.*;

class Solution {
    public List<Integer> lexicalOrder(int n) {
        List<Integer> list = new ArrayList();
        for (int i = 1; i <= 9; i++) {
            dfs(i, list, n);
        }
        return list;
    }

    private void dfs(int num, List<Integer> list, int n) {
        if (num <= n) {
            list.add(num);
        }
        //num + 0,1,2,3,4,5,6,7,8,9
        for(int i = 0; i <= 9; i++){
            int newN = num * 10 + i;
            if(newN > n){
                return;
            }
            dfs(newN, list, n);
        }

    }
}

```