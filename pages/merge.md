#归并排序
**归并排序**（维基百科）（英语：Merge sort，或mergesort），是创建在归并操作上的一种有效的排序算法，效率为O(n log n)。1945年由约翰·冯·诺伊曼首次提出。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。


归并排序**介绍**：

1. 稳定性:归并排序是一种稳定的排序。
2. 存储结构要求:可用顺序存储结构。也易于在链表上实现。
3. 时间复杂度:对长度为 n 的文件，需进行 趟二路归并，每趟归并的时间为 O(n)，故其时间复杂度无论是在最好情况下还是在最坏情况下均是 O(nlgn)。
4. 空间复杂度:需要一个辅助向量来暂存两有序子文件归并的结果，故其辅助空间复杂度为 O(n)

**归并介绍**：

队并处理就是讲两个有序数组的元素合并到一个数组：<br />
如array a:[2, 6]和array b:[1, 3]<br />

根据数组下标对比。当数组下标元素加入结果数组，改数组的下标+1。先对比a[0]和b[0],a[0]>b[0]，所以先将b[0]:1加入结果数组，a[0]再与b[1]:3对比，小于b[1]，将2加入结果数组，再将b[1]与a[1]对比，小于并将b[1]加入结果数组，b数组已经到尾部，所以讲a中剩余元素加入结果数组。

归并排序有两种方式：

**迭代法**（自底向上方法）:
迭代法是先将长度为n（n>=2）的数组分为两个元素的排序的子数组，然后子数组在两两组合排序为子数组，最后数组元素长度为n停止迭代。<br />
需要注意的是当n为奇数时候，第一轮并不需要加入归并。

array [6, 5, 3, 1, 8, 7, 2, 4]

第一步：拆分数组为单个元素<br />
[6] [5] [3] [1] [8] [7] [2] [4]<br />
第二步：第一轮迭代、归并<br />
[5, 6] [1, 3] [7, 8] [2, 4]<br />
第三步：第二轮迭代、归并<br />[1, 3, 5, 6] [2, 4, 7, 8]<br />
第四步：最后归并两个子数组<br />[1, 2, 3, 4, 5, 6, 7, 8]<br />

伪代码：

```c
int a[8] = {6, 5, 3, 1, 8, 7, 2, 4}
void merge_sort(a[], int len){
	int b[8];	//	临时数组
	for(i = 1, i < len, i += i){	//	每次迭代子元素的个数为上一次两个数组归并，随意i += i
		for(j = 0, j < len, j += i+i ){		//	用j至j+i和j+i至j+i+i定义两个子数组的开始和结尾的元素位置
			int k = j;	//	记录初始位置
			int start1 = k, end1 = j+i;		//	数组1开始和结尾的位置
			int start2 = j+1, end2 = j+i+i;		//	数组2的开始和结尾位置
			
			//	开始归并操作 对加入临时数组的下标+1
			while (start1 < end1 && start2 < end2)
				b[k] = a[start1] < a[start2] ? a[start1] && start1++: a[start2] && start2++;
				k++;
			while (start1 < end1)
				b[k] = a[start1];
				start1++; k++;
			while (start2 < end2)
				b[k] = a[start2];
				start2++; k++;
		}
		//	讲归并后的数组更新
		a = b;	
	}
}
```

**递归法**（自顶向下的方法）:
递归法就是先将当前数组一分为二，对子区间归并处理，当子区间长度为1时候停止递归。递归处理的顺序和迭代相同，都是先归并长度为1的数组。

```c
//
//  main.c
//  merge
//
//  Created by weiner.
//  Copyright © 2015年 weiner. All rights reserved.
//

#include <stdio.h>
void merge_sort(int a[], int reg[], int start, int end) {
    if (start >= end)   //  当相等时候元素长度为1
        return;
    int len = end - start, mid = (len >> 1) + start;
    int start1 = start, end1 = mid;
    int start2 = mid + 1, end2 = end;   //  这里定义两个子数组的开始和结尾位置
    merge_sort(a, reg, start1, end1); //  这里类似二叉树的遍历 对mid左边和右边分别递归
    merge_sort(a, reg, start2, end2);
    int k = start;
    while (start1 <= end1 && start2 <= end2)    //  这里的条件不同于迭代是因为元素长度为1时候是以相等判断
        reg[k++] = a[start1] < a[start2] ? a[start1++] : a[start2++];
    while (start1 <= end1)
        reg[k++] = a[start1++];
    while (start2 <= end2)
        reg[k++] = a[start2++];
    for (k = start; k <= end; k++)
        a[k] = reg[k];    //    更新数组
}

int main() {
    // insert code here...
    int a[8] = {6, 5, 3, 1, 8, 7, 2, 4};
    int reg[8]; //  临时数组
    merge_sort(a, reg, 0, 7);
    for (int i=0; i<8; i++) {
        printf("%d,",a[i]);
    }
    return 0;
}

```



