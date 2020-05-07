Solution 1: 
```java
public class Persist {
    public static int persistence(long n) {
        int pCount = -0;
        long cur = n;
        while (true) {
            if(cur / 10 > 0){
                pCount++;
            }else{
                break;
            }
            cur = getDigitMultiResult(cur);
        }
        return pCount;
    }

    private static long getDigitMultiResult(long n) {
        long result = 1;
        long cur = n;
        while (cur != 0) {
            result *= (cur % 10);
            cur = cur / 10;
        }
        return result;
    }

}
```