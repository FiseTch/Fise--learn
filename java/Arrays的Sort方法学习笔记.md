### `sort`方法的源码解读

-----

#### `sort`方法的描述

-----

> `sort`方法主要用于各种类型数组的排序。



#### `sort`方法的入参

----

```java
    public static void sort(int[] a) {
    }

 
    public static void sort(int[] a, int fromIndex, int toIndex) {
    }


    public static void sort(long[] a) {
    }


    public static void sort(long[] a, int fromIndex, int toIndex) {
        
    }


    public static void sort(short[] a) {
    }


    public static void sort(short[] a, int fromIndex, int toIndex) {    
    }

    public static void sort(char[] a) {
    }


    public static void sort(char[] a, int fromIndex, int toIndex) {
    }


    public static void sort(byte[] a) {
    }


    public static void sort(byte[] a, int fromIndex, int toIndex) {        
    }

    
    public static void sort(float[] a) {       
    }

    public static void sort(float[] a, int fromIndex, int toIndex) {
    }


    public static void sort(double[] a) {
    }

    
    public static void sort(double[] a, int fromIndex, int toIndex) {       
    }

```





#### `sort`方法的出参

---

>`sort`方法所有的出参都为空，数组在调用`sort`方法后，自身的元素排列顺序就会是有序的
>
>

#### `sort`的调用方式

----


```java
	    int[] ints = new int[]{5, 2, 4, 3};
        Arrays.sort(ints);
        for (int i = 0; i < ints.length; i++) {
            System.out.println(ints[i]);
        }    
```



#### `sort`方法的实现

> `jdk1.8`的`sort`方法内部实现会根据排序数组本身的长度（阈值）来选择不同的算法
>
> 1. 当数组长度小于47时，内部默认使用插入排序来实现







