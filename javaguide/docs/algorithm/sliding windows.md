## Sliding windows
### 424. 替换后的最长重复字符
* 这里面比较不好理解的是，为什么historyMax不用每次精准地去维护
* 没人说得清楚还
* 但是这个historyMax是historywindow的max，不是currentwindow的max，所以它有可能是虚高的，那通过它来判断，有什么用啊！！
```aidl
// 硬分析一场，
// 不由分说，先看这个right的更新
// 然后判断当下这个window要不要左边缩回来。要缩就一直缩，缩到满足条件了，再继续right++
// 这里的max_length, 要有一个变量来维护
// 这里的maxc_count，不用做那么精准。---》 这个还没有理解透
class Solution {
    public int characterReplacement(String s, int k) {
        int left=0, max_count=0, max_length=0, len=s.length();
        int[] letters = new int[26];
        for(int right=0; right<len; right++){
            char c = s.charAt(right);
            letters[c-'A']++;
            max_count = Math.max(max_count, letters[c-'A']);
            while(right-left+1> k+max_count){
                char l_c = s.charAt(left);
                letters[l_c-'A']--;
                left++;
            }
            max_length = Math.max(max_length, right-left+1);
        }
        return max_length;
    }
}
```