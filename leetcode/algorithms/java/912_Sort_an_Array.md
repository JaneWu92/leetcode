```java
class Solution {
    Random random = new Random();
    public int[] sortArray(int[] nums) {
        quickSort(nums, 0, nums.length-1);
        return nums;
    }
    private void quickSort(int[] nums, int left, int right){
        if(left == right) return;
        int p = randomPartition(nums, left, right);
        if(left < p-1) quickSort(nums, left, p-1);
        if(p+1 < right)quickSort(nums, p+1, right);
    }
    private int randomPartition(int[] nums, int left, int right){
        int inc = random.nextInt(right - left + 1);
        swap(nums, left, left+inc);
        return partition(nums, left, right);
    }
    private int partition(int[] nums, int left, int right){
        int more = right + 1;
        for(int i = left + 1; i < more; i++){
            if(nums[i] > nums[left]){
                swap(nums, i--, --more);
            }
        }
        swap(nums, left, more - 1);
        return more - 1;
    }
    private void swap(int[] nums, int i, int j){
        if(nums[i] == nums[j]) return;
        nums[i] = nums[i] ^ nums[j];
        nums[j] = nums[i] ^ nums[j];
        nums[i] = nums[i] ^ nums[j];
    }

}
```