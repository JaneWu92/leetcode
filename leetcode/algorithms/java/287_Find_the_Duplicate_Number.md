```java

//题目出错了，这个条件应该是不该存在的“数组中只有一个重复的数字，但它可能不止重复出现一次”。对于二分法的解法来说
//这题的重点在于，怎么缩小这个怀疑的范围。
//最直接的想就是，我每个数去遍历一下他们的数量，大于的1的得到结果。
//但这里的问题是，这样子查找是无差别的。
//而在这题中，其实有一个条件，要用到，1~n的数，一共有n+1个，那么肯定有一个是超过一个的。
//那就有一个特征可以把以上和以下的区分开来，就是<自己的数。

class Solution {
    public int findDuplicate(int[] nums) {
        int len = nums.length;
        int left = 1;
        int right = len - 1;
        while(left < right){
            int mid = (left+right) >> 1;
            int count = 0;
            for(int i = 0; i < len; i++){
                if(nums[i] <= mid) count++;
            }
            if(count > mid){
                right = mid;
            }else{
                left = mid+1;
            }
        }
        return left;


    }
}
```