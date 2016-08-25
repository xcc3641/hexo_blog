title:  Java 简单算法学习（慢填）
date: 2016-05-17 19:15:57
---



### 二分查找

```java
/二分法查找/折半查找
public static int binarySearch(int[] array, int key){
    int low = 0, high = array.length - 1, middle;
    while(low <= high){
        middle = (low + high) / 2;
        if(array[middle] == key){  //找到,返回下标索引值
            return middle;
        }else if(array[middle] > key){  //查找值在低半区
            high = middle - 1;
        }else{  //查找值在高半区
            low = middle + 1;
        }
    }
    return -1;  //找不到
}

```
