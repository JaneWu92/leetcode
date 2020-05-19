```java
class Solution {
    public int myAtoi(String str) {
        str = str.trim();
        if (str.length() == 0) {
            return 0;
        }
        char firstC = str.charAt(0);
        if (firstC != '+' && firstC != '-' && (firstC < '0' || firstC > '9')) {
            return 0;
        } else if ((firstC == '+' || firstC == '-') && str.length() == 1) {
            return 0;
        }
        //Below should be +***, -***, 9***
        int base = 1;
        int cur = 0;
        if (firstC == '+') {
            str = str.substring(1);
        } else if (firstC == '-') {
            str = str.substring(1);
            base = -1;
        }
        //str, base
        for (int i = 0; i < str.length(); i++) {
            char c = str.charAt(i);
            if (c < '0' || c > '9') {
                return cur;
            }
            int c_int = (c - '0');
            if ((base < 0) && ((cur < Integer.MIN_VALUE / 10) || base * c_int < Integer.MIN_VALUE - 10 * cur)) {
                return Integer.MIN_VALUE;
            }
            if ((base > 0) && ((cur > Integer.MAX_VALUE / 10) || base * c_int > Integer.MAX_VALUE - 10 * cur)) {
                return Integer.MAX_VALUE;
            }
            cur = base * c_int + 10 * cur;
        }
        return cur;
    }
}

```