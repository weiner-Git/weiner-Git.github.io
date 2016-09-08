#常见排序算法-快速排序

**1、快速排序**
（英语：Quicksort），又称划分交换排序（partition-exchange sort），一种排序算法，最早由东尼·霍尔提出。在平均状况下，排序n个项目要Ο(n log n)次比较。在最坏状况下则需要Ο(n2)次比较，但这种状况并不常见。事实上，快速排序通常明显比其他Ο(n log n)算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地被实现出来。

快速排序的思路很简单，按下面三步：<br/>
	1、从数列中挑出一个元素，称为"基准"（pivot，可以随意选择，以中间值好理解）<br/>
	2、重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区操作。
	3、对基准的前后两边分区不断重复第一步和第二部，递归地把小于基准值元素的子数列和大于基准值元素的子数列排序。
	
举个列子
对数集 {32，53，52，49，20，57，42}进行快排<br/>
第一步：选49为基准。<br/>
{32，53，52，`49`，20，57，42}<br/>
第二步：顺序的与基准比较大小，小于的放入左边集合，大于的放入右边集合<br/>
{32，20，42} {`49`} {53，52，57}<br/>
第三部：对左右子集重复一、二动作，直到剩下一个元素。
{32，`20`，42} {`49`} {53，`52`，57}<br/>
{`20`}{32，42} {`49`} {`52`}{53，57}<br/>
{`20`}{`32`}{42} {`49`} {`52`}{`53`}{57}<br/>

最后快拍的结果为：
{20，32，42，49，52，53，57}

下面看一下代码(代码中基准是数组第一个元素)：

```
//
//  main.c
//  quicksort
//
//  Created by weiner
//  Copyright © 2015年 weiner. All rights reserved.
//

#include <stdio.h>

void quicksort(int a[] ,int left, int right){
    if (left >= right) {
        return;
    }
    int i = left;
    int j = right;
    int key = a[i];
    
    while (i < j) {
        int jy = a[j];
        if (key <= jy) {
            j--;
        }
        a[i] = a[j];
        
        int iy = a[i];
        if (key > iy) {
            i++;
        }
        a[j] = a[i];
    }
    
    a[i] = key;
    quicksort(a, left, i-1);
    quicksort(a, i+1, right);
}

int main(int argc, const char * argv[]) {
    // insert code here...
    int a[7] = {32,53,52,49,20,57,42};
    quicksort(a,0,6);
    for (int i=0; i<7; i++) {
        printf("%d,",a[i]);
    }
}
```

这里要注意while里面的两个if判断，判断条件要正好匹配完成，不要都包含=号，题主这里都用了‘=’，出现了42，49位置错误，调试的时候发现i++和j++都执行了，犯了个错误。