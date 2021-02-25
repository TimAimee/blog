# HashMap结构

	JDK 1.7 HashMap 采用数组 + 链表的数据结构，
  	JDK 1.8 HashMap 采用数组 + 链表 + 红黑二叉树的数据结构，
  
## 简单说明
	
	数组+链表，链表过长，数组+红黑树;
	默认长度16，容量超过75%，翻倍扩容。 
	
## index的取法

	hash(key)->HashValue
	HashValue=HashValue~(HashValue>>>16) //避免进行长度&(下一步操作)，HashValue的高位没有参与。
	index=HashValue&(length-1)

## 线程问题
	
	HashMap 是线程不安全的。

	JDK 1.7，在数组扩容的时候，存在 Entry 链死循环和数据丢失问题。
	
	JDK 1.8，put 方法存在数据覆盖的问题
	
	HashTable 线程安全的，Synchronized 来保证线程安全
	CurrentHashMap 1.7 线程安全，Segment(可重入锁&分段锁)+Entry
	CurrentHashMap 1.8 线程安全，Segment(可重入锁&分段锁)+Entry，大量的利用了 volatile，final，CAS 等 lock-free 技术
 

 


