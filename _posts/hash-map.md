---
title: JAVA HashMap 原理浅析
date: 2017-02-23 12:13:37
tags:
---

众所周知,HashMap 是用 key 的 hash 算法来定位 value 的一种实现. 具体是怎么做的呢?来拆开看一看.

HashMap 有几个成员变量非常关键:
>HashMapEntry<K,V>[] table

实际存放数据数组,从声明中也可以看出来,每个节点至少包括key和value.初始长度16.

>float loadFactor

table的负载系数,默认是0.75.当table的使用率达到这个值就要对table进行扩容

>int size

当前数据的数量

>int threshold

	threshold = table.length * loadFactor
当size 达到这个值就对table进行扩容.

### put 过程
简略版:

对key取hashCode,然后hashCode 对 table.length 取模,得到一个0~(table.length-1)的下标,把键值包装成 HashMapEntry 存放在table的下标处.

这是最理想的情况,仅表示原理,现实中遇到很多问题.

完整版:


##### 取下标
对key取hashCode,对hashCode取模.

对于取 HashCode Android 和 JDK 有不同的做法.
Android 版使用的 的是 singleWordWangJenkinsHash().
JDK则是Object 自带的hashCode 方法,然后用高16位对低16位进行异或,用于增加低16位的随机性.

接下来的取模的操作:

为了能高效的取模,HashMap的length必须的2的倍数,这样length-1 的二进制表示为低位全1高位全0的形式.

exp:

```
4-1 == 3 == 11b;
16-1 == 15 == 1111b;
```

然后取模的时候

```java
h & (length-1);
```
简直一颗赛艇.

##### 冲突
此时table的下标也有了,这时面临另外一个问题,坑被人占了怎么办?
要知道,特别是table比较小的时候,比如初始时只有16,两个不同的key获得相同的下标概率还是很可观的.
总不能直接把占坑的踢出去吧,那就一个坑多坐几个人.

这个时候看 HashMapEntry 的结构

```java
static class HashMapEntry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        HashMapEntry<K,V> next;
        int hash;
}
```
可以看出来这是一个链表的结构, table里存在的是表头, next 指向下一个元素.遇到下标冲突是就遍历整个链表,如有有元素有相同key的就替换里面的value,如果没有就把当前元素插到表头.

##### 扩容

当size不停的++,总会遇到 size>=threshold ,这时就需要扩容,过程比较简单,创建一个新数组并将旧数组复制过去.
注意, newCapacity=table.length*2;

```java
    void resize(int newCapacity) {
        HashMapEntry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        HashMapEntry[] newTable = new HashMapEntry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```
### get 过程
get 与 put 的查找过程完全相同,找到了就将 value 返回.









