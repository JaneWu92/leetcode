```aidl
interface Map<K,V>{
    interface Entry<K, V>
}
class HashMap extends Map<K,V>{
    
class Map.Entry<K, V>{
    K key;
    V value;
}
}
```