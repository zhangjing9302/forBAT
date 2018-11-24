# 一、HashMap与HashTable的关系

​       1.两者均继承自Map；都使用key-value键值对进行存储数据；两者的底层均为数据+链表，内存地址即为数组的下标索引

​       2.HashMap是线程不安全的，HashTable是线程安全的

​       3.HashTable不允许key值和value值为空，HashMap可以允许key-value为空（但是只允许一个key值为空）

​       4.HashMap继承的是AbstractMap，实现Map接口；HashTable继承Dictionary抽象类，实现Map接口

​       5.HashMap默认容量为16，扩充因子为0.75；HashTable默认容量为11，扩充因子为0.75

​      6.HashMap扩容为翻倍；HashTable扩容为翻倍+1；

​     7.HashMap使用contains()方法判断是否含有某个键；HashTable使用get()方法判断是否含有某个键

​     8.两者计算内存地址的映射方法不同

# 二、HashMap的源码分析

## 1.HashMap高性能的必须保证的三个方面：

   1）hash算法高效

   2）hash值到内存地址的算法较快（存值较快）

   3）根据内存地址可以较快的得到对应的取值（取值快）

### 1.如何保证hash算法高效？

1）计算key值算法

```
int hash = hash(key.hashCode());
public native int hashCode();
static int hash(int h){
	h ^= (h >>> 20) ^ (h >>> 12);
	return h ^ (h >>> 7) ^ (h >>> 4);
}
```

调用了Object类的hashcode()算法和HashMap的hash()方法。hashcode为native函数，可以认为是高性能的；hash方法是基于位计算的，所以也可以认为是高性能的。

2）通过hash值获取内存地址

```
int i = indexFor(hash, table.length);
static int indexFor(int h, int length){
	return h & (length - 1);
}
```

indexFor()函数通过将hash值和数组长度按位与直接得到数组索引。最后由indexFor()函数返回的数组索引直接通过下标便可取得相应的值

## 2.如何解决哈希冲突？

1）

2）HashMap方法维护着一个Entry数组。每个Entry对象包括key, value,next,hash几项。next指向另外的Entry对象。HashMap的put()方法中，当有冲突时，会将新的Entry对象安放在对应的索引下，该对象的next指向旧的Entry对象。HashMap实际上就是一个链表的数组

总结：基于HashMap的实现算法及解决冲突的方法，只要hashcode和hash()方法实现的比较好，能够尽可能减少冲突的产生。对HashMap的操作几乎等于对数组的访问，性能较好。但是如果在大量冲突产生的情况下，HashMap事实上就退化为几个链表，对HashMap的操作等价于遍历链表，此时性能很差。

## 3.HashMap的扩容方法

和ArrayList和Vector一样，这种基于数组的结构，不可避免的需要在数组空间不足时，进行扩展。

1）指定大小的构造函数

```
public HashMap(int initialCapacity)
public HashMap(int initialCapacity, float loadFactor)
```

初始容量即数组的大小，HashmMap会使用大于等于initialCapacity并且是2的指数次幂的最小整数作为内置数组的大小。负载因子又叫填充比，他是介于0和1之间的浮点数，它决定了Hash Map在扩容之前，其内部数组的填充度。默认情况下，HashMap的初始大小为16，负载因子为0.75。

2）扩容操作

HashMap扩容的代码如下：

```
void resize(int newCapacity){
	Entry[] oldTable = table;
	int oldCapacity= oldTable.length;
	if(oldCapacity == MAXMUM_CAPACITY){
		threhold = Integer.MAX_VALUE;
		return;
	}
	//建立新的数组
	Entry[] newTable = new Entry[newCapacity];
	//将原有数组转到新的数组中
	transfer(newTable);
	table = newTable;
	//重新设置阈值，为新的容量和负载因子的乘积
	threshold = (int)(newCapacity * loadFactory);
}
```

其中，数组迁移逻辑主要在transfer()函数中实现，该函数实现和注释如下：

```
void transfer(Entry[] newTable){
	Entry[] src = table;
	int newCapacity = newTable.length;
	//遍历数组内所有表项
	for(int j = 0; j < src.length; j++){
		Entry<K,V> e = src[j];
		//当该表项索引有值存在时，则进行迁移
		if(e != null){
			src[j] = null;
			do{
				//进行数据迁移
				Entry<K,V> next = e.next;
				//计算该表现在新数组内的索引，并放置到新的数组中
				//建立新的链表关系
				int i = indexFor(e,hash, newCapacity);
				e.next = newTable[i];
				newTable[i] = e;
				e = next;
			}while(e != null)
		}
	}
}
```

HashMap的扩容操作会遍历整个HashMap，应该尽量避免该操作发生，设置合理的初始因子和负载因子，可以有效的减少HashMap的扩容次数

# 三、如何遍历HashMap？

http://www.trinea.cn/android/hashmap-loop-performance/

public class TestHashMap {

​	public static void main(String[] args) {

​		Map<String,String> map = new HashMap<String,String>();

​		map.put("1", "value1");

​		map.put("2", "value2");

​		map.put("3", "value3");

​		map.put("4", "value4");

​		map.put("5", "value5");

​		//遍历map

​		//第一种方法

​		System.out.println("***********************");

​		for(String key:map.keySet()) {

​			System.out.println("key:"+key+",value:"+map.get(key));

​			

​		}

​		//第二种方法

​		System.out.println("***********************");

​		Iterator mapit=map.entrySet().iterator();

​		while(mapit.hasNext()) {

​			Map.Entry<String, String> mapEntry= (Entry<String, String>) mapit.next();

​			System.out.println("key:"+mapEntry.getKey()+",value:"+mapEntry.getValue());

​		}

​		

​		//第三种方法,容量大时推荐

​		System.out.println("***********************");

​		for(Map.Entry<String, String> mapEntry2:map.entrySet()) {

​			System.out.println("key:"+mapEntry2.getKey()+",value:"+mapEntry2.getValue());

​		}

​		//第四种方法

​		System.out.println("***********************");

​		for(String value:map.values()) {

​			System.out.println("value:"+value);

​			

​		}

​		

​		

​	}

​	

# 四、HashMap和HashMap的区别？

| HashMap               | HashSet            |
| --------------------- | ------------------ |
| 实现的是Map接口             | 实现的是Set接口          |
| 使用键值对存储数据             | 只存储对象              |
| 使用put()方法将与元素放入map中   | 使用add()将元素放入Set中   |
| HashMap中使用键对象来计算hash值 | 使用成员对象计算hashcode值。 |
| 获取对象较快，使用唯一的键值对来获取    | 较慢                 |
|                       | 不允许集合中有重复的值        |

将对象存储在HashSet之前，要先确保对象重写equals()和hashcode()方法，这样才能比较对象的值是否相等，以确保set中没有储存相等的对象。如果没有重写这两个方法，将会使用这个方法的默认实现。使用add()方法时，当元素重复时会立即返回false，成功添加返回true

## 1.HashSet源码分析

```
 //调用HashMap的containsKey判断是否包含指定的key
    //HashSet的所有元素就是通过HashMap的key来保存的
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    // 将元素(e)添加到HashSet中，也就是将元素作为Key放入HashMap中
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    // 删除HashSet中的元素(o)，其实是在HashMap中删除了以o为key的Entry
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
```

总结：HashSet是基于HashMap实现。可以看到，所有放入HashSet中的集合元素实际上由hashMap的key来保存，而HashMap则存储了一个PRESENT，它是一个静态的object对象。

五、HashMap与LinkedHashMap和TreeMap的区别？

| HashMap                        | LinkedHashMap   | TreeMap         |
| ------------------------------ | --------------- | --------------- |
| 使用键值获取，访问速度快                   | 较HashMap慢       | 实现的是SortMap接口   |
| 遍历时，无序                         | 使用Iterator遍历时有序 | 根据键值排序（默认按照升序）  |
|                                | 可以按照应用次数排序      | 使用Iterator遍历时有序 |
| 在Map中插入、删除和定位元素，Hash Map是最好的选择 | 输出数序与输入相同时可以使用  | 按照自然顺序遍历键       |

总结：

1.HashSet是通过HashMap实现的，TreeSet是通过TreeMap实现的，只不过是Set用的是Map的key，value为object的静态对象

2.Map的key和Set都有一个共同特性是集合的唯一性，TreeMap更是多了一个排序的功能

3.hashCode和equals()是HashMap用的，因为无需排序所以只需要关注定位和唯一性即可。

   a.hashcode是用来计算hash值的，hash值是用来确定hash表索引的

   b.hash表中的一个索引处存放的是一张链表，所以还是要通过equal放噶循环比较链上的每一个对象才可以真正定位到兼职对应的Entry

   c.put时，如果hash表中没有定位到，就在链表千加一个Entry，如果定位到了，则更换Entry中的value，并返回旧value

4.由于TreeMap需要排序，所以需要一个Comparator为键值进行大小比较，当然也是用Comparator定位的

   a.Comparator可以在创建TreeMap时指定

   b.如果创建时没有确定，那么就会使用key.compareTo()方法，这就要求key必须实现Comparable接口

  c.TreeMap是使用Tree数据接口实现的，所以使用compare接口就可以完成定位了

5.Collection没有get()方法来取得某个元素，只能通过Iterator()遍历元素

6.Set和Collection拥有一摸一样的接口

7.List可以通过get()方法获取元素值

8.一般使用ArrayList。用LinkedList构造堆栈、队列Queue

9.Map用put(k,v)/get(k)，还可以使用containsKey()/containsValue()来检查其中是否含有某个key/value

10.Map中元素，可以将key序列、value序列单独抽取出来

​     a.使用keySet()抽取key序列，将map中的所有keys生成一个Set

​     b.使用values()抽取value序列，将map中的所有values生成一个Collection

​     c.为什么一个生成Set，一个生成Collection？那是因为，key总是独一无二的，value允许重复

六、java集合框架

![WechatIMG1](/Users/zhangjing/git/WechatIMG1.png)

