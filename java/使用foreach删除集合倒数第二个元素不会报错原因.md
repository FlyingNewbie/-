# 使用foreach删除集合倒数第二个元素不会报错原因

## 1、foreach中是首先调用了集合实现类的Iterator的迭代

执行顺序如下：首先调用hasNext()，如果返回true，则调用next()返回下一个元素；如果返回false代表遍历结束



## 2、案例分析

### 2.1、代码如下

```
List<String> list = new ArrayList<String>();

    list.add("1");

    list.add("2");

    for (String item : list) {

        if ("1".equals(item)) {

            list.remove(item);

        }

}
```

ArrayList的迭代源码如下（此处删除了一个无关紧要的方法）:

```java
 private class Itr implements Iterator<E> {
        int cursor;       // 正在遍历的指针位置
        int lastRet = -1; // 最后一个返回结果的位置，-1为未返回
        int expectedModCount = modCount;
    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        //校验修改次数
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        //ArrayList底层采用Object数组实现，此处复制原数组
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        //维护指针位置
        cursor = i + 1;
        //返回指定的元素
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        //校验修改次数
        checkForComodification();

        try {
            //调用了ArrayList的remove方法
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            //此处维护了期望修改次数和已修改次数相等
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        //如果修改次数与期望修改次数不等则抛出异常
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```


for循环遍历第一遍：

​		调用Iterator的hasNext()方法，指针位置cursor的初始值为0，此时size为2，所以hasNext()返回true，

紧接着进入next()方法，next()方法里面第一步调用方法checkForComodification()校验当前修改次数和期望的修改次数是否相等，如果相等，则继续进行，否则抛出java.util.ConcurrentModificationException异常。初始化时

expectedModCount 和 modCount是相等的，此处是继续进行，返回指定位置的数值，开始进行

 if ("1".equals(item)) 操作，此时返回true开始进行list.remove(item)操作，ArrayList的remove()源码如下:

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

//快速删除
private void fastRemove(int index) {
    	//增加修改次数
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            //将指定的元素从原数组中删除，并维护相应的数组秩序
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // 最后一个交给垃圾回收器回收
    }
```

使用了remove方法后，1这个元素已经从数组中删除



foreach开始第二遍遍历:

​	使用迭代器中的next()方法后，此时cursor的值是1，size也是1，没有下一个元素，遍历结束，此时不会出现任何异常；如果是删除其他元素，则会继续进入next()方法中，在第一步checkForComodification()中就会校验抛出异常，因为expectedModCount的在ArrayList中的remove中没有进行维护，所以expectedModCount比modCount小，也就不相等；而迭代器中remove方法是有对expectedModCount的值进行维护的。





## 总结：

​	foreach循环遍历也是采用迭代器进行遍历的；foreach循环删除倒数第二个元素不会报错而且会成功只是特例；因此不能在foreach遍历时删除元素，只能在迭代器中进行元素的删除操作。