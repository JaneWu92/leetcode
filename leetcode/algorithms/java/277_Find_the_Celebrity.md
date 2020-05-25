Solution 1:  
Pass  
```java
/* The knows API is defined in the parent class Relation.
      boolean knows(int a, int b); */

public class Solution extends Relation {
    public int findCelebrity(int n) {
        int[][] knowsWho = new int[n][n];
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                boolean ifKnow = knows(i, j);
                int num = ifKnow == true ? 1 : 0;
                knowsWho[i][j] = num;
            }
        }
        int[] knowWhoNum = new int[n];
        int[] whoKnowNum = new int[n];
        for(int i = 0; i < n; i++){
            for(int j = 0; j < n; j++){
                knowWhoNum[i] += knowsWho[i][j];
            }
        }
        for(int j = 0; j < n; j++){
            for(int i = 0; i < n; i++){
                whoKnowNum[j] +=knowsWho[i][j];
            }
        }
        for(int i = 0; i < n; i++){
            if(knowWhoNum[i] == 1 && whoKnowNum[i] == n){
                return i;
            }
        }
        return -1;
    }
}
```
Solution 2:  
Pass
```java
/* The knows API is defined in the parent class Relation.
      boolean knows(int a, int b); */

public class Solution extends Relation {
    public int findCelebrity(int n) {
        boolean candidate[] = new boolean[n];
        for (int i = 0; i < n; i++) {
            candidate[i] = true;
        }
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if(i == j){
                    continue;
                }
                if (knows(i, j) == true || knows(j, i) == false) {
                    candidate[i] = false;
                }
            }
        }
        for (int i = 0; i < n; i++) {
            if (candidate[i] == true) {
                return i;
            }
        }
        return -1;
    }
}
```