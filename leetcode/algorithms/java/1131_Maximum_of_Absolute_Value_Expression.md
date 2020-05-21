Solution 1:  
No pass  
Time exceed limitation
```java
// start at 20:37
// end at
class Solution {
    public int maxAbsValExpr(int[] arr1, int[] arr2) {
        int max = Integer.MIN_VALUE;
        for(int i = 0; i < arr1.length; i++){
            for(int j = 0; j < arr1.length; j++){
                int num = Math.abs(arr1[i] - arr1[j]) + Math.abs(arr2[i] - arr2[j]) + Math.abs(i - j);
                max = num > max? num:max;
            }
        }
        return max;
    }
}
```