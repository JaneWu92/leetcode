```java
public class DRoot {
  public static int digital_root(int n) {
    int cur = n;
    while(cur / 10 > 0){
      cur = digitSum(cur);
    }
    return cur;
  }
  
  private static int digitSum(int n){
    int cur = n;
    int sum = 0;
    while(cur > 0){
      sum += cur % 10;
      cur = cur / 10;
    }
    return sum;    
  }
}
```