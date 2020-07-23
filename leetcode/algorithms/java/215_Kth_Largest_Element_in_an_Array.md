Solution1:
```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        return findMthOrdered(nums, 0, nums.length-1, nums.length-k+1);
    }

    //find mth num when nums is asc ordered, 
    private int findMthOrdered(int [] nums, int left, int right, int m){
        if(left == right){
            return nums[left];
        }else if(left + 1 == right){
            if(m == 1) return Math.min(nums[left], nums[right]);
            return Math.max(nums[left], nums[right]);
        }

        int comp = nums[left];
        int less = partition(nums, left, right, comp);
        if(less - left + 1 == m){
            return nums[less];
        }else if(less - left + 1 > m){
            return findMthOrdered(nums, left, less -1, m);
        }else if(less - left + 1 < m){
            return findMthOrdered(nums, less+1, right, m-(less+1-left));
        } 
        return -1;
    }

    private int partition(int[] nums, int left, int right, int comp){
        int less = left;
        int more = right + 1;
        int help = left+1;
        while(help > less && help < more){
            if(nums[help] <= comp){
                swap(nums, help++, ++less);  
            }else{
                swap(nums, help, --more);
            }
        }
        swap(nums, left, less);
        return less;
    }

    private void swap(int[] nums, int i, int j){
        if(nums[i] == nums[j]) return;
        nums[i] = nums[i] ^ nums[j];
        nums[j] = nums[i] ^ nums[j];
        nums[i] = nums[i] ^ nums[j];
    }
}
```

Solution2: 
Solution1的几个问题，
1. random partition,   
2. quickselect里面的m，可以不用那么复杂，就用index来表示。不用想成第几大，而直接看成我要的这个数字是整个数组的第几位。  
3. partition的这个函数也可以再精炼一些。比compare大的，才换，不然指针就一直往前走。

```java
class Solution {
    Random random = new Random();
    public int findKthLargest(int[] nums, int k) {
        return quickselect(nums, 0, nums.length-1, nums.length-k);
    }

    private int quickselect(int[] nums, int left, int right, int idx){
        if(left == right) return nums[left];
        int p = randomPartition(nums, left, right);
        if(p == idx) return nums[p];
        return p < idx ? quickselect(nums, p+1, right, idx) : quickselect(nums, left, p-1, idx);
    }

    private int randomPartition(int[] nums, int left, int right){
        int inc = random.nextInt(right - left + 1);
        swap(nums, left+inc, left);
        return partition(nums, left, right);
    }

    private int partition(int[] nums, int left, int right){
        int more = right + 1;
        for(int help = left + 1; help < more; help++){
            if(nums[help] > nums[left]){
                swap(nums, help--, --more);
            }
        }
        swap(nums, left, more-1);
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



