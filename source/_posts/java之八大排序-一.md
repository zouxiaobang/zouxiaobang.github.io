---
title: java之八大排序
date: 2016-11-10 12:15:07
tags: [java]
categories: [Java]
---

## 八大排序

排序，在java中一直是其算法的典型。而在JDK中，也java也已经封装了一个sort排序方式--Arrays.sort，其使用了经过调优的快速排序方式来进行数组的升序排序。

![八大排序关系图](/img/sort/8sort.jpg)

接下来，将通过几篇博客的容量来分享我对排序的理解。

### 插入排序
插入排序，就是通过每次的比较，将该数插入到一段有序的序列中。

#### 1、直接插入排序
直接插入排序是一种比较简单，比较直观的排序方法。
时间复杂度：O(n^2)
原理：将一个数通过与前面的有序序列的每一个数进行对比并插入。
做法：从第二个数开始循环，比较该数前面的所有的数，如果前数比该数大，则将前数后移一位，直到前数比该数小，则停止，并将该数填入到空位中。
![InsertSort](/img/sort/insertSort.jpg)

``` bash
public static void insertSort(int nums){
	//记录要进行比较的数
	int compare = 0;
	for(int i = 1;i < nums.length;i ++){
		compare = nums[i];
		//前数：在compare的前面所有的数已经形成一个有序的数组，用j来记录该有序数组的最后一位数
		int j = i - 1;
		for(;j >= 0 && compare < nums[j];j --){
			//如果compare比前面的数小，则将前面的数往后移一位
			nums[j+1] = nums[j];
		}
		//如果compare不比前面的数小，则将该数填入到空位中
		nums[j] = compare;
	}
}
```
由以上代码可以看出，如果有相同的元素存在，是不会改变前后的顺序的，而从后面无序序列的顺序依然和还没排序时的顺序是一样的，
所以，插入排序时一种稳定的排序。

#### 2、希尔排序
Shell排序是一种比较复杂的算法，同时通过多个循环来完成这个算法的。
时间复杂度：O(n) < O < O(n^2) : 大概是O(n^1.3)
原理：通过循环，每次都与其相隔d距离(步长)的数进行比较并交换，循环比较之后缩短步长d，再继续循环
做法：步长d初始为该数组的长度的一半，每次缩短都是去上次的步长的一半(Math.ceil取顶)，通过比较没一步的同个位置的数的大小并决定是否交换
![ShellSort](/img/sort/shellSort.jpg)
刚开始步长比较大，则插入元素少，速度就会比较快；随着步长逐渐变小，插入的频率增大，速度降低。
在这个多循环的算法中，真正的核心循环在于循环比较
![](/img/sort/shellSort2.jpg)

``` bash
public static void shellSort(int nums){
	//步长
	int d = nums.length;

	//不断循环
	while(true){
		//每次都取上次步长的一半来划分多少步
		d = Math.ceil(d/2);
		for(int i = 0;i < d;i ++){
			//待比较的步
			for(int j = i+d;j < nums.length;j += d){
				//前一步
				int preStep = j - d;
				int temp = nums[j];
				for(;preStep >= 0 && nums[j] < nums[preStep];preStep -= d){
					//交换
					nums[j] = nums[preStep];
				}
				nums[preStep+d] = temp;
			}			
		}
		if(d == 1){
			//步长为1时退出循环
			break;
		}
	}
}
```
由上面的算法可以看出来它具有很多的循环体，在复杂的外表下，其实核心还是比较交换，但是，和直接插入排序所不同的是，它在循环比较的过程中，相同元素的相对次序是无规律的，并不能保持不变。例如i=j，而且i在j的前面，但是，通过步长的多次改变之后，i和j有可能会是相邻，但是他们的相对前后顺序就不一定能保证是与未排序之前是一样的。所以，希尔排序是一种不稳定的排序方式。


[八大排序之选择排序](https://zouxiaobang.github.io/2016/11/11/java之八大排序-二/)






