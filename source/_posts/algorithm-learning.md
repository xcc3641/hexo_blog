title:  数据结构和算法的个人路程
date: 2016-05-17 19:15:57
---

数据结构和算法一直都是自己的一个痛点。但是当自己沉下心，慢慢去学习或者复习的时候，反而开始体会到其中的一二乐趣。
我并不怎么聪慧，所以一点一点的开始学习。很多解法肯定不是最优的，以后功力足够再进行__优化__。

这篇是自己的学习算法的路程。
最基本的我不会过多阐述，一些思路和过程会通过注释的形式写出。

书籍阅读：
- [算法（第四版）](https://book.douban.com/subject/10432347/)
- [剑指 Offer](https://book.douban.com/subject/6966465/)
- [大话数据结构](https://book.douban.com/subject/6424904/)

<!--  more -->

### 目录

<!-- toc -->

- [目录](#目录)
- [基本算法](#基本算法)
	- [二分查找](#二分查找)
	- [简单选择排序](#简单选择排序)
	- [直接插入排序](#直接插入排序)
	- [快速排序](#快速排序)
- [题目](#题目)
	- [在 O(1) 时间删除链表结点](#在-o1-时间删除链表结点)
	- [调整数组顺序使奇数位于偶数前面](#调整数组顺序使奇数位于偶数前面)
	- [链表中倒数第k个结点](#链表中倒数第k个结点)
	- [反转链表](#反转链表)
	- [合并两个有序链表形成一个有序链表](#合并两个有序链表形成一个有序链表)
- [高质量代码](#高质量代码)
	- [规范性](#规范性)
	- [完整性](#完整性)
	- [鲁棒性](#鲁棒性)
- [额外读物](#额外读物)

<!-- tocstop -->



### 基本算法
#### 二分查找

```java
    /**
     * @param left 左边开始的 index
     * @param right 右边最后的 index
     * @param object 需要查找的数字
     * @param array 该数字容器
     */
    private static void find(int left, int right, int object, int array[]) {
        // 找到这个数组中间 index
        int middle = (left + right) / 2;
        if (array[middle] == object) {
            System.out.println("找到这个数字的位置在:" + middle);
            // array[middle] 大于需要查找的值
        } else if (array[middle] > object) {
            // 故该值在 0 —-> middle 这个区间
            right = middle - 1;
            // 递归
            find(0, right, object, array);
        } else {
            // 否则该值在 middle —-> 最右 这个区间,递归
            find(middle, right, object, array);
        }
    }
```

#### 简单选择排序

- [维基百科]((https://zh.wikipedia.org/wiki/%E9%80%89%E6%8B%A9%E6%8E%92%E5%BA%8F))

> 选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

![](http://ww3.sinaimg.cn/large/72f96cbagw1f7m8q9nfhcg202s0ab755.gif)

- 代码实现如下：

```java
    private static void sort(int[] data) {
        int size = data.length;
        for (int i = 0; i < size; i++) {
            int min = i;
            for (int j = i + 1; j < size; j++) {
                // 交换位置
                if (data[j] < data[min]) {
                    int temp = data[j];
                    data[j] = data[min];
                    data[min] = temp;
                }
            }
        }
    }
```



#### 直接插入排序
- [维基百科](https://zh.wikipedia.org/wiki/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)
> **插入排序**（英语：Insertion Sort）是一种简单直观的[排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。**插入排序** 在实现上，通常采用in-place排序（即只需用到O(1)的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

一般来说，**插入排序** 都采用in-place在数组上实现。具体算法描述如下：

1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
6. 重复步骤2~5

如果*比较操作*的代价比*交换操作*大的话，可以采用[二分查找法](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E6%B3%95)来减少*比较操作*的数目。该算法可以认为是**插入排序**的一个变种，称为[二分查找插入排序](https://zh.wikipedia.org/w/index.php?title=%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F&action=edit&redlink=1)。

![](http://ww4.sinaimg.cn/large/72f96cbagw1f7mfuov7c6g208c050q55.gif)

- 代码实现如下：

```java
  public static void insertion_sort( int[] arr ) {
      for( int i=0; i<arr.length-1; i++ ) {
          for( int j=i+1; j>0; j-- ) {
              if( arr[j-1] <= arr[j] )
                  break;
              int temp = arr[j];
              arr[j] = arr[j-1];
              arr[j-1] = temp;
          }
      }
  }
 ```

 #### 快速排序



算法概述 / 思路
快速排序一般基于递归实现。其思路是这样的：
1. 选定一个合适的值（理想情况中值最好，但实现中一般使用数组第一个值）, 称为 “枢轴”(pivot)。
2. 基于这个值，将数组分为两部分，较小的分在左边，较大的分在右边。
3. 可以肯定，如此一轮下来，这个枢轴的位置一定在最终位置上。
4. 对两个子数组分别重复上述过程，直到每个数组只有一个元素。
5. 排序完成。

![](http://ww1.sinaimg.cn/large/801b780agw1f7np89e4bfj207s05y0sy.jpg)

```java
public static void quickSort(int[] arr){
    qsort(arr, 0, arr.length-1);
}
private static void qsort(int[] arr, int low, int high){
    if (low < high){
        int pivot=partition(arr, low, high);        //将数组分为两部分
        qsort(arr, low, pivot-1);                   //递归排序左子数组
        qsort(arr, pivot+1, high);                  //递归排序右子数组
    }
}
private static int partition(int[] arr, int low, int high){
    int pivot = arr[low];     //枢轴记录
    while (low<high){
        while (low<high && arr[high]>=pivot) --high;
        arr[low]=arr[high];             //交换比枢轴小的记录到左端
        while (low<high && arr[low]<=pivot) ++low;
        arr[high] = arr[low];           //交换比枢轴小的记录到右端
    }
    //扫描完成，枢轴到位
    arr[low] = pivot;
    //返回的是枢轴的位置
    return low;
}
```

上面这个快速排序算法可以说是最基本的快速排序，因为它并没有考虑任何输入数据。但是，我们很容易发现这个算法的缺陷：这就是在我们输入数据基本有序甚至完全有序的时候，这算法退化为冒泡排序，不再是 O(n㏒n)，而是 O(n^2) 了。

- 更多可以参考[用 Java 写算法之五：快速排序](http://flyingcat2013.blog.51cto.com/7061638/1281614)
### 题目

#### 在 O(1) 时间删除链表结点

> 给定单向链表的头指针和一个结点指针，定义一个函数在 O(1) 时间删除该结点。

```java
public static void deletNode(Node head, Node target) {
        // 先判空
        if (head == null || target == null) {
            throw new RuntimeException("不能为空");
        }

        // 如果要删除的不是尾结点 此时时间复杂度 O(1)
        // 我觉得这里的思路很好，直接把下一个结点内容覆盖目标结点，不用遍历
        if (target.next != null) {
            // 找到 target 结点的下一个结点 next
            Node next = target.next;
            // 将 next 的值和指向都赋值给 target
            target.value = next.value;
            target.next = next.next;
        }

        // 链表只有一个结点 删除头结点(同样是尾结点)
        else if (head == target) {
            // 这里在 Java 引用传递的原因，如果要删除这种情况下的结点，需要有返回值。
            // 先这样的思路即可
            head = target = null;
        }
        // 多个结点 删除尾部结点
        else {
            Node temp = head;
            while (temp != null) {
                // 在当前结点的下一个结点是 target 的时候
                if (temp.next == target) {
                    // 将当前结点的 next 指向 null 即可删除 target
                    temp.next = null;
                }
                temp = temp.next;
            }
        }
    }
```


#### 调整数组顺序使奇数位于偶数前面

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

```java
    public static int[] reOrderArray(int[] array) {
        // 这个方法比较笨
        ArrayList<Integer> oddList = new ArrayList<Integer>();
        ArrayList<Integer> evenList = new ArrayList<Integer>();

        for (int anArray : array) {
            if (anArray % 2 == 0) {
                // 偶数
                evenList.add(anArray);
            } else {
                // 奇数
                oddList.add(anArray);
            }
        }
        System.out.println("奇数:"+oddList.size());
        System.out.println("偶数:"+evenList.size());

        for (int i = 0; i < oddList.size(); i++) {
            array[i] = oddList.get(i);
        }

        for (int i = oddList.size(); i < evenList.size() + oddList.size(); i++){
            array[i] = evenList.get(i-oddList.size());
        }

        return array;
    }

```

#### 链表中倒数第k个结点

> 输入一个链表，输出该链表中倒数第k个结点。

```java
    public static ListNode findKthToTail(ListNode head, int k) {
        // 利用栈的先进后出特性
        Stack<ListNode> stack = new Stack<ListNode>();
        // 判空
        if (head == null || k <= 0) return null;
        // 依次入栈
        while (head != null) {
            stack.push(head);
            head = head.next;
        }
        // 判断临界值
        if (k > stack.size()) return null;

        for (int i = 0; i < k - 1; i++) {
            stack.pop();
        }
        return stack.pop();
    }
```

接受老司机指导，推荐用快慢指针（记住这个经典的方法）来做这个问题，自己下来用代码实现了下：

```java
    public static ListNode findKthToTail2(ListNode head, int k) {
        // 两个指针的经典问题 找到倒数第 K 个 我们可以将 after 指针慢 first K 个结点
        // 当 first == null 的时候 此时 after 就是倒数第 K 个了
        ListNode first;
        ListNode after = null;
        int count = 0;
        first = head;
        while (first != null) {
            count++;
            if (count >= k) {
                after = head;
                head = head.next;
            }
            first = first.next;
        }
        return after;
    }
```



#### 反转链表

> 输入一个链表，反转链表后，输出链表的所有元素。

```java
    public ListNode ReverseList(ListNode head) {
        //当前节点是head，pre为当前节点的前一节点，next为当前节点的下一节点
        // pre --> head --> next1 --> next2
        if (head == null) return null;
        ListNode pre = null;
        ListNode next = null;
        while (head != null) {
            // 先暂存 当前结点(head) 的下一个结点 next
            next = head.next;
            // 然后将 当前结点(head) 的下一个结点指向上一个结点 pre
            head.next = pre;
            // 因为遍历,下一轮中的结点的上一个结点就是当前结点
            pre = head;
            // 然后把暂存的结点赋值给当前结点,让遍历继续
            head = next;
        }
        return pre;
    }
```



#### 合并两个有序链表形成一个有序链表

```java
    public static Node merge(Node list1, Node list2) {
        if (list1 == null) return list2;
        if (list2 == null) return list1;
        Node mergeHead = null;
        if (list1.value < list2.value) {
            mergeHead = list1;
            mergeHead.next = merge(list1.next, list2);
        } else {
            mergeHead = list2;
            mergeHead.next = merge(list1, list2.next);
        }
        return mergeHead;
    }
```

这里用到了递归，首先我们会找到一个头结点，进行同样的判断递归合并排序。

### 高质量代码

#### 规范性

- 书写清晰
- 布局清晰
- 命名合理

####  完整性

- 完成基本功能
- 考虑边界条件
- 做好错误处理

#### 鲁棒性

- 采取防御式编程
- 处理无效的输入

### 额外读物

- [算法时间复杂度计算](http://www.jianshu.com/p/99bac69fdd97)
