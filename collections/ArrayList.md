### public class ArrayList\<E> extends AbstractList\<E> implements List\<E>, RandomAccess, Cloneable, java.io.Serializable

#### ArrayList :
　　动态数组列表,底层采用数组的结构来存储元素，因此ArrayList可以支持随机访问，有序（元素插入顺序）,查询快速，随机插入删除慢（设计到底层数组的移动，在一个位置增加元素后，则该位置
往后的所有元素都要往后移动一位，删除同理）。 默认容量为10，超过容量后会自动进行数组拷贝，将容量扩大至原来的1.5倍。

　　适合场景：需要快速随机访问元素，改动比较少，而且是单线程环境下，因为ArrayList是线程不安全的。

```java
        /**
         * 默认初始化容量大小 10
         */
        private static final int DEFAULT_CAPACITY = 10;
    
        /**
         * 实例化空的ArrayList时共享该空数组。
         */
        private static final Object[] EMPTY_ELEMENTDATA = {};
    
        /**
         * 实例化默认大小的空ArrayList时共享该空数组。这个属性与上一个的区别是当往ArrayList里添加第一个元素时，可以知道将ArrayList扩充到多大。
         */
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
        /**
         * ArrayList中保存元素的Object数组。ArrayList的容量就是该数组的长度。任何一个使用DEFAULTCAPACITY_EMPTY_ELEMENTDATA初始化的空ArrayList在第一个元素被添加进来时，
         * 都会被扩充至 默认大小 DEFAULT_CAPACITY (10) 。
         */
        transient Object[] elementData; // non-private to simplify nested class access 没有使用private是为了简化嵌套类对属性的访问。
    
        /**
         * The size of the ArrayList (the number of elements it contains).
         * ArrayList的大小（列表中包含的元素数量）
         * @serial
         */
        private int size;
```

### 简述ArrayList部分方法执行流程

　　1. add(E): add方法向list中添加一个元素到末尾，首先会调用ensureCapacityInternal方法来确保当前list能装下现有大小size + 1 个元素，如果容量不够，则进行扩容操作（数组拷贝到新
    的大小的数组里），然后直接将元素赋值在size++ 的位置。
    
```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!! 记录结构修改次数，迭代的时候有用到。
        elementData[size++] = e;    //赋值元素e到数组的最后一位
        return true;
    }
    
    /**
     * Increases the capacity of this <tt>ArrayList</tt> instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     * 该方法再JDK1.8中好像没有被用到.
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default ele/ment table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    /**
     * 计算出实际的最小容量
     * 1.当list是初始化状态时，判断参数传进来的minCapacity是否大于10，比如list第一次调用add(E)时，minCapacity = 0 + 1 = 1;所以第一次扩容会直接扩到 10 。
     *  当空list初始化后第一次调用addAll()时，则会由参数里的集合的大小来决定，大于10，则保留minCapacity ，否则 minCapacity = 10；
    */
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
        }

    //add方法中调用，传入最少需要的容量（当前list大小加一），如果当前数组大小不够则扩容。
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    // 判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);  //真正执行扩容操作的方法
    }
    
     /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);     //数组原大小右移一位（除以2）,即新的容量为原来的1.5倍.
        if (newCapacity - minCapacity < 0)                      //保证扩大1.5倍后满足最小容量需求
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);      //复制数据到新的数组里
    }
```

　　2. get(int): get方法获取特定位置上的元素，由于数组是连续的，所以可以直接访问到该元素，时间复杂度为 O(1)

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * @param  index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        rangeCheck(index);      //检查索引参数的合法性，超出数组大小时会抛出著名的 IndexOutOfBoundsException

        return elementData(index);
    }
```

　　3. iterator(): iterator方法返回 ArrayList.Itr 的一个实例，该类实现了自己的迭代逻辑，在迭代的过程中会检查 modCount是否与刚获取iterator时的expectedModCount一致，
    如果不一致，说明在迭代的过程中有修改过list的结构，会直接抛出并发修改异常 ConcurrentModificationException . 如果需要在迭代的过程中对list进行元素的移除，可以调用iterator.remove()
    方法，该方法会移除当前迭代过的那个位置的元素（因为该方法内部会同时更新 expectedModCount）.
    
　　4. listIterator(): listIterator方法返回 ArrayList.ListItr 的一个实例，该迭代器类继承自 ArrayList.Itr，在 ArrayList.Itr 的所有功能上还实现了往前迭代 previous()
    并且提供了 set(E) 修改上一个迭代过的位置的元素 和 add(E) 在当前游标位置加入一个元素的方法.
    

## 注意点

　　1. 使用subList(fromIndex,toIndex) 方法返回的子列表是一个 ArrayList.SubList 的实例，该类直接持有调用subList()方法的对象的引用，所以在这个子列表上的结构操作会直接反映
    到原来列表上。
    利用这个特性可以删除list中的某一段数据:    **list.subList(from,to).clear();**

```java
    private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;

        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            this.parent = parent;  //持有父类对象的引用
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }

        public E set(int index, E e) {
            rangeCheck(index);
            checkForComodification();
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }

        public int size() {
            checkForComodification();
            return this.size;
        }
        
        //......
    }
```