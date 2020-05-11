Solution 1:  
Pass  
```java
class Solution {

    public String longestPalindrome(String s) {
        if (s.length() < 2) {
            return s;
        }
        boolean dp[][] = new boolean[s.length()][s.length()];
        int start = 0;
        int end = 0;
        for (int i = 0; i < s.length(); i++) {
            dp[i][i] = true;
        }
        for (int j = 1; j < s.length(); j++) {
            for (int i = 0; i < j; i++) {
                if (s.charAt(i) == s.charAt(j)) {
                    if (j == i + 1) {
                        dp[i][j] = true;
                        if (j - i > end - start) {
                            start = i;
                            end = j;

                        }
                    } else {
                        dp[i][j] = dp[i + 1][j - 1];
                        if (dp[i][j] == true && j - i > end - start) {
                            start = i;
                            end = j;
                        }
                    }
                }
            }
        }
        System.out.println(s.substring(start, end + 1));
        return s.substring(start, end + 1);
    }
```