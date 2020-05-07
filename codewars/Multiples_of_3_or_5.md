```java
public class Solution {
  public int solution(int number) {
    int num3 = 0;
    int num5 = 0;
    int num15 = 0;
    int result = 0;
    while(true){
      num3 += 3;
      if(num3 < number){
        result += num3;
      }else{
        break;
      }
    }
    while(true){
      num5 += 5;
      if(num5 < number){
        result += num5;
      }else{
        break;
      }
    }
    while(true){
      num15 += 15;
      if(num15 < number){
        result -= num15;
      }else{
        break;
      }
    }
  return result;   
  }
}
```