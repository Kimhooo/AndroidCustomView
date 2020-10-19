LinkedHashMap 源码分析：

~~~java
1.当有元素执行了map.put(key,value)的时候：  
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMapEntry<K,V> p =
            new LinkedHashMapEntry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
     private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
        LinkedHashMapEntry<K,V> last = tail;
        //尾巴节点指向新new出来的这个节点
        tail = p;
        if (last == null)
            //新建第一个节点时，head 头部指向 第一个节点
            head = p;
        else {
            //新建第二个节点时，收尾相连，尾巴tail永远指向最后一个节点
            p.before = last;
            last.after = p;
        }
    }
2.执行插入操作后进入afterNodeInsertion()方法，这个方法是在linkedHashMap中实现的
  void afterNodeInsertion(boolean evict) { // possibly remove eldest r
        LinkedHashMapEntry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {//removeEldestEntry 这个方法返回false 进不去好像
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
3.重点，get操作如何进行，访问后会将元素，这个LinkedHashMap 也重写了
   public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
  如果accessOrder 为true
 void afterNodeAccess(Node<K,V> e) { // move node to last
       //1.先断开被访问过的这个节点的before和after链
       //2.将访问节点链接到last后面，last就是tail 尾巴节点
       //3.将tail指向新的节点
        LinkedHashMapEntry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMapEntry<K,V> p =
                (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
~~~

LruCache 中如何删除元素

~~~java
LruCache.java
 public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            putCount++;
            size += safeSizeOf(key, value);
            previous = map.put(key, value);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }

        trimToSize(maxSize);
        return previous;
    }

    /**
     * @param maxSize the maximum size of the cache before returning. May be -1
     *     to evict even 0-sized elements.
     */
    private void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }

                if (size <= maxSize) {
                    break;
                }

                // BEGIN LAYOUTLIB CHANGE
                // get the last item in the linked list.
                // This is not efficient, the goal here is to minimize the changes
                // compared to the platform version.
                Map.Entry<K, V> toEvict = null;
                for (Map.Entry<K, V> entry : map.entrySet()) {
                    toEvict = entry;
                }
                // END LAYOUTLIB CHANGE

                if (toEvict == null) {
                    break;
                }

                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

            entryRemoved(true, key, value, null);
        }
    }
~~~

