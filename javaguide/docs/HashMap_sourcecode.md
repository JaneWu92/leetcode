## code
```aidl
interface Map<K,V>{
    interface Entry<K, V>
}

class HashMap extends Map<K,V>{
// all constant should fined as static final 
// the access modifier mostly is default
static final int INITIAL_SIZE = 1 << 4;
static float DEFAULT_LOAD_FACTOR = 0.75;
private int capacity = INITIAL_SIZE;
private int threshold = INITIAL_THRESHOLD;
// 1. default 2. transient
transient int size;
trasient Node<K, V>[] table;
public V put(K key, V value){
    return putVal(hash(), key, value);
}
// return previous value(null if not contained this key previously)
public V putVal(int hash, K key, V value){
    Node<K,V> tab; Node<K,V> p; int n,i;
    // lazy resize, note that after resize tab should redirect to  table again
    if((tab=table) == null || tab.length == 0)
        n = (tab=resize()).length;
    // 2 branch: 1. that bucket is empty, 2. that bucket is not empty
    if(p=tab[i=hash&(n-1)] == null)
        tab[i] = new Node(hash,key,value);
    else{
        // 2 branch: head key same, head key different
        // several branch: 
        // 1. one node, 2. linkedlist, 3. tree
        // idea is find that existing node(if not existing node, create one), and replace with old one
        Node<K,V> e; K k;
        if(p.hash == hash && (p.key != null && (p.key == key || p.key.equals(key)))))
            e = p
        else if(p instanceOf TreeNode)
            e = putTreeVal()
        else{
        // should be linkedlist here
            Node<K,V> next;
            do{
                next = p.next;
                if(p == null){
                    p.next = new Node(hash, key, value, null);
                    break;
                }    
                else if(p.hash == hash && (p.key == key || (p.key != null && p.key.equals(key)) )){   
                    e = p;
                    break;
                }
                p = next;
            }while(true)
            // repleace old value with new value
            if(e!=null){
                V oldV = e.val;
                e.val = value;
            }
        }// end else
        // 还有一个size++和扩容不知道要放哪里
    }// end else
    
}// end putVal

int hash(K k){
    int oriHash = k.hashcode();
    int hash = (oriHash >> 16) & oriHash;
    return hash;
}

// the class is defined static may because by default the nested class should be static
// because the static class can be instantiated without the outer class
// and also it's defined default(package related) here so that outside its package it will not be accessed   
static class Node<K, V> implements Map.Entry<K, V>{
    // hash and key here are both final, which contains the idea that it's not decided to change once initialized
    // however value is not final, which is the case to replace old value
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
}
```

## points
### 为什么hashmap的capacity要是2的倍数
* 取模才能够直接用位与运算
* 分布的也才能够更均匀(1111能全部用上)  
比如：
```aidl
len=16 1111 2^4-1=15
len=14 1101 
如果key=3的话，
位与运算：上面是3，下面的1
模运算： 都是3
```
