

## 集合
* HashSet: add, remove

* HashMap: put, remove

* convert of Array and List
    * int[] to List
        * List<Integer> list = Arrays.stream(nums).boxed().collect(Collectors.toList());
    * Integer[] to List
        * List<Integer> list = Arrays.stream(nums).collect(Collectors.toList());
    * List to int[]
        * int[] arr = list.stream().mapToInt(i->i).toArray();
    * List to Integer[]
        * Integer[] arr = list.stream().toArray();
        
* Queue, Stack, Deque
```
Queue 方法	等效 Deque 方法
add(e)	addLast(e)
offer(e)	offerLast(e)
remove()	removeFirst()
poll()	pollFirst()
element()	getFirst()
peek()	peekFirst()


Stack方法	等效 Deque 方法
push(e)	addFirst(e)
pop()	removeFirst()
peek()	peekFirst()
```
* Stack用什么实现比较好
    * Deque<TreeNode> stk = new LinkedList();
* Queue的remove/poll, add/offer, element/peek区别
    * 旧的remove, add, element在数组为空的时候都会抛异常
    * 新的poll, offer, peek在数组为空的时候是会返回null
    * 注意，这都是Queue的。在Stack是没有的
    
* stack and queue API and implemention
    * stack
        * push, pop, peek  --> Stack
    * queue
        * offer, poll, peek  --> LinkedList

* reverse
    * Collections.reverse(list);
## ASCII
A~Z .. a~Z
65 ~ 122 = 57 
ASCII码表 0-127
int一般来说32位，
long64位。


## 位操作
位操作一定要加括号。
左移右移： <<
与和或：   &, |
```
                if((1L << c & second) == 0){
                    second = (1L << c) | second;
```
        