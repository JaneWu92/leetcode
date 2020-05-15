Solution 1:  
Not Pass  
Time exceed limitation  
test case: 8 30 6 4
Math.pow(8, 30) is 1.**E27 which is quite huge if you want to loop  
It seems that it's because it's depth first.  
We may need a width first algorithm to decrease the time complexity.  
However actually I am not quite sure the difference between depth first and width first here.  
I mean, is the problem here is really because the depth first algorithm?
```java
import java.util.*;

class Solution {
    int numStay = 0;
    public double knightProbability(int N, int K, int r, int c) {
        if(K == 0){
            return 1;
        }
        double numAllPosibilities = Math.pow(8, K);
        ifNextStepStay(N,K, r,c);
        return ((double)numStay)/numAllPosibilities;
    }
    private void ifNextStepStay(int N,int K, int r, int c){
        if(K > 0){
            List<int[]> list = getAllDirection(r,c);
            for(int[] ints: list){
                int rr = ints[0];
                int cc = ints[1];
                if(inBox(N, rr, cc)){
                    if(K == 1){
                        numStay++;
                    }
                    ifNextStepStay(N, K-1, rr, cc);
                }
            }
        }
    }
    private boolean inBox(int N, int r, int c){
        return (r>=0 && r<=N-1) && (c>=0 && c<=N-1);
    }
    private List<int[]> getAllDirection(int r, int c){
        List<int[]> list = new ArrayList<int[]>();
        list.add(new int[]{r+2,c-1});
        list.add(new int[]{r+1,c-2});
        list.add(new int[]{r+2,c+1});
        list.add(new int[]{r+1,c+2});
        list.add(new int[]{r-2,c-1});
        list.add(new int[]{r-1,c-2});
        list.add(new int[]{r-2,c+1});
        list.add(new int[]{r-1,c+2});
        return list;
    }
}
```

Solution 2:  
Pass  
The important thing here is that reuse the previous result.
```java
import java.util.*;

class Solution {
    // create dp[N][N]
    // firstly init dp[r][c] as 1.
    // for each loop in K: create one more tempdp[N][N]
    // loop N, N
    // index: i, j
    // there's 2 point here: loopCurrent and other
    // "ohter" is in another loop of 8 directions.
    // tempdp[other] += dp[i][j]   if other is in box and dp[i][j] > 0
    // end loop N,N, dp = tempdp
    // end loop K, sum all dp and devide total

    public double knightProbability(int N, int K, int r, int c) {
        double[][] dp = new double[N][N];
        dp[r][c] = 1.0;
        int[][] eightP = {{-2,-1},{-2,1},{2,-1},{2,+1},{-1,-2},{-1,2},{1,-2},{1,2},};
        for(int m = 0; m < K; m++){
            double[][] tdp = new double[N][N];
            for(int i = 0; i < N; i++){
                for(int j = 0; j < N; j++){
                    for(int n = 0; n < 8; n++){
                        int[] oneOfEightInc = eightP[n];
                        int[] other = {i + oneOfEightInc[0], j + oneOfEightInc[1]};
                        if(dp[i][j] > 0 && inBox(other, N)){
                            tdp[other[0]][other[1]] += dp[i][j];
                        }
                    }
                }
            }
            dp = tdp;
        }
        double s = sum(dp);
        double total = Math.pow(8, K);
        return s/total;
    }

    private boolean inBox(int[] point, int N){
        return (point[0] >= 0 && point[0] < N) && (point[1] >= 0 && point[1] < N);
    }
    private double sum(double[][] arr){
        double sum = 0;
        for(double[] r: arr){
            for(double c: r){
                sum+=c;
            }
        }
        return sum;
    }
}

```