Solution1:
我这个可是o(n)的平均时间复杂度。怎么时间这么慢!  
执行用时：
                    89 ms
                    , 在所有 Java 提交中击败了
                    8.79%
                    的用户

```java
class Solution {
    Random ran = new Random();
    public void wiggleSort(int[] nums) {
        int[] help = new int[nums.length];
        int idx = (nums.length-1) >> 1;
        int median = quickSelect(nums, 0, nums.length-1, idx);

        int s = 0, e = nums.length - 1;
        for(int i = 0; i < nums.length; i ++){
            if(nums[i] < median) help[s++] = nums[i];
            if(nums[i] > median) help[e--] = nums[i];
        }
        for(int i = s; i <= e; i++) help[i] = median;

        int c = 0;
        for(int i = idx; i >= 0; i--){
            nums[c] = help[i];
            c+=2;
        }
        c = 1;
        for(int i = nums.length - 1; i >= idx + 1 ; i--){
            nums[c] = help[i];
            c+=2;
        }
    }

    private int quickSelect(int[] nums, int left, int right, int idx){
        int p = randomPartition(nums, left, right);
        if(p == idx) return nums[p];
        if(p < idx) return quickSelect(nums, p+1, right, idx);
        if(p > idx) return quickSelect(nums, left, p-1, idx);
        return -1;
    }
    
    private int randomPartition(int[] nums, int left, int right){
        int inc = ran.nextInt(right - left + 1);
        swap(nums, left, left+inc);
        return partition(nums, left, right);
    }

    private int partition(int[] nums, int left, int right){
        int more = right + 1;
        for(int i = left; i < more; i++){
            if(nums[i] > nums[left]){
                swap(nums, i--, --more);
            }
        }
        swap(nums, more - 1, left);
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