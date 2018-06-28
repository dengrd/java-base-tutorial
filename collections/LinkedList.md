### public class LinkedList\<E> extends AbstractSequentialList\<E> implements List\<E>, Deque\<E>, Cloneable, java.io.Serializable

![LinkedList类层次结构图](https://raw.githubusercontent.com/dengrd/images-repo/master/201806/LinkedList.png)


#### LinkedList :
　　底层使用双向链表结构实现，因此添加元素的时候不需要扩容操作，元素是分散在内存中的，这也导致LinkedList执行插入和删除操作快速（只需要
修改前后节点的引用就可以，不像ArrayList一样需要移动数组中的元素），随机访问慢（即不能通过下标直接找到该元素在内存中的位置，需要从头节点
或尾节点<二分法标判断头结点和尾结点哪个距离近> 开始遍历整个list才能找到该下标上的元素）。

　　类源码中有说 *Note that this implementation is not synchronized.If multiple threads access a linked list concurrently,
and at least one of the threads modifies the list structurally, it must be synchronized externally.* 即该类的实现不是同步的，
是非线程安全的。如果多个线程并发访问同一个LinkedList，并且最少有一个线程对该list做了结构性的修改（即新增或删除元素，改变某个元素的值不算
结构性改变），则必须要在外面进行同步。通常可以通过 **List list = Collections.synchronizedList(new LinkedList(...));** 来获取线程
安全的list。

　　同ArrayList一样，LinkedList也有fail-fast快速失败机制，在获取了iterator后再去修改list的结构会使得iterator在迭代的时候抛出并发修改
异常 ConcurrentModificationException 。注意，该机制并不能保证一定会阻止任何并发修改操作，它只能尽最大的努力去发现并阻止，因此我们不能
只依赖该机制来保证程序的正确性，该机制只应该用来检测bug。

　　LinkedList 还实现了Deque双端队列接口，因此可以作为队列和堆栈来使用，pop()获取并移除list中第一个元素，push()往list头部加入元素，
offer()往list尾部加入一个元素。


　　**适合场景**：需要频繁地进行新增或删除操作，读取操作比较少的地方，单线程环境。

### 源码解析：

```java
    //记录list的大小
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    //记录list的头结点，头结点的前一个结点一定为null
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    //记录list的尾结点，尾结点的后一个结点一定为null
    transient Node<E> last;
    
    //LinkedList静态内部类，代表list每一个元素的数据结构
    private static class Node<E> {
        E item;         //结点保存的元素的实际值
        Node<E> next;   //结点的下一个结点
        Node<E> prev;   //结点的上一个结点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

list尾部增加一个结点 add(E e)：

```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;         //记录之前最后的结点为l
        final Node<E> newNode = new Node<>(l, e, null);     //构造新的结点，前一个引用之前的最后结点l.
        last = newNode;     //改变类属性last指向这个新结点
        if (l == null)
            first = newNode;        //l==null说明添加元素前list没有一个元素，所以头结点也是该新结点
        else
            l.next = newNode;       //将之前最后的结点l的后一个指向该新结点
        size++;         //修改list大小
        modCount++;     //增加list修改次数
    }
```

list的某个位置插入一个元素 add(int index, E element)：

```java
    public void add(int index, E element) {
        checkPositionIndex(index);      //检查参数index是否合法

        if (index == size)
            linkLast(element);          //index为list的大小，等同于add(E e)方法
        else
            linkBefore(element, node(index));   //node(index)方法遍历找该位置的元素，在其前面增加一个结点
    }
    
    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;         //记录succ结点的前一个结点为pred
        final Node<E> newNode = new Node<>(pred, e, succ);     //新增一个结点，前一个指向pred,后一个指向succ
        succ.prev = newNode;        //succ的前一个指向修改为新增的结点
        if (pred == null)
            first = newNode;        //pred==null说明succ是头结点，因此新节点变为新的头结点
        else
            pred.next = newNode;    //将pred的后一个指向改为新的结点。
        size++;
        modCount++;
    }
```

其他的增加方法或移除方法类似，都是只需要关心操作位置前后的结点的引用关系的修改，其他的元素没影响，因此插入删除操作效率高


获取某个位置上的元素的值 get(int index):

```java
    public E get(int index) {
        checkElementIndex(index);   //检查索引是否合法
        return node(index).item;    
    }
    
    //遍历list获取index上的结点
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {                  //比较index与list的size/2 的大小，index在前半段则从头结点开始遍历
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {                                    //index在后半段，从尾结点开始遍历
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```