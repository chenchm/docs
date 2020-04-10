**数据结构**

`ArrayList`底层是使用数组实现的，数组是典型的顺序存储结构，具有以下特点

- 在物理内存中是连续存储的，在逻辑上是存储连续的顺序表
- 数组的优势是查询的时间复杂度为O(1)，只要通过首地址和偏移量就能访问数组中的元素。
- 数组的劣势是插入和删除时，最坏情况下的时间复杂度为O(n)，当向第一个位置插入或删除元素时，需要将数组中的其他元素向后或者向前移动一位。
- 数组的容量是固定的。当元素数量，不断增加时需要对数组进行扩容。

**常量以及成员变量**

```java
//通过无参构造函数创建时，数组的默认大小
private static final int DEFAULT_CAPACITY = 10;

//当初始容量为0时，elementData指向的数组
private static final Object[] EMPTY_ELEMENTDATA = {};

//通过无参构造函数创建时，elementData指向的数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//arrayList真正用于存储数据的数组
transient Object[] elementData; // non-private to simplify nested class access

//数组元素数量
private int size;

//结构性修改的次数
protected transient int modCount = 0;
```

**构造函数**

```java
//指定初始容量的构造函数
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        //若初始容量大于0，创建一个大小为初始容量的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //若初始容量等于0，将EMPTY_ELEMENTDATA数组赋值给elementData
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        //否则抛出非法参数异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//无参构造函数
public ArrayList() {
    //将DEFAULTCAPACITY_EMPTY_ELEMENTDATA数组赋值给elementData
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//通过集合来创建ArrayList
public ArrayList(Collection<? extends E> c) {
    //这里需要注意的是某些collection的toArray方法返回的不是Object类型数组
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

从上面的三种构造函数可以得出结论，无论通过哪种方式创建的ArrayList对象，其elementData数组的类型都是Object[].class

**主要方法**

```java
//将元素添加到数组最后
public boolean add(E e) {
    //确保数组容量足够添加下一个元素，当容量不足时会进行扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

//将元素添加到指定位置
public void add(int index, E element) {
    //对index进行判断是否超过数组长度，超过会抛出IndexOutOfBoundsException
    rangeCheckForAdd(index);

    //确保数组容量足够添加下一个元素，当容量不足时会进行扩容
    ensureCapacityInternal(size + 1);
    //之后会将指定index和之后的元素全部向后移动一位，再对指定位置赋值
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

public E get(int index) {
    //判断index是否越界
    rangeCheck(index);

    return elementData(index);
}

//删除指定位置的元素
public E remove(int index) {
    //判断index是否越界
    rangeCheck(index);

    modCount++;
    //获得要删除位置的元素，该方法会返回删除的元素
    E oldValue = elementData(index);

    int numMoved = size - index - 1; // 计算要向前移动的元素数量
    if (numMoved > 0)
        //将index之后的元素向前移动一位
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //将最后一个元素置空
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

//根据对象来删除元素
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            //通过对象的equals方法来判断是否为同一个对象
            //因为删除第一个对象后就直接return了，所以只会删除第一个匹配的对象
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void ensureCapacityInternal(int minCapacity) {
    //先通过calculateCapacity方法计算容量值
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

//判断是否是通过无参构造函数创建的对象第一次添加元素，如果是则返回默认大小10，否则返回minCapacity
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++; //将modCount加1

    //minCapacity值要大于当前数组长度才进行扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    //记录旧数组的容量
    int oldCapacity = elementData.length;
    //将新数组的容量扩大至1.5倍，oldCapacity >> 1相当于除以2，使用位运算的原因是位运算更快
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //进行一次数组拷贝
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```



**注意点**

1. ArrayList是线程不安全的
2. 使用for循环时进行删除元素操作，由于每删除一个元素后都会改变数组结构（元素位置移动）所以删除过程中可能会出现BUG。如果使用forearch循环时进行元素删除操作，会抛出ConcurrentModificationException，原因是foreach写法是对实际的Iterable、hasNext、next方法的简写，这里会做迭代器内部修改次数检查，因为上面的remove()方法会修改modCount的值，所以才会报出并发修改异常。要避免这种情况的出现则在使用迭代器迭代时（显示或for-each的隐式）不要使用ArrayList的remove，改为用Iterator的remove即可。

```java
public E next() {
    checkForComodification();
    try {
        E next = get(cursor);
        lastRet = cursor++;
        return next;
    } catch (IndexOutOfBoundsException e) {
        checkForComodification();
        throw new NoSuchElementException();
    }
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

