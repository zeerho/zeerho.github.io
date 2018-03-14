---
title: 由HashMap和HashTable说开.md
date: 2018-03-14 22:55:16
tags: [Java]

---

# HashMap 和 HashTable 的区别

- 父类不同，不过这点不重要。
- HashMap 的键、值可为 null，HashTable 都不行。
- HashMap 非线程安全，HashTable 通过在所有闭包方法上加上 synchronized 关键词来实现线程安全。
- 基于上一点，HashMap 的速度要明显更快。

除了这些区别外，还有一些细节但是重要的区别。

两者都是立足在哈希的基础上实现快速查找。两者的哈希方法有所不同。

**HashMap**

```java
static final int hash(Object key) {
    int h;
    // java8 之前多做了几次混淆
    // h ^= (h >>> 20) ^ (h >>> 12);
    // return h ^ (h >>> 7) ^ (h >>> 4);
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

**HashTable**

```java
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

HashTable 做了一次模运算，略慢一点。

HashMap 混淆了高低位的信息之后对低位进行掩码，而 HashTable 的散列程度直接取决于整体的哈希值的散列程度。也就是说，要实现相同的散列程度，HashTable 对于 key 对象本身的  hashCode() 方法要求更高。

# 对象的 hashCode() 方法

