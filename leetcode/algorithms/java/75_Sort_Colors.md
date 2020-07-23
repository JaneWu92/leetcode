```java

class Solution {
    public void sortColors(int[] nums) {
        int len = nums.length;
        int zeroIdx = 0;
        int twoIdx = len - 1;
        for(int i = 0; i <= twoIdx; i++){
            if(nums[i] == 0){
                swap(nums, i, zeroIdx++);
            }else if(nums[i] == 2){
                swap(nums, i--, twoIdx--);
            }
        }
    }

    private void swap(int[] nums, int idx1, int idx2){
        if(nums[idx1] != nums[idx2]){
            nums[idx1] = nums[idx1] ^ nums[idx2];
            nums[idx2] = nums[idx1] ^ nums[idx2];
            nums[idx1] = nums[idx1] ^ nums[idx2];
        }
    }
}
```