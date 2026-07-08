# 力扣记录

这里记录刷题过程中的题型、核心思路和代码模板。每道题尽量写清楚“为什么这样做”，不要只留下代码。

## 哈希表

### 1. 两数之和

思路：遍历数组，用哈希表保存已经见过的数和下标。对当前数 `x`，检查 `target - x` 是否出现过。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> index;

        for (int i = 0; i < nums.size(); ++i) {
            int need = target - nums[i];
            if (index.find(need) != index.end()) {
                return {index[need], i};
            }
            index[nums[i]] = i;
        }

        return {};
    }
};
```

### 3. 无重复字符的最长子串

思路：滑动窗口。右指针扩张窗口，遇到重复字符时移动左指针，保证窗口内没有重复字符。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char, int> last;
        int left = 0;
        int ans = 0;

        for (int right = 0; right < s.size(); ++right) {
            char c = s[right];
            if (last.count(c) && last[c] >= left) {
                left = last[c] + 1;
            }
            last[c] = right;
            ans = max(ans, right - left + 1);
        }

        return ans;
    }
};
```

### 49. 字母异位词分组

思路：把每个字符串排序后作为 key，异位词排序后的 key 相同。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> groups;

        for (const string& s : strs) {
            string key = s;
            sort(key.begin(), key.end());
            groups[key].push_back(s);
        }

        vector<vector<string>> ans;
        for (auto& item : groups) {
            ans.push_back(item.second);
        }
        return ans;
    }
};
```

### 128. 最长连续序列

思路：把所有数放入哈希集合。只有当 `num - 1` 不存在时，才把 `num` 当作连续序列的起点向后扩展。

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> seen(nums.begin(), nums.end());
        int ans = 0;

        for (int num : seen) {
            if (seen.find(num - 1) != seen.end()) {
                continue;
            }

            int cur = num;
            int len = 1;
            while (seen.find(cur + 1) != seen.end()) {
                ++cur;
                ++len;
            }
            ans = max(ans, len);
        }

        return ans;
    }
};
```

## 双指针与滑动窗口

### 283. 移动零

思路：用 `write` 指针维护下一个非零元素应该写入的位置。

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int write = 0;

        for (int x : nums) {
            if (x != 0) {
                nums[write++] = x;
            }
        }

        while (write < nums.size()) {
            nums[write++] = 0;
        }
    }
};
```

### 11. 盛最多水的容器

思路：双指针从两端向中间移动。每次移动较短的那条边，因为面积受短边限制。

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;
        int right = height.size() - 1;
        int ans = 0;

        while (left < right) {
            int h = min(height[left], height[right]);
            ans = max(ans, h * (right - left));

            if (height[left] < height[right]) {
                ++left;
            } else {
                --right;
            }
        }

        return ans;
    }
};
```

### 15. 三数之和

思路：排序后固定第一个数，剩余两个数用双指针查找。注意去重。

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ans;
        sort(nums.begin(), nums.end());

        for (int i = 0; i + 2 < nums.size(); ++i) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }

            int left = i + 1;
            int right = nums.size() - 1;
            int target = -nums[i];

            while (left < right) {
                int sum = nums[left] + nums[right];
                if (sum == target) {
                    ans.push_back({nums[i], nums[left], nums[right]});
                    while (left < right && nums[left] == nums[left + 1]) ++left;
                    while (left < right && nums[right] == nums[right - 1]) --right;
                    ++left;
                    --right;
                } else if (sum < target) {
                    ++left;
                } else {
                    --right;
                }
            }
        }

        return ans;
    }
};
```

### 42. 接雨水

核心结论：每个位置能接的水量是

```text
min(左侧最高柱子, 右侧最高柱子) - 当前柱子高度
```

如果结果为负，说明这个位置不能接水。

#### 双指针写法

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        if (height.empty()) return 0;

        int left = 0;
        int right = height.size() - 1;
        int leftMax = 0;
        int rightMax = 0;
        int ans = 0;

        while (left < right) {
            leftMax = max(leftMax, height[left]);
            rightMax = max(rightMax, height[right]);

            if (height[left] < height[right]) {
                ans += leftMax - height[left];
                ++left;
            } else {
                ans += rightMax - height[right];
                --right;
            }
        }

        return ans;
    }
};
```

### 438. 找到字符串中所有字母异位词

思路：固定长度滑动窗口，维护窗口内每个字符的出现次数。

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> ans;
        if (s.size() < p.size()) return ans;

        vector<int> need(26, 0), window(26, 0);
        for (char c : p) need[c - 'a']++;

        int len = p.size();
        for (int i = 0; i < s.size(); ++i) {
            window[s[i] - 'a']++;
            if (i >= len) {
                window[s[i - len] - 'a']--;
            }
            if (window == need) {
                ans.push_back(i - len + 1);
            }
        }

        return ans;
    }
};
```

## 前缀和

### 560. 和为 K 的子数组

思路：记录前缀和出现次数。当前前缀和为 `cur` 时，如果之前出现过 `cur - k`，说明中间这段和为 `k`。

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int ans = 0;
        int cur = 0;
        unordered_map<int, int> count;
        count[0] = 1;

        for (int x : nums) {
            cur += x;
            if (count.find(cur - k) != count.end()) {
                ans += count[cur - k];
            }
            count[cur]++;
        }

        return ans;
    }
};
```

## 链表

### 160. 相交链表

!!! info "思路"
    1. 使用双指针，分别从两个链表头部开始遍历。
    2. 指针到达尾部后切换到另一条链表的头部。
    3. 如果两个链表相交，两个指针会在相交点相遇；如果不相交，会同时走到 `nullptr`。

### 234. 回文链表

!!! info "思路"
    1. 使用快慢指针找到链表中点。
    2. 反转链表后半部分。
    3. 比较前半部分和反转后的后半部分。
    4. 更简单但额外占空间的做法：复制一份值数组后双指针比较。

## 复盘模板

```text
题目：
题型：
核心技巧：
复杂度：
第一次错误：
是否需要二刷：
```
