Solution 1:  
Pass  
There's a way to improve this and make it simpler:  
Just 2 items: step and directions.  
step is 1,1,2,2,3,3,4,4,5,5
and directions are {0, 1}, {1, 0}, {0, -1}, {-1, 0}  
step + dir = gogo

```java
import java.util.stream.*;

class Solution {
    public int[][] spiralMatrixIII(int R, int C, int r0, int c0) {
        int[][] result = new int[R * C][2];
        int count = 1; // to maintain the count of all walded cells: count <= R * C
        int[][] dirs = { {0, 1}, {1, 0}, {0, -1}, {-1, 0}};
        int dirIndex = 0;
        int spiralNUm = 1;
        result[count - 1] = new int[]{r0, c0};
        count++;
        while (true) {
            if (count > R * C) {
                break;
            }
            int oriR0 = r0;
            int oriC0 = c0;
            r0 = r0 + dirs[dirIndex][0] * spiralNUm;
            c0 = c0 + dirs[dirIndex][1] * spiralNUm;
            for (int j = oriC0 + dirs[dirIndex][1]; (dirs[dirIndex][1] == 1 && j <= c0) || (dirs[dirIndex][1] == -1 && j >= c0); j = j + dirs[dirIndex][1]) {
                if (j >= 0 && r0 >= 0 && j < C && r0 < R) {
                    result[count - 1] = new int[]{r0, j};
                    count++;
                }
            }
            oriR0 = r0;
            oriC0 = c0;
            dirIndex = (dirIndex + 1) % 4;
            r0 = r0 + dirs[dirIndex][0] * spiralNUm;
            c0 = c0 + dirs[dirIndex][1] * spiralNUm;
            for (int i = oriR0 + dirs[dirIndex][0]; (dirs[dirIndex][0] == 1 && i <= r0) || (dirs[dirIndex][0] == -1 && i >= r0); i = i + dirs[dirIndex][0]) {
                if (i >= 0 && c0 >= 0 && i < R && c0 < C) {
                    result[count - 1] = new int[]{i, c0};
                    count++;
                }
            }
            dirIndex = (dirIndex + 1) % 4;
            spiralNUm++;
        }
        return result;
    }
}
```