* Java Generics
    * 设想我们要设计一个MyList集合。
        1. List在设计的时候，类型属性全都是Object(因为Object是所有的父类)
            1. Object[] array; Object get(int idx); void add(Object o);
            2. 这里的问题是，你随便add任意类型，随便get转成任意类型，容易出错。用户接口不友好。编译器没有太多办法帮你做类型检查。
        2. 为了能够得到类型检查，另外一种方案。为每个类型都单独搞一类。MyList_String, MyList_Integer。缺点明显。代码看起来太蠢了。 
    * 要解决以上的问题，Java提供了一种机制，generic type。<T>在尖括号里放的是type parameter。
        1. 作用：编译器能够类型检查，然后用户又不需要为每一种类型都写出一个类。
        2. 要注意，这种检查只发生在compile time: List<Integer> l, 但你又l.add("sdfsd")，编译器就不让你通过。
            1. 编译的时候，编译器会把所有的类型，都换成它能看到的最高实际类。
            2. 比如<T>，就是Object，<E extends Comparable>，就是Comparable
                1. class BStack<E extends Comparable>{E[] s} => class BStack{Comparable[] s}
```aidl
        List list = new ArrayList();
        list.add("here's a string");
        Integer i = (Integer) list.get(0); //runtime exception

        List<String> list2 = new ArrayList<>();
        list2.add("sdf");
        Integer i2 = list2.get(0); //compile error
        list2.add(4);  //compile error
```
* 为什么Map interface里的key是Object类型而不是K类型: V get(Object key);
    * 这么设计的原因是，设计者并不想要求这个key一定要跟我声明的K类型一样。
    * 它只要跟我put进去的key是equals的就okay。所以你看put的声明，key是K类型的：V put(K key, V value);
    * 比如说，你声明Map<ArrayList, String>，但你是可以get() based on LinkedList的。因为他们内容上可以“一样”。
    * 但也有说法说，那你上面声明Map<List, String>不就得了。
* static class
    * java里声明在一个class里的class，叫nested class。
    * static class只能声明在某一个类里面的。它的外面的类叫outer class.
    * static class只能访问outer class的staic member
    * static class can be instantiated without instantiating its outer class:  A.AInner1 a1 = new A.AInner1();
    * 声明在类里面的，也能是non-static class。但是它就只能在outer class里被实例化，外面不行。然后outer class的member无论属性它都可以访问。
    * 例子： HashMap里的static class Node, 因为这个类不是public的，所以不能在外部实例化。只能在HashMap class内被实例化。
    