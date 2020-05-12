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