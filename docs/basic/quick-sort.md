本页面将简要介绍快速排序。

## 简介

快速排序（英语：Quicksort），又称分区交换排序（partition-exchange sort），简称快排，是一种被广泛运用的排序算法。

## 工作原理

快速排序的工作原理是通过 [分治](./divide-and-conquer.md) 的方式来将一个数组排序。

快速排序分为三个过程：

1. 将数列划分为两部分（要求保证相对大小关系）；
2. 递归到两个子序列中分别进行快速排序；
3. 不用合并，因为此时数列已经完全有序。

和归并排序不同，第一步并不是直接分成前后两个序列，而是在分的过程中要保证相对大小关系。具体来说，第一步要是要把数列分成两个部分，然后保证前一个子数列中的数都小于后一个子数列中的数。为了保证平均时间复杂度，一般是随机选择一个数 $m$ 来当做两个子数列的分界。

之后，维护一前一后两个指针 $p$ 和 $q$ ，依次考虑当前的数是否放在了应该放的位置（前还是后）。如果当前的数没放对，比如说如果后面的指针 $q$ 遇到了一个比 $m$ 小的数，那么可以交换 $p$ 和 $q$ 位置上的数，再把 $p$ 向后移一位。当前的数的位置全放对后，再移动指针继续处理，直到两个指针相遇。

其实，快速排序没有指定应如何具体实现第一步，不论是选择 $m$ 的过程还是划分的过程，都有不止一种实现方法。

第三步中的序列已经分别有序且第一个序列中的数都小于第二个数，所以直接拼接起来就好了。

### 线性找第 k 大的数

找第 $k$ 大的数（K-th order statistic），最简单的方法是先排序，然后直接找到第 $k$ 大的位置的元素。这样做的时间复杂度是 $O(n\log n)$ ，对于这个问题来说很不划算。事实上存在时间复杂度 $O(n)$ 的解法。

考虑快速排序的划分过程，在快速排序的「划分」结束后，数列 $A_{p} \cdots A_{r}$ 被分成了 $A_{p} \cdots A_{q}$ 和 $A_{q+1} \cdots A_{r}$ ，此时可以按照左边元素的个数（ $q - p + 1$ ）和 $k$ 的大小关系来判断是只在左边还是只在右边递归地求解。

可以证明，在期望意义下，程序的时间复杂度为 $O(n)$ 。

## 性质

### 稳定性

快速排序是一种不稳定的排序方法。

### 时间复杂度

快速排序的最佳时间复杂度和平均时间复杂度为 $O(n\log n)$ ，最坏时间复杂度为 $O(n^2)$ 。

实践中几乎不可能达到最坏情况，且因为快速排序的内存访问遵循局部性原理，多数情况下快速排序的表现大幅优于堆排序等其他复杂度为 $O(n \log n)$ 的排序算法。[^ref1]

在选择 $m$ 的过程中，使用 [Median of Medians](https://en.wikipedia.org/wiki/Median_of_medians) 算法就可以保证最坏时间复杂度为 $O(n\log n)$ 。但是其过于复杂，实践中一般不使用。

## 代码实现

### C++[^ref2]

```cpp
struct Range {
  int start, end;
  Range(int s = 0, int e = 0) { start = s, end = e; }
};

template <typename T>
void quick_sort(T arr[], const int len) {
  if (len <= 0) return;  // 避免 len 等于负值时宣告堆叠阵列宕机
  // r[]模拟堆叠，p为数量,r[p++]为push,r[--p]为pop且取得元素
  Range r[len];
  int p = 0;
  r[p++] = Range(0, len - 1);
  while (p) {
    Range range = r[--p];
    if (range.start >= range.end) continue;
    T mid = arr[range.end];
    int left = range.start, right = range.end - 1;
    while (left < right) {
      while (arr[left] < mid && left < right) left++;
      while (arr[right] >= mid && left < right) right--;
      std::swap(arr[left], arr[right]);
    }
    if (arr[left] >= arr[range.end])
      std::swap(arr[left], arr[range.end]);
    else
      left++;
    r[p++] = Range(range.start, left - 1);
    r[p++] = Range(left + 1, range.end);
  }
}
```

## 内省排序[^ref3]

内省排序（introspective sort）是快速排序和 [堆排序](heap-sort.md) 的结合，由 David Musser 于 1997 年发明。内省排序其实是对快速排序的一种优化，保证了最差时间复杂度为 $O(n\log n)$ 。

内省排序的思路其实很简单，就是限制快速排序最大递归深度为 $\lfloor \log_2n \rfloor$ ，超过限制时就转换为堆排序。这样既保留了快速排序内存访问的局部性，又可以防止快速排序在某些情况下性能退化为 $O(n^2)$ 。

2000 年 6 月，SGI C++ STL 的 stl_algo.h 中 sort(）函数的实现采用了内省排序算法。

## 三路快速排序

三路快速排序（英语：3-way Radix Quicksort）是快速排序和 [基数排序](radix-sort.md) 的混合。它的算法思想基于 [荷兰国旗问题](https://en.wikipedia.org/wiki/Dutch_national_flag_problem) 的解法。

与原始的快速排序不同，三路快速排序在随机选取分界点 $m$ 后，将待排数列划分为三个部分：小于 $m$ 、等于 $m$ 以及大于 $m$ 。

三路快速排序在处理含有多个重复值的数组时，效率远高于原始快速排序。

三路快速排序的最佳时间复杂度为 $O(n)$ 。

## 参考资料与注释

[^ref1]:  [C++ 性能榨汁机之局部性原理 - I'm Root lee !](http://irootlee.com/juicer_locality/) 

[^ref2]:  [算法实现/排序/快速排序 - 维基教科书，自由的教学读本](https://zh.wikibooks.org/wiki/%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/%E6%8E%92%E5%BA%8F/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F) 

[^ref3]:  [introsort](https://en.wikipedia.org/wiki/Introsort) 
