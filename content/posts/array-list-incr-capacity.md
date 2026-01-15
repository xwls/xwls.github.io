---
title: ArrayList 扩容规则
date: 2022-10-15 15:28:59+08:00
categories:
- Java
tags:
- 源码
- 算法
summary: Java 中的常用集合类 ArrayList 的扩容规则。
---

## 验证程序

```java
import java.lang.reflect.Field;
import java.util.ArrayList;

class Scratch {

    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            printInfo(list);
            list.add(String.valueOf(i));
        }
    }

    /**
     * 打印 ArrayList 的大小和容量
     */
    private static void printInfo(ArrayList<?> list) {
        Class<?> aClass = list.getClass();
        Field elementData = null;
        try {
            // 获取 ArrayList 中用来存储元素的数组
            elementData = aClass.getDeclaredField("elementData");
        } catch (NoSuchFieldException e) {
            throw new RuntimeException(e);
        }
        elementData.setAccessible(true);
        Object[] o;
        try {
            o = (Object[]) elementData.get(list);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        }
        // 输出当前大小和容量
        System.out.println("size = " + list.size() + ", capacity = " + o.length);
    }
}
```

## 执行结果

```text
size = 0, capacity = 0
size = 1, capacity = 10
size = 2, capacity = 10
size = 3, capacity = 10
size = 4, capacity = 10
size = 5, capacity = 10
size = 6, capacity = 10
size = 7, capacity = 10
size = 8, capacity = 10
size = 9, capacity = 10
size = 10, capacity = 10
size = 11, capacity = 15
size = 12, capacity = 15
size = 13, capacity = 15
size = 14, capacity = 15
size = 15, capacity = 15
size = 16, capacity = 22
size = 17, capacity = 22
size = 18, capacity = 22
size = 19, capacity = 22
size = 20, capacity = 22
size = 21, capacity = 22
size = 22, capacity = 22
size = 23, capacity = 33
size = 24, capacity = 33
size = 25, capacity = 33
size = 26, capacity = 33
size = 27, capacity = 33
size = 28, capacity = 33
size = 29, capacity = 33
size = 30, capacity = 33
size = 31, capacity = 33
size = 32, capacity = 33
size = 33, capacity = 33
size = 34, capacity = 49
size = 35, capacity = 49
size = 36, capacity = 49
size = 37, capacity = 49
size = 38, capacity = 49
size = 39, capacity = 49
size = 40, capacity = 49
size = 41, capacity = 49
size = 42, capacity = 49
size = 43, capacity = 49
size = 44, capacity = 49
size = 45, capacity = 49
size = 46, capacity = 49
size = 47, capacity = 49
size = 48, capacity = 49
size = 49, capacity = 49
size = 50, capacity = 73
size = 51, capacity = 73
size = 52, capacity = 73
size = 53, capacity = 73
size = 54, capacity = 73
size = 55, capacity = 73
size = 56, capacity = 73
size = 57, capacity = 73
size = 58, capacity = 73
size = 59, capacity = 73
size = 60, capacity = 73
size = 61, capacity = 73
size = 62, capacity = 73
size = 63, capacity = 73
size = 64, capacity = 73
size = 65, capacity = 73
size = 66, capacity = 73
size = 67, capacity = 73
size = 68, capacity = 73
size = 69, capacity = 73
size = 70, capacity = 73
size = 71, capacity = 73
size = 72, capacity = 73
size = 73, capacity = 73
size = 74, capacity = 109
size = 75, capacity = 109
size = 76, capacity = 109
size = 77, capacity = 109
size = 78, capacity = 109
size = 79, capacity = 109
size = 80, capacity = 109
size = 81, capacity = 109
size = 82, capacity = 109
size = 83, capacity = 109
size = 84, capacity = 109
size = 85, capacity = 109
size = 86, capacity = 109
size = 87, capacity = 109
size = 88, capacity = 109
size = 89, capacity = 109
size = 90, capacity = 109
size = 91, capacity = 109
size = 92, capacity = 109
size = 93, capacity = 109
size = 94, capacity = 109
size = 95, capacity = 109
size = 96, capacity = 109
size = 97, capacity = 109
size = 98, capacity = 109
size = 99, capacity = 109
```

## 结论

* 刚创建未添加任何元素时，容量是 0。
* 当添加第一个元素时，容量为默认容量 10。
* 需要扩容时，容量会变为原先的 1.5 倍。