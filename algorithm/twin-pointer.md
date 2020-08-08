# 双指针
LeetCode上可以用双指针来解答的算法题。

### 1.盛最多水的容器
[11. 盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)。

```js
const maxArea = function (height) {
  let start = 0, end = height.length - 1
  let area = 0
  while (start < end) {
    const currentArea = (end - start) * Math.min(height[start], height[end])
    area = Math.max(area, currentArea)
    if (height[start] < height[end]) {
      start++
    } else {
      end--
    }
  }
  return area
}
```

### 2.三数之和
[15. 三数之和](https://leetcode-cn.com/problems/3sum/)。

```js
const threeSum = function (nums) {
  if (nums.length < 2) return []

  // 首先给数组排序
  nums.sort((a, b) => a - b)

  // 优化1，如果数组全为正或者全为负，直接返回
  if (nums[0] > 0 || nums[nums.length - 1] < 0) return []

  const res = []

  for (let i = 0; i < nums.length; i++) {
    // 去重
    if (i > 1 && nums[i - 1] === nums[i]) continue

    // 优化二，如果接下来的数据全为正，直接结束循环
    if (nums[i] > 0) break

    const target = 0 - nums[i]
    let start = i + 1, end = nums.length - 1
    while (start < end) {
      const current = nums[start] + nums[end]
      if (current < target) {
        start++
      } else if (current > target) {
        end--
      } else {
        res.push([nums[i], nums[start], nums[end]])

        // 去重
        while (start < end && nums[start + 1] === nums[start]) start++
        while (start < end && nums[end - 1] === nums[end]) end--
        // 移动指针
        start++
        end--
      }
    }
  }

  return res
}
```

### 3.删除排序数组中的重复项
[26. 删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)。

```js
const removeDuplicates = function (nums) {
  let slow = 0
  for (let fast = 0; fast < nums.length - 1; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++
      nums[slow] = nums[fast]
    }
  }
  return slow + 1
}
```

### 4.环形链表
[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)。

```js
const hasCycle = function (head) {
  const p = slow = fast = new ListNode()
  p.next = head
  while (fast && fast.next) {
    slow = slow.next
    fast = fast.next.next
    if (slow === fast) {
      return true
    }
  }
  return false
}
```

