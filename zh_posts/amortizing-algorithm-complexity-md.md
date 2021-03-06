---
title: 关于算法的均摊复杂度优化
date: 2016-11-16 22:34:34
tags:
- amortized complexity
- algorithm design
---

## 扔瓶子问题
瓶子问题是这样一个问题，假设我们有 100 层楼和 2 个瓶子，请你用最小尝试次数，把瓶子扔下去会碎的最低层数找。关于这个问题，如果我们拥有无限个瓶子，或至少 $\lceil \log_2(100) \rceil$ 个瓶子的时候，我想绝大多数人都可以轻易得出解法。但现在的问题是，在用瓶子做判断的时候，其中一个方向是不可逆的。假设我们用二分的方式划分区间，那么如果我们在第 50 层判定的时候瓶子碎了，我们接下来就只能从第 1 层线性往上尝试了。这样显然违背了二分降低平均搜索次数的初衷。

<!-- more -->

因为我们是从降低平均搜索次数的目标出发的，这个时候可能已经有人能想到最优的解法了。不过让我们先考虑一个还不错的中间解法。

1. 假如我们只有 1 个瓶子，那么我们显然只能在区间里做线性查找；
2. 此时我们有两个瓶子，那其实我们可以把区间分成若干个子区间，用两个瓶子划分两个层次各自做线性查找。其中一个瓶子以 1 为步长，而第二个瓶子以子区间的长度作为步长。为了让两个瓶子划分的搜索各自均匀，我们应该尽量让它们的搜索空间相等。因此我们可得，划分的区间长度最好应为 $\sqrt[2]{100} = 10$。
3. 同理可得，当我们有 k 个瓶子的时候，我们可以用瓶子划分 k 个层次，并让每个层次搜索空间尽量相等。那么我们可知，最小的那个区间长度应为 $\sqrt[k]{100}$。并且，当 $k  \ge \lceil \log_2{100} \rceil$ 的时候，因为每个瓶子的搜索空间最小为 1， 此时我们的解法就退化为二分搜索了。

看上去上面的解法已经是最优了，那它和最优解法上究竟浪费了哪里的效率呢？这个问题实际上和现实里二分搜索所浪费的地方是一样的。对这个瓶子的问题，我们可以考虑最好情况和最坏情况。

最好情况下，在第一个层次瓶子按 10， 20 这样的顺序扔，如果它在 10 层就碎了，在第二个层次，瓶子从 1， 2 这样的顺序扔，它也可能在 1 层就碎了，那么此时，它的搜索次数为 2。

那么最坏情况呢？在第一个层次，瓶子会从 10 一路遍历到 100 然后在 100 层碎了。然后在第二个层次从 91 一路遍历到 99 层才碎。那么它的搜索次数就是 18 了。

显然这样的分布还没达到足够均匀。而不均匀的原因在一开篇就讲了，就是因为搜索方向上的不对称。假使我们用用瓶子在某层做判定，当瓶子碎了，我们可以直接进入下一个层次，而若瓶子没碎，就相当于我们已经浪费了一次机会。那么我们在划分左右两个区间的时候，为了让两边最终搜索次数足够均匀，就应该让右边的搜索次数期望，比左边区间的期望少 1 才对。

现在假设我们第一次选择的分界点在 $x$，那么对第二个区间，它的长度就应该是 $x-1$ 这样依次往下，直到所有区间的总长超过 $100$ 。即有 $\sum_{i=c}^{x} \ge 100$，得 $x = 14$ 。即第一个瓶子的选点分别是 $14, 14 + 13 = 27, 14 + 13 + 12 = 39, \ldots$ 这样的话，平均的搜索次数就接近最优了。

## 二分 V.S. 斐波那契

瓶子的问题说完了，现在来看看实际中的二分搜索。为什么说二分也存在类似的搜索方向上成本不对称呢？那是因为在我们目前的计算机体系里，不存在一个一次输入的情况下返回“大于”，“等于”，“小于” 3 种结果的逻辑会运行的和一次输入返回“大于”，“不大于” 2 种结果一样快。那么对于一个需要判定相等时短路返回，没找到时返回 `std::end(container)` 的二分搜索来说，一个典型的实现可以是这样的：

```c++
template <typename RandomIt>
RandomIt bsearch(
  typename std::iterator_traits<RandomIt>::value_type target,
  RandomIt lo, 
  RandomIt hi)
{
  auto not_found = hi;
  auto mid = lo + (hi - lo) / 2;
  while (lo < hi) {
    auto val = *mid;
    if (target < val) hi = mid;
    else if (target > val) lo = mid + 1;
    else return mid;
  }
  return not_found;
}
```

考虑单次判定的时候，如果目标值在左边，那么我们一次小于判定后可以直接转向左边区间；而如果目标值在右边，你还得多做一次大于判定。也就是说，当你转向右边区间的时候，前面的那个小于判定就相当于我们浪费的一个瓶子了。因为在这个实现的每次循环里需要做两次判定，不论如何设计执行的顺序，也一定会形成一边的搜索成本比另外一边要高的情况。

那么我们的优化方向也和瓶子问题一样了，不过关于如何在数学上求取平均下最优还是有点复杂的（其实是懒的去算了），这里就不展开了。但即使不算还是能从它的递归结构里能看出它和斐波那契数列的关系的。最主要的还是体会其中的思想，即用分配的不对称来应对成本的不对称，让成本低的区间相对更大，让成本高的区间更小，达到更低的均摊复杂度。

