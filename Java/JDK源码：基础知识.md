## JDK源码：基础知识
##### String  
1. `String` 中维护了一个字符数组 `private final char value[];`
2. 操作都会产生一个深拷贝副本
3. `intern` 可以将字符串移到字符串常量池中  
4. 字符串内容比较不要用 `==`
5. char 类型是数值类型？
6. "0".equals(0) => false

	```
	public boolean equals(Object anObject) {
	    if (this == anObject) {
	        return true;
	    }
	    if (anObject instanceof String) {
	        String anotherString = (String)anObject;
	        int n = value.length;
	        if (n == anotherString.value.length) {
	            char v1[] = value;
	            char v2[] = anotherString.value;
	            int i = 0;
	            while (n-- != 0) {
	                if (v1[i] != v2[i])
	                    return false;
	                i++;
	            }
	            return true;
	        }
	    }
	    return false;
	}
	```

7. `replace` 返回**替换所有字符**后的新字符串  
	`replaceAll` & `replace` 都有使用正则表达式的版本
	
	```
	public String replace(char oldChar, char newChar) {
	    if (oldChar != newChar) {
	        int len = value.length;
	        int i = -1;
	        char[] val = value; /* avoid getfield opcode */
	
	        while (++i < len) {
	            if (val[i] == oldChar) {
	                break;
	            }
	        }
	        if (i < len) {
	            char buf[] = new char[len];
	            for (int j = 0; j < i; j++) {
	                buf[j] = val[j];
	            }
	            while (i < len) { // 同时处理需要替换的字符和不需要替换的字符
	                char c = val[i];
	                buf[i] = (c == oldChar) ? newChar : c;
	                i++;
	            }
	            return new String(buf, true);
	        }
	    }
	    return this;
	}
	```
8. `String` 提供了正则表达式的包装，何种设计模式？

	```
	public boolean matches(String regex) {
	    return Pattern.matches(regex, this);
	}
	``` 

9. `split` & `join` 是一对相反的方法

	```
	String message = String.join("-", "Java", "is", "cool");
	// message returned is: "Java-is-cool"
	```

10. 比较复杂的方法 `toUpperCase` `toLowerCase` `split`  
注意 `split` 方法返回的是字符串数组  
11. `substring` 在 jdk 1.6 中没有创建新字符串，但在之后的版本中创建了新的字符串
12. 常用 API
	- `someString.split("\\s+");`
	- `int n = Integer.parseInt("10");`
	- `String repeated = StringUtils.repeat(str, 3);`
	- `int count = StringUtils.countMatched("11221", "1");`

##### StringBuffer
1. 默认长度为16  
2. 几乎所有方法(除了构造方法，和一部分读方法)被 Synchronized 修饰，一部分读方法没有加 `synchronized` 是因为调用了其他被 `synchronized` 修饰的同名方法，`readObject` 串行化未加 `synchroinzed` 不知道原因？
3. extends AbstractStringBuilder
4. `setCharAt` 可以修改字符数组  
5. 流、串行化
6. `String` 没有 `reverse` 方法，`StringBuffer` 有 `reverse`
7. 至于 `StringBuilder` 如何进行扩容或是 `append` 可以往内存处想，实际上会重新开辟内存空间，而不会直接覆盖原数组后面的地址，否则可能会破坏其他变量占用的内存

	```
	String test = "abcdefg";
	char b = test.charAt(0);
	b = 't';
	System.out.println(test); // 数值类型，不变
	
	public synchronized void setCharAt(int index, char ch) {
	    if ((index < 0) || (index >= count))
	        throw new StringIndexOutOfBoundsException(index);
	    toStringCache = null;
	    value[index] = ch;
	}
	```

##### StringBuilder
1. 扩容大概为两倍 `int newCapacity = value.length * 2 + 2;`
2. 默认长度为16
3. 线程不安全
4. JVM 优化问题

##### Arrays & Collections
1. Arrays.copyOf => System.arraycopy => native arraycopy
2. Arrays.binarySerach 如果是对浮点数进行二分搜索，则需要转换成对应的整型；<= + 1 - 1
3. Arrays.asList 返回 `Arrays` 的内部类 `ArrayList`
4. `toString` `deepToString` 区别是什么
5. Arrays.sort 
	-  插入排序 `private static final int INSERTIONSORT_THRESHOLD = 7;` 
	-  双枢纽快排；
	-  归并排序； 
	-  并行排序；
6. Collections 阈值

	```
	private static final int BINARYSEARCH_THRESHOLD   = 5000;
	private static final int REVERSE_THRESHOLD        =   18;
	private static final int SHUFFLE_THRESHOLD        =    5;
	private static final int FILL_THRESHOLD           =   25;
	private static final int ROTATE_THRESHOLD         =  100;
	private static final int COPY_THRESHOLD           =   10;
	private static final int REPLACEALL_THRESHOLD     =   11;
	private static final int INDEXOFSUBLIST_THRESHOLD =   35;
	```

##### ArrayList
动态数组

1. 默认容量 10
2. 扩容时每次变成 1.5 倍

##### LinkedList 
双向链表，默认容量 0

##### HashMap
主要使用了 数组 + 单向链表 + 红黑树的方式实现  

1. 成员变量
	- 默认容量 16
	- 树化阈值 8 并且总容量超过 64；否则可能导致 hash 不均匀
	- 去树化阈值 6

2. 成员方法
	- `hash(key)` key 内存地址 异或 内存地址的高16位移到低16位，目的是 hash 的更均匀，`null` 键的 hash 值是 0
	- `tableSizeFor` 确保数组大小为2的n次幂:
		- 将取模定位变成了&(n-1);
		- resize 不需要再次 hash 定位，直接 `e.hash & oldCap`，只会出现在原数组位置和原数组下标 + n 的位置
	
		```
		static final int tableSizeFor(int cap) {
			int n = cap - 1; // 使得如果本身是2的n次幂，计算后不变
			n |= n >>> 1;
			n |= n >>> 2;
			n |= n >>> 4;
			n |= n >>> 8;
			n |= n >>> 16;
			return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1; // 最后要得到2的n次幂，而不是2的n+1次幂-1
 		}
		```
	- `put` 
		1. null 键放在数组下标为0的位置上 
		2. hash 碰撞时遍历链表，如果发现key已经存在，则更新之，否则加入到链表末尾 
		3. 1.7 及之前是头插法，并发时成环导致死循环   
		4. 数组的初始化会延迟到第一个元素到来时执行  
	- `puzzle` HashMap(1000) 会怎样？ 1024 * 0.75 < 1000

##### HashTable
1. HashTable 不能接受为 null 的键或值   
2. HashTable 每个方法都被 Synchronized 修饰了  
3. HashMap 是 fail-fast 的，当有其他线程（或使用错误的API） 改变了HashMap的结构时，将会抛出 ConcurrentModificationException，HashTable 的 enumerator 的迭代器不是 fail-fast 的。  
4. 默认大小是 11

##### LinkedHashMap
1. 继承自 HashMap，实现了 Map 接口
2. 在 HashMap 的基础上加上了双向链表
3. LRU linkNodeLast accessNode 

	```
	protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
   }
   ```

	```
	linkNodeLast(Node e) {
		Node last = tail;
		tail = e;
		if (last == null) 
			head = e;
		else {
			e.before = last;
			last.after = e;
		}
	}
	
	accessNode(Node e) {
		Node last = tail;
		Node p = e, b = p.before, a = p.after;
		p.after = null;
		
		if (b == null) 
			head = a;     // head = null can be accepted
		else  
			b.after = a;
			
		if (a != null)
			a.before = b; // head.before = null can be accepted
		else 
			last = b;
		
		if (last == null)
			head = p;
		else {
			p.before = last;
			last.after = p;
		}
		
		tail = p;
	}
	
	removeNode(Node e) {
		Node p = e, b = p.before, a = p.after;
		p.before = p.after = null;
		if (b == null) 
			head = a;
		else
			b.after = a;
		
		if (a == null)
			tail = b;
		else
			a.before = b;
	}
	```

##### ConcurrentHashMap
1. 默认容量 16 成员变量基本与 HashMap 相同，但没有继承 HashMap
2. ConcurrentHashMap 不能插入 null 键或者 null 值
3. 1.7 中使用 segments extends ReentrantLock 分段锁
4. 在 1.7 中也会用到 CAS 对一些写采用原子方式，写操作需要获取锁，不能并行，读操作可以并行，写的同时也可以读，使得其并行度高于同步容器
4. 1.8 中使用 CAS，和少量 Synchronized 在 put 操作时锁链表
5. 1.8 中得到容器的容量是大致的近似值
6. 1.7 中得到容器的容量会在两次失败（modCount 数不一致）后锁住整个表
7. 1.8 中 resize 时会调动所有线程帮助一起 resize，如果一个线程正在处理一个链表，则将该数组位置设置为 ForwardingNode 下一个线程看到它时自动跳到数组下一个下标处。
8. 与 `Collections.synchronizedMap` 相比，迭代不用加锁，不回抛出 `ConcurrentModificationException` 
9. ConcurrentHashMap 里面有 MapReduce 有关的内容，引入 `ConcurrentHashMap` 除了解决并发，是否还解决了大容量存储的问题？ 
10. 弱一致性

	> ConcurrentHashMap 的迭代器不同于 CopyOnWriteArrayList 等，写时拷贝反映的是创建时的副本。ConcurrentHashMap 的迭代器在遍历过程中，内部的元素可能发生变化。  
	如果变化发生在已遍历的部分，迭代器不会反映出来，如果变化发生在未被遍历的部分，迭代器会发现并反映出来，这就是弱一致性。
	
