---
layout: post
title: Codility刷题存疑：MinAbsSum
date: 2016-12-27
category: notes
tags: [codility, c++]
---

### 解题思路
虽然是题目列在动态规划上面，但是还是想别的方法可不可以解。于是就有了下面的方案。
所有数字求绝对值，然后按从大到小排序。
设两个和sum1和sum2。
每次加一个值，若sum1>sum2，加到sum2上，否则加到sum1上。

### 代码
{% highlight C++ %}
// you can use includes, for example:

#include <algorithm>

// you can write to stdout for debugging purposes, e.g.
// cout << "this is a debug message" << endl;

int solution(vector<int> &A) {
// write your code in C++14 (g++ 6.2.0)

    int n = A.size();

    if (n == 0)
    return 0;
    if (n == 1)
    return abs(A[0]);

    for (int i = 0; i < n; i++){
        A[i] = abs(A[i]);
    }

    sort(A.begin(), A.end());
    reverse(A.begin(), A.end());

    int sum1 = A[0];
    int sum2 = A[1];

    for (int i = 2; i < n; i++){
        if (sum1 < sum2)
            sum1 += abs(A[i]);
        else
            sum2 += abs(A[i]);
    }

    // Start: 这一段代码是为了通过simple3和range2..20的测试用例 
    int tmp = abs(sum1 - sum2);
    sum1 = A[0] + A[1];
    sum2 = 0;
    for (int i = 2; i < n; i++){
        if (sum1 < sum2)
            sum1 += abs(A[i]);
        else
            sum2 += abs(A[i]);
    }
    // End

    return abs(sum1 - sum2) < tmp ? abs(sum1 - sum2) : tmp;
}
{% endhighlight %}

### 测试结果
首先，复杂度为`O(N * max(abs(A))**2)`，符合题目要求。
通过了所有的正确性检验，针对**[3,3,3,4,5]**这样类似的测试用例作出上文中`// Start`和`// End`之间的改动。
但是性能检验时，**第一个测试用例（medium1：medium random）**出错了，其余都对。这个测试用例期望是0但是我的代码给了4。
由于是性能检验，所以也不会给出错在什么样的测试用例上，让人有点匪夷所思。感觉问题应该是出在和**[3,3,3,4,5]**这样的测试用例类似的问题上。

### 引用
[Test results - Codility](https://codility.com/demo/results/training45PKZU-T7B/)


