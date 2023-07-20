---
title: Java中算法所用语法总结
date: 2022-01-20 12:23:21
categories: 
- Algorithm
tags:
- Java
- Algorithm
description: (java刷题还是没cpp舒服)本篇文章主要讲述了Java在刷算法题时的应用总结，包括输入输出、字符串、容器等等。对于以上的每个方面，都有详细介绍以便在需要时查阅。
top_img: 
cover: /image/covers/java.png
---


## 基本输入输出

```java
// 基本输入
Scanner in = new Scanner (System.in);
Scanner in = new Scanner (new BufferedInputStream(System.in));//更快

// 字符串输入
String str = in.next();
// int long double 大位数 高精度
int i = in.nextInt();
long j = in.nextLong();
double k = in.nextDouble();
BigInteger l = in.nextBigInteger();
BigDecimal m = in.nextBigDecimal();

//输出
PrintWriter out = new PrintWriter(new BufferedOutputStream(System.out));//使用缓存加速，比直接使用System.out快
out.println(n); 
out.printf("%.2f\n", ans); // 与c语言中printf用法相同
```

## 大整数与高精度

### 大整数

```java
import java.math.BigInteger; 
//主要有以下方法可以使用： 
BigInteger add(BigInteger other) 		// +
BigInteger subtract(BigInteger other) 	// -
BigInteger multiply(BigInteger other) 	// *
BigInteger divide(BigInteger other)		// /
BigInteger [] dividedandRemainder(BigInteger other) //数组第一位是商，第二位是余数
BigInteger pow(int other)				// other次方
BigInteger mod(BigInteger other) 		// 模
BigInteger gcd(BigInteger other) 		// 最大公约数
int compareTo(BigInteger other) 		//负数则小于,0则等于,正数则大于
static BigInteger valueOf(long x)
//输出数字时直接使用 System.out.println(a) 即可
```

### 高精度

```java
BigDecimal add(BigDecimal other)
BigDecimal subtract(BigDecimal other)
BigDecimal multiply(BigDecimal other)
BigInteger divide(BigInteger other)
BigDecimal divide(BigDecimal divisor, int scale, BigDecimal.ROUND_HALF_UP)//除数，保留小数位数，保留方法四舍五入
BigDecimal.setScale()方法用于格式化小数点 //setScale(1)表示保留一位小数，默认用四舍五入方式
```

## String, StringBuilder

### String

```java
// String

// 返回 char指定索引处的值
char charAt(int index) 

// 按字典顺序比较两个字符串。
// 如果String对象按字典顺序排列在参数字符串之前，结果为负整数。 结果是一个正整数，如果String对象按字典顺序跟随参数字符串。 如果字符串相等，结果为零;
int compareTo(String anotherString)

// 返回指定字符第一次出现的字符串内的索引。没有返回-1。
public int indexOf(int ch, [int fromIndex])
public int indexOf(String str)
    
// 返回一个字符串，该字符串是此字符串的子字符串。 子字符串以指定索引处的字符开头，并扩展到该字符串的末尾。
// 返回一个字符串，该字符串是此字符串的子字符串。 子串开始于指定beginIndex并延伸到字符索引endIndex - 1 。 因此，子串的长度为endIndex-beginIndex 。
public String substring(int beginIndex)
public String substring(int beginIndex, int endIndex)

// 返回从替换所有出现的导致一个字符串oldChar在此字符串newChar 。
public String replace(char oldChar, char newChar)

// 将此字符串拆分为给定的regular expression的匹配。
public String[] split(String regex, int limit)
public String[] split(String regex)

// 将此字符串转换为新的字符数组。
public char[] toCharArray()
```

### StringBuilder

```java
// StringBuilder

// 将参数附加到此序列。
public StringBuilder append(许多重载)
// 返回char在指定索引在这个序列值。 第一个char值为索引0 ，下一个索引为1 ，依此类推，与数组索引一样。
public char charAt(int index)
// 删除此序列的子字符串中的字符。 子串开始于指定start并延伸到字符索引end - 1
public StringBuilder delete(int start, int end)
public StringBuilder deleteCharAt(int index)
// 用指定的String中的字符替换此序列的子字符串中的String 。 子串开始于指定start并延伸到字符索引end - 1 
public StringBuilder replace(int start, int end, String str)
// 将Object参数的字符串表示插入到此字符序列中。
public StringBuilder insert(int offset, Object obj)
// 反转
public StringBuilder reverse()
// 返回字符串
public String toString()
// 返回长度
public int length()
// 设置长度
public void setLength(int newLength)
//返回char在指定索引在这个序列值。
public char charAt(int index)
// 指定索引处的字符设置为ch 。
public void setCharAt(int index,  char ch)
// 返回一个新的String ，其中包含此字符序列中当前包含的字符的子序列。 子串从指定的索引开始，并延伸到该序列的末尾。
public String substring(int start， int end)
```

## 进制转换

```java
String s = Integer.toString(a, x); //把int型数据转换乘X进制数并转换成string型
int b = Integer.parseInt(s, x);//把字符串当作X进制数转换成int型
```

## 排序

```java
Arrays.sort(int[] a, int fromIndex, int toIndex)
Collections.sort(List<T> list, Comparator<? super T> c)

// 排序举例
// 整型数组从大到小
class MyComparator implements Comparator<Integer>{
    //返回值为负则o1排在o2前面，反正在后面，为0则表示相等
    @Override
    public int compare(Integer o1, Integer o2) {
        //如果n1小于n2，我们就返回正值，如果n1大于n2我们就返回负值，
        //这样颠倒一下，就可以实现反向排序了
        if(o1 < o2) { 
            return 1;
        }else if(o1 > o2) {
            return -1;
        }else {
            return 0;
        }
    }
}
```

## C++中的容器在Java中的用法

![容器](/image/rongqi.png)

|      C++       |     Java      |              说明              |
| :------------: | :-----------: | :----------------------------: |
|     vector     |   ArrayList   |          可变长度数组          |
|      list      |   LinkList    |              链表              |
|     deque      |  ArrayDeque   |            双端队列            |
|     stack      |     Stack     |               栈               |
|     queue      |     Queue     |              队列              |
| priority_queue | PriorityQueue |          支持优先队列          |
|      set       |    TreeSet    | 集合、数据有序、二叉搜索树实现 |
| unordered_set  |    HashSet    |        哈希表组织的set         |
|                | LinkedHashSet |    按插入有序，支持哈希查找    |
|      map       |    TreeMap    |     键值对映射，按key有序      |
| unordered_map  |    HashMap    |         hash组织的map          |
|                | LinkedHashMap |    按插入有序，支持哈希查找    |



### Vector -> ArrayList

- 可调整大小的数组的实现`List`接口。

 ```java
  // 返回此列表中的元素数。
  public int size()
  public boolean isEmpty()
  public boolean contains(Object o)
  // 返回此列表中指定元素的第一次出现的索引，如果此列表不包含元素，则返回-1。 
  public int indexOf(Object o)
  public E get(int index)
  public E set(int index, E element)
  public boolean add(E e)
  public void add(int index, E element)
  public boolean addAll(Collection<? extends E> c)
  // 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
  public boolean addAll(int index, Collection<? extends E> c)
  public E remove(int index) // 返回被删除元素
  protected void removeRange(int fromIndex,  int toIndex)
  // 将指定的元素追加到此列表的末尾。
  public boolean add(E e)
  public void add(int index, E element)
  public void clear()
 ```

### list -> LinkedList

- 用法基本同ArrayList, 但底层实现为链表

### ArrayDeque

- 可以用此数据结构实现栈、队列和双端队列

- **不允许**null元素

 ```java
  // 当作为栈的时候
  public void push(E e) 	// 向队首插入元素
  public E pop()			// 弹出返回队首元素
  public E peek()			// 检索队首元素
  // 当作为队列时
  public boolean offer(E e)	// 向队尾插入元素
  public E pop()				// 弹出返回队首元素
  public E peek()				// 检索队首元素
  
  // 当作为双端队列时
  public void addFirst(E e)	// 向队首插入元素
  public void addLast(E e)	// 向队尾插入元素
  public E getFirst()
  public E getLast()
  public E removeFirst()
  public E removeLast()	
 ```

### PriorityQueue

- 基于优先级堆的无限优先级**queue**。 

- **不允许**null元素

- 小顶堆

 ```java
  // 优先队列举例
  // 实现大顶堆排序
  Comparator<Object> Order = new Comparator<>() {
      public int compare(Object n1, Object n2) {
          if ((int) n1 > (int) n2) return -1;
          else if ((int) n1 == (int) n2) return 0;
          else return 1;
      }
  }
  PriorityQueue<Integer> pq = new PriorityQueue<>(10, Order);
  pq.add(1);	// 添加元素
  pq.add(3);
  pq.add(2);
  System.out.println(pq.peak());
  System.out.println(pq.poll());
  System.out.println(pq.peak());
  System.out.println(pq.poll());
  System.out.println(pq.peak());
  System.out.println(pq.poll());
  /*
  输出： 3 3 2 2 1 1
  */
  // 若想对堆中元素进行遍历
  Object[] nums = pq.toArray();
  Arrays.sort(nums, Order);
  for (int i = 0; i < nums.length; i++) System.out.println(nums[i]);
  /*
  输出： 3 2 1
  */
 ```

### 迭代器

 ```java
  Collection<Integer> c = new Collection<>();
  Iterator iter = c.Iterator();
  while (iter.hasNext()) {
      Integer temp = iter.next();
      System.out.println(temp);
  }
 ```

### HashSet & TreeSet

- 不会存储重复的元素

- TreeSet 保证集合有序，基于红黑树。而HashSet 基于哈希表实现。

 ```java
  // TreeSet
  // 构造函数
  TreeSet(Comparator<? super E> comparator)
  TreeSet(SortedSet<E> set)
  TreeSet(Collection<? extends E> collection)
  // 增加元素
  boolean add(E object)
  boolean addAll(Collection<? extends E> collection)
  // 弹出
  E pollFirst()
  E pollLast()
  
  public boolean remove(Object o)
  int size()
  // 等等
      
  
  // HashSet
  public boolean add(E e)
  public int size()
  public boolean remove(Object o)
  public boolean isEmpty()
  public void clear()
  public boolean contains(Object o)
 ```

### HashMap & TreeMap

- 键值对

- Map是java中的接口，Map.Entry是Map的一个内部接口。

- 由于Map中没有实现Iterator接口，故需要转化为set类型，用entrySet()方法。

- TreeSet不允许有重复的Key，重复put只会覆盖上一个值

 ```java
  Map<String, String> map = new HashMap<String, String>();
  // 插入键值对
  map.put("2", "value1");
  map.put("1", "value2");
  map.put("3", "value3");
  
  // 遍历方法1：遍历KeySet, 取值 
  for (String key : map.keySet()) {
      System.out.println(map.get(key));
  }
  
  // 遍历方法2：通过Iterator
  Iterator<Map.Entry<String, String>> iter = map.entrySet().iterator();
  while (iter.hasNext()) {
      Map.Entry entry = iter.next();
      System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
  
  // 遍历方法3：直接遍历Entry
  for (Map.Entry<String, String> entry : map.entrySet()) {
      System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
  
  // 遍历方法4：仅仅遍历Value
  for (String v : map.values()) {
      System.out.println("value= " + v);
  }
 ```

 ```java
  // LeetCode 347 返回前K高频元素
  class Solution {
      public int[] topKFrequent(int[] nums, int k) {
          Map<Integer, Integer> map = new TreeMap<>();
          for (int num : nums) {
              if (map.containsKey(num)) {
                  map.put(num, map.get(num) + 1);
              }
              else map.put(num, 1);
          }
          Set<Map.Entry<Integer, Integer>> set = map.entrySet();
          PriorityQueue<Map.Entry<Integer, Integer>> pq = new PriorityQueue<>((o1, o2) -> (o1.getValue() - o2.getValue()));
          for (Map.Entry<Integer, Integer> entry : set) {
              pq.offer(entry);
              if (pq.size() > k) {
                  pq.poll();
              }
          }
          int[] result = new int[k];
          for (int i = k - 1; i >= 0; i--) result[i] = pq.poll().getKey();
          return result;
  
      }
  }
 ```

  
