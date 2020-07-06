Solution 1:  
Time limit exceeded.  
In this solution, I use recursive algorithm to calculate
```java
class Solution {
    public int minPathSum(int[][] grid) {
        return minPath(grid, 0, 0);
    }

    private int minPath(int[][] grid, int x, int y){
        int m = grid.length - 1;
        int n = grid[0].length - 1;
        if(x == m && y == n){
            return grid[x][y];
        }
        int min = 0;
        if(x+1 <=m && y+1 <=n){
            min = Math.min(minPath(grid, x+1, y), minPath(grid, x, y+1));
        }else if(x+1 > m){
            min = minPath(grid, x, y+1);
        }else if(y+1 > n){
            min = minPath(grid, x+1, y);
        }
        return grid[x][y] + min;
    }
    
}
```

Solution 2: 
```java
class Solution {

    public int minPathSum(int[][] grid) {
        int[][] temp = new int[grid.length][grid[0].length];
        for(int i = grid.length-1; i>=0; i--){
            for(int j = grid[0].length-1; j>=0; j--){
                if(i == grid.length-1 && j == grid[0].length-1){
                    temp[i][j] = grid[i][j];
                }else if(i == grid.length-1){
                    temp[i][j] = grid[i][j] + temp[i][j+1];
                }else if( j == grid[0].length-1){
                    temp[i][j] = grid[i][j] + temp[i+1][j];
                }else{
                    temp[i][j] = grid[i][j] + Math.min(temp[i][j+1], temp[i+1][j]);
                }
            }
        }
        return temp[0][0];
    }
}
```


Solution 2: 








