```java
class Solution {
    public String largestNumber(int[] nums) {
        String[] nums_s = new String[nums.length];
        for(int i = 0; i < nums.length; i++){
            nums_s[i] = String.valueOf(nums[i]);
        }
        Arrays.sort(nums_s, (s1, s2) -> {
            int minLength = Math.min(s1.length(), s2.length());
            for(int i = 0; i < minLength; i++){
                if(s1.charAt(i) != s2.charAt(i)){
                    return s2.charAt(i) - s1.charAt(i);
                }
            }
            return (s2+s1).compareTo(s1+s2);
        });
        StringBuilder sb = new StringBuilder();
        for(String s: nums_s) sb.append(s);
        while(sb.length()> 1 && sb.charAt(0) == '0'){
            sb.deleteCharAt(0);
        }
        return sb.toString();
    }
}
```

```aidl
算法正确性的证明：
反证法：
假设存在一个最大序列不满足该排序规则，那么至少存在两个数字的顺序不按上述传递性的定义排列，
如......s2s1.......，其中s2s1 < s1s2。
接下来，其他字符串的位置不变，按照传递性定义，将s2和s1交换位置后，
可证明......s1s2...... > ......s2s1......，这与假设不符，即......s2s1.......并不是最大序列，
所以最优序列必定满足上述传递性的定义。

重点在于提出，存在一个相邻的排序例子。
```