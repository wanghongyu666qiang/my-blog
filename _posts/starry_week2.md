# 🧠 starry的算法修炼笔记 | Week 2

**作者：** starry
**时间：** 2026年4月11日  
**标签：** LeetCode刷题 · 数据结构 · C++ · Python
（注：跟随代码随想录敲的代码，有需要的可以看代码随想录，本文的图片也来自代码随想录）
---

> 📌 **本週核心技能**
> - 虚拟头节点 dummy head
> - 快慢指针（双指针进阶）
> - 哈希表：unordered_set / unordered_map

---

## 一、链表操作：虚拟头节点是神器

### 1.1 反转列表

核心思路：用两个指针 `pri` 和 `cur` 边走边反转。

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* pri = NULL, *cur = head;
    while (cur != NULL) {
        ListNode* curx = cur->next;
        cur->next = pri;
        pri = cur;
        cur = curx;
    }
    return pri;
}
```

> 💡 **技巧：** 把可能越界的判断放在 `while` 循环条件里，比在循环体内多加两个 `if` 更简洁。

---

### 1.2 两两交换数字

```cpp
ListNode* swapPairs(ListNode* head) {
    ListNode* vhead = new ListNode;
    vhead->next = head;
    ListNode* cur = vhead;
    while (cur->next != NULL && cur->next->next != NULL) {
        ListNode* temp = cur->next;
        ListNode* temp1 = cur->next->next;
        cur->next = temp1;
        temp->next = temp1->next;
        temp1->next = temp;
        cur = cur->next->next;
    }
    return vhead->next;
}
```

> ⚠️ **重要：** 一定要从**虚拟节点**开始跑！否则头节点改变后很难处理。

---

### 1.3 删除链表的倒数第 N 个元素

用快慢指针，让 fast 先走 n+1 步，这样 slow 恰好停在待删节点的前一个位置。

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode* vhead = new ListNode;
    vhead->next = head;
    ListNode* fast = vhead, *slow = vhead;
    for (int i = 0; i <= n; i++) fast = fast->next;
    while (fast != NULL) {
        slow = slow->next;
        fast = fast->next;
    }
    slow->next = slow->next->next;
    return vhead->next;
}
```

> 💡 fast 移动 n+1 次，slow 才指向真正要删除节点的前一个指针。（这里leecode中的n<=元素，所以无需判断，要加强其健壮性的话，可以'''if(i!=n&&fast=NULL)return false;'''

---

## 二、链表进阶：相交与环形

### 2.1 查找相交节点

先分别遍历两链表求长度，让长链表先走差值步，再同步前进找交点。

```cpp
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    int len_a = 0, len_b = 0;
    ListNode *a = headA, *b = headB;
    while (a) { len_a++; a = a->next; }
    while (b) { len_b++; b = b->next; }

    // 保证 a 指向长的那条
    if (len_b > len_a) { swap(len_a, len_b); swap(a, b); }
    a = headA; b = headB;

    for (int i = 0; i < len_a - len_b; i++) a = a->next;
    while (a && a != b) { a = a->next; b = b->next; }
    return a;
}
```

---

### 2.2 找到环形链表入口 ⭐

这是本周最难的一题！分两步走：

1. **快慢指针判断是否有环**
2. **有环时：从头节点和相遇点各派一个指针，相遇点即为入口**

推导公式：`x = n(y + z) - y`，即头节点到入口 = 相遇点走 n-1 圈后再走 z。

```cpp
ListNode* detectCycle(ListNode* head) {
    ListNode* fast = head, *slow = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (fast == slow) {
            ListNode* index1 = head, *index2 = fast;
            while (index1 != index2) {
                index1 = index1->next;
                index2 = index2->next;
            }
            return index1;
        }
    }
    return NULL;
}
```

> 📸 图解如下——

![环形链表入口推导图](Pasted%20image%2020260411162920.png)

---

## 三、哈希表：高效查找的利器

**什么时候用哈希法？** 当需要查询某个元素是否出现过，或者是否在集合里时，第一时间想到哈希法。

> 📌 **常用容器**：`vector`、`list`、`deque`、`set`、`unordered_set`、`map`、`unordered_map`、`array`、`string`

---

### 3.1 有效的字母异位词

```cpp
bool isAnagram(string s, string t) {
    int a[26] = {0};
    for (char c : s) a[c - 'a']++;
    for (char c : t) a[c - 'a']--;
    for (int i = 0; i < 26; i++)
        if (a[i] != 0) return false;
    return true;
}
```

---

### 3.2 求交集（利用哈希表快速查找，避免双层循环！）

```cpp
vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
    unordered_set<int> result, num1_set(nums1.begin(), nums1.end());
    for (int num : nums2)
        if (num1_set.find(num) != num1_set.end())
            result.insert(num);
    return vector<int>(result.begin(), result.end());
}
```

---

### 3.3 快乐数（用哈希表检测循环）

```cpp
int happy(int n) {
    int sum = 0;
    while (n > 0) {
        sum += (n % 10) * (n % 10);
        n /= 10;
    }
    return sum;
}

bool isHappy(int n) {
    unordered_set<int> dp;
    while (true) {
        int h = happy(n);
        if (h == 1) return true;
        if (dp.find(h) != dp.end()) return false;
        dp.insert(h);
        n = h;
    }
}
```

---

### 3.4 两数之和（哈希表yyds！）

暴力解法是两层 for 循环 O(n²)，用哈希表只需一次遍历：

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> mp;
    for (int i = 0; i < nums.size(); i++) {
        if (mp.find(target - nums[i]) != mp.end())
            return {mp[target - nums[i]], i};
        mp[nums[i]] = i;
    }
    return {};
}
```

> 💡 将遍历过的数字存入表，查找 `target - 当前数字`，同时保证了相同元素只输出一次，确实妙不可言。

---

### 3.5 四数相加 ⭐

本质是四数之和 → 两数之和，用两重循环把所有 `a+b` 存入哈希表：（把n****4的复杂度变为n****2的复杂度）

```cpp
// C++版
int fourSumCount(vector<int>& nums1, vector<int>& nums2,
                 vector<int>& nums3, vector<int>& nums4) {
    unordered_map<int, int> mp;
    for (int a : nums1)
        for (int b : nums2) mp[a + b]++;

    int sum = 0;
    for (int a : nums3)
        for (int b : nums4)
            if (mp.find(-a - b) != mp.end())
                sum += mp[-a - b];
    return sum;
}
```

```python
# Python版
def fourSumCount(nums1, nums2, nums3, nums4):
    d = {}
    for a in nums1:
        for b in nums2:
            d[a+b] = d.get(a+b, 0) + 1
    res = 0
    for a in nums3:
        for b in nums4:
            if -(a+b) in d:
                res += d[-(a+b)]
    return res
```

---

## 四、本周总结

> 🔑 **核心结论：数组中同一个元素不能使用两遍，就用哈希表自动去重！**
>
> 多用 `unordered_set` / `unordered_map`，哈希表本身就是去重的好帮手。

| 题型 | 技巧 |
|------|------|
| 链表增删改 | **虚拟头节点**，永远从 dummy 开始 |
| 链表找特定节点 | **快慢指针**（环形入口、倒数第N个） |
| 判断/查找重复 | **哈希表**（unordered_set/map） |
| 多数之和 | 哈希化简：N数之和 → 两数之和 |

---

![本周算法全家福](Pasted%20image%2020260411193452.png)

---

*📅 记录于 2026年4月 | 算法修炼进行中...*
