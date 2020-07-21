```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if(intervals.length < 2){
            return intervals;
        }
        List<int[]> resultList = new ArrayList<int[]>();
        Arrays.sort(intervals, (v1,v2) -> v1[0] - v2[0]);
        int cur[] = intervals[0];
        for(int i = 1; i < intervals.length ; i++){
            int[] second = intervals[i];
            if(second[0] > cur[1]){
                resultList.add(cur);
                cur = second; 
            }else{
                int secondIndex = second[1] <= cur[1] ? cur[1] : second[1];
                cur[1] = secondIndex;
            }
        }
        resultList.add(cur);
        return resultList.toArray(new int[resultList.size()][2]);
    }
}
```