```
package MyHashMap;
import java.util.Map;
import java.util.Objects;

public class MyNode<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    MyNode<K,V> next;

    MyNode(int hash,K k,V value,MyNode<K,V>next){
        this.hash=hash;
        this.key =k;
        this.value=value;
        this.next=next;
    }

    @Override
    public K getKey() {
        return key;
    }

    @Override
    public V getValue() {
        return value;
    }
    
    @Override
    public V setValue(V value) {
        V oldVaue=this.value;
        this.value=value;
        return oldVaue;
    }

    //返回这个Node节点的hashCode
    public final int hashCode(){
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    @Override
    public boolean equals(Object obj) {
        if(obj==this)return true;
        //如果obj对象是Entry接口对象
        if(obj instanceof Map.Entry){
            //转化
            Map.Entry<?,?> o=(Map.Entry<?,?>)obj;

            //如果key和value都相等
            if(Objects.equals(key,o.getKey()) && Objects.equals(value,o.getValue()))
                return true;
        }
        return false;
    }
}
```
```
package MyHashMap;

public class MyHashMap<K,V> {
    /**
     * 没有红黑树的版本,主要是put方法和get方法
     */
    MyNode<K,V>[] table;

    int size=0;
    int threshold=12;
    //在构造函数里直接初始化MyNode数组，真正的HashMap , initialized on first use即第一次使用的时候进行初始化
    MyHashMap(){
        //在构造函数里初始化
        table = (MyNode<K,V>[])new MyNode[16];
    }

    /**
     * //获取hash值
     * @param key key值
     * @return
     */
    static final int hash(Object key){

        int h;
        //获取这个key的hash值并且 高16位异或低16位 得到hash值
        return (key==null) ? 0 :((h=key.hashCode()) ^ (h>>>16));
    }


    /**
     * 得到桶的索引
     * @param h key的hash值
     * @param length 当前Hashmap的长度
     * @return
     */
    static int indexFor(int h,int length){
        // h&(length-1) 当length总为2的n次方时，相当于 h%length
        return h&(length-1);
    }

    /**
     *  get 方法，返回
     * @param key
     * @return
     */
    public  V get(Object key){
        //如果key为null
        if(key==null)
            return getForNullKey();
        //获取key的hashCode
        int hash=hash(key);
        //遍历一个桶内的链表

        for(MyNode<K,V> node=table[indexFor(hash,table.length)];node!=null ;node=node.next){

            if(node.hash==hash && node.key.equals(key) || key==node.key)
                return node.value;
        }
        return null;
    }

    /**
     * 返回key为null的MyNode
     * @return
     */
    private V getForNullKey() {
        for(MyNode<K,V> myNode=table[0];myNode!=null;myNode=myNode.next){
            if(myNode.key==null ){
                return myNode.value;
            }
        }
        return null;
    }

    /**
     * put方法
     * @param key key值
     * @param value value
     * @return
     */
    public V put(K key,V value){
        //如果key为空值，那么总是放在第一个链表中
        if(key==null){
            return putForNullKey(value);
        }

        //获取当前key值的桶索引
        int hash=hash(key);
        int index=indexFor(hash,table.length);

        for(MyNode<K,V> node=table[index];node!=null;node=node.next){
            //如果在这个桶中某个MyNode节点的hash值和value值都相同，替换为新的value,返回旧的value
            if((node.hash==hash) && (node.key==key || node.key.equals(key))){
                V oldValue=node.value;
                node.value=value;
                return oldValue;
            }
        }

        //当前桶内没有key值相同的MyNode,就添加一个MyNode
        addMyNode(hash,key,value,index);
        return null;
    }

    /***
     * 向一个桶内添加新的MyNode
     * @param hash hash值
     * @param key  key值
     * @param value value值
     * @param index 桶的索引
     */
    private void addMyNode(int hash, K key, V value, int index) {
        //获取当前桶内的第一个MyNode
        MyNode<K,V> node=table[index];
        //采用头插法
        table[index]=new MyNode<K, V>(hash,key,value,node);
        size++;
        //如果size超过阀值，就扩容，resize
        if(size>=threshold){
            resize();
        }
    }

    /**
     * 扩容函数，不实现
     */
    private void resize() {
    }

    /**
     * put 键为null的函数，不实现
     * @param value
     * @return
     */
    public V putForNullKey(V value){
        for(MyNode<K,V> node=table[0];node!=null;node=node.next){
            if(node.key==null){
                V oldValue=node.value;
                node.value=value;
                return oldValue;
            }
        }

        //增加一个Node值
        addMyNode(0,null,value,0);
        return null;
    }

    public  static void main(String[] args){

        MyHashMap<Integer,Integer> myHashMap=new MyHashMap<Integer, Integer>();
        myHashMap.put(2,3);
        System.out.println(myHashMap.get(2));
    }
}

```
