---
title: LeetCode.209 | 长度最小的子数组
date: 2022-08-10 00:23:49
categories: 
    - LeetCode
tags: 
    - 前缀和
    - 二分法
---

## [题目描述](https://leetcode.cn/problems/minimum-size-subarray-sum/)
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220810002446.png)

## 题目翻译
在一串数组中，找到连续的、和大于等于target值的子数组，并找到其中最短的那个，返回它的长度

## 心路历程
### 直观思路
对于这类问题，一个朴素的思路就是采用双层循环的方式暴力破解，但是暴力破解不满足题目对时间复杂度的要求

### 如何不使用双层循环嵌套完成数组的遍历
观察数组的特点，由于每个元素都>=1, 那么其依次加和得到的`前缀和数组`一定是一个单调递增数组，我们可以首先通过一次循环完成其前缀和数组的初始化。
例如nums=[1,2,3,4], 其前缀和数组S就是[0,1,3,6,10]. 即s[i] = s[i-1] + nums[i-1] 
![](https://raw.githubusercontent.com/SmartMalphite/PicBed/master/img-hexo/20220810015239.png)

```go
func minSubArrayLen(target int, nums []int) int {
    s := make([]int, len(nums)+1)
    s[0] = 0
    for i:=1;i<=len(nums);i++{
        s[i] = s[i-1] + nums[i-1]
    }
```

### 得到前缀和数组S之后怎么找到符合要求的子数组
由于前缀和数组S中每个值代表的都是前i位数据的和，那么当出现S[i] >= target之后， 我们需要找到一个元素，使其满足 `s[x] <= s[i] - target`

例如nums=[2,3,1,2,4,3], 其前缀和数组S就是[0,2,5,6,8,12,15]. Target=7
当遇到`S[4]=8`大于等于Target时，找到小于等于`8-7=1`的最后一个值，也就是`S[0]`.此时的子数组长度为`4-0=4`

## 整体代码实现
```go
func minSubArrayLen(target int, nums []int) int {
    result := math.MaxInt32
    s := make([]int, len(nums)+1)
    s[0] = 0
    for i:=1;i<=len(nums);i++{
        s[i] = s[i-1] + nums[i-1]
    }
    for k, v := range s{
        if v < target{
            continue
        }
        d := v - target
        l, r := 0, k
        for l <= r {
            mid := (l + r) / 2
            if s[mid] > d{
                r = mid - 1
            }else{
                l = mid + 1
            }
        }
        result = min(result, k-r)
    }
    if result == math.MaxInt32{
        return 0
    }
    return result
}

func min(a,b int) int{
    if a < b{
        return a
    }
    return b
}
```