```java
import java.util.Arrays;

/**
 * If use the force to make it, the biggest problem is that there will be duplicated count on the cells.
 * The key point here is that how to avoid this duplication.
 * Avoid duplication is by reusing previous result.
 * If you want to calculate how many enemies it can pomb, it requires:
 * how many continous neighboring "E" is in the left, right, up, down.
 * Note: it's dynamic programing algorithm, you may want to have a series of practice to make it stable.
 */

class Solution {
    public int maxKilledEnemies(char[][] grid) {
        int rowL = grid.length;
        if (rowL == 0) {
            return 0;
        }
        int colL = grid[0].length;
        if (colL == 0) {
            return 0;
        }
        int[][] leftGrid = new int[rowL][colL];
        int[][] rightGrid = new int[rowL][colL];
        int[][] upGrid = new int[rowL][colL];
        int[][] downGrid = new int[rowL][colL];
        int[][] resultGrid = new int[rowL][colL];
        // left and right: for each row
        for (int i = 0; i < rowL; i++) {
            if (grid[i][0] == 'E') {
                leftGrid[i][0] = 1;
            }
            if (grid[i][colL - 1] == 'E') {
                rightGrid[i][colL - 1] = 1;
            }
        }
        for (int i = 0; i < rowL; i++) {
            for (int j = 1; j < colL; j++) {
                if (grid[i][j] == 'E') {
                    leftGrid[i][j] = leftGrid[i][j - 1] + 1;
                } else if (grid[i][j] == '0') {
                    leftGrid[i][j] = leftGrid[i][j - 1];
                    resultGrid[i][j] = resultGrid[i][j] + leftGrid[i][j - 1];
                }
            }
            for (int j = colL - 1 - 1; j > -1; j--) {
                if (grid[i][j] == 'E') {
                    rightGrid[i][j] = rightGrid[i][j + 1] + 1;
                } else if (grid[i][j] == '0') {
                    rightGrid[i][j] = rightGrid[i][j + 1];
                    resultGrid[i][j] = resultGrid[i][j] + rightGrid[i][j + 1];
                }
            }
        }
        // up and down: for each column
        for (int j = 0; j < colL; j++) {
            if (grid[0][j] == 'E') {
                upGrid[0][j] = 1;
            }
            if (grid[rowL - 1][j] == 'E') {
                downGrid[rowL - 1][j] = 1;
            }
        }
        for (int j = 0; j < colL; j++) {
            for (int i = 1; i < rowL; i++) {
                if (grid[i][j] == 'E') {
                    upGrid[i][j] = upGrid[i - 1][j] + 1;
                } else if (grid[i][j] == '0') {
                    upGrid[i][j] = upGrid[i - 1][j];
                    resultGrid[i][j] = resultGrid[i][j] + upGrid[i - 1][j];
                }
            }
            for (int i = rowL - 1 - 1; i > -1; i--) {
                if (grid[i][j] == 'E') {
                    downGrid[i][j] = downGrid[i + 1][j] + 1;
                } else if (grid[i][j] == '0') {
                    downGrid[i][j] = downGrid[i + 1][j];
                    resultGrid[i][j] = resultGrid[i][j] + downGrid[i + 1][j];
                }
            }
        }
        int max = 0;
        for (int i = 0; i < rowL; i++) {
            int[] row = resultGrid[i];
            Arrays.sort(row);
            if (row[colL - 1] > max) {
                max = row[colL - 1];
            }
        }
        return max;
    }
}

```