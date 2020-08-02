java容器类总结
https://www.cnblogs.com/wishyouhappy/p/3669198.html

Android容器类小结
https://www.jianshu.com/p/fdbc9d05981e


------Java容器类------
Iterator
--ListIterator(implements)

Collection
--List(implements)
  --AbstractList(implements)
    --Vector(extends)
      --Stack(extends)
    --ArrayList(extends)
    --AbstractSequentialList(extends)
      --LinkedList(extends)
--Set(implements)
  --AbstractSet(implements)
    --HashSet(extends)
      --LinkedHashSet
    --TreeSet(extends)
--Queue(implements)
--AbstractCollection(implements)
  --AbstractList(implements)
    --Vector(extends)
      --Stack(extends)

Map
--AbstractMap(implements)
  --HashMap(extends)
  --LinkedHashMap(extends)
  --WeakHashMap(extends)
  --HashTable(extends)
  --IdentityHashMap(extends)
  --TreeMap(extends)
--SortedMap(implements)
  --TreeMap(implements)


------Android容器类------
Collection
--ArraySet(implements)

Set
--ArraySet(implements)

Map
--ArrayMap(implements)

Cloneable
--SparseArray(implements)
--SparseIntArray(implements)
--SparseBooleanArray(implements)
--SparseLongArray(implements)

ArraySet实现了Set和Collections接口，在api 23中添加
ArrayMap实现了Map接口，在api 19中添加
SparseArray，SparseIntArray，SparseBooleanArray实现了Cloneable接口，在api 1中添加
SparseLong实现了Cloneable接口，在api 18中添加

功能上划分为两类：
存储元素
ArraySet优化了HashSet对元素的存储

存储键值对
ArrayMap优化了HashMap存储 Object --> Object的键值存储;
SparseArray优化了 int --> Object的键值存储;
SparseIntArray优化了 int --> int的键值存储;
SparseBooleanArray优化了 int --> boolean的键值存储;
SparseLongArray优化了 int --> long的键值存储。

ArraySet, ArrayMap
使用数组mKeys存储key的hash值，hash值在mKeys的位置为index，并将value存储到mValues数组对应下标的
位置（ArrayMap中key和value分别在mValues的index * 2和index * 2 + 1的位置）。查找或者修改元素时，
使用二分查找在mKeys中找到元素在mValues的下标，然后进行修改或者返回。

SparseArray
使用int类型的mKeys数组存储int类型的键，下标为index，将Object类型的value存储在在Object类型的数组
mValues的index位置，在查找和修改时，使用二分查找在mKeys中找到元素在mValues的下标，然后进行修改或者
返回。在删除value时，SparseArray并不直接进行数组元素的移动，而是将待删除的value标记为DELETED状态，
在gc的过程中将所有非DELETED状态的元素移动到数组的最前面，减少二分查找的时间。

SparseIntArray, SparseLongArray, SparseBooleanArray
这3个容器可以理解成专用容器，使用int类型数组和对应类型的数组；使用二分查找快速查找元素，然后进行删除，
修改，添加操作。

使用建议：
兼容性
在组织结构中，可以看到，并不是所有的容器都是从api 1就开始提供的，在使用具体的容器时，需要考虑应用的兼容。
对性能的影响
虽然ArrayMap在删除时不直接使用移动元素的方式删除元素，但是在获取数组元素等操作中还是
对数量的限制
在对元素进行查找时会使用二分查找，元素数量较大（超过1000）时，查找效率会降低。

Android容器优化是使用int到其他类型的映射，使用数组保存着两个映射达到优化HashMap对k-v的存储。
这种优化适用于元素数量较少（少于1000）的情况。