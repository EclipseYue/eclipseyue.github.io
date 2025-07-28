记录力扣题
## 哈希表
1. 两数之和
```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> num_map;
        vector<int> result;

        for (int i = 0; i < nums.size(); ++i) {
            int complement = target - nums[i];
            if (num_map.find(complement) != num_map.end()) {
                result.push_back(num_map[complement]);
                result.push_back(i);
                return result;
            }
            num_map[nums[i]] = i;
        }

        return result; // 如果没有找到，返回空向量
    }
};
```
3. 无重复字符的最长子串
```C++

49. 字母异位词分组
排序做哈希，哈希表取出
```C++
class Solution {
    public:
        vector<vector<string>> groupAnagrams(vector<string>& strs){
            std::unordered_map<string,vector<string>> mp;
            for(const string& s: strs){
                string key = strs;
                sort(key.begin(),key.end());
                mp[key].push_back();
            }
            vector<vector<string>> res;
            for(auto &c: mp){
                res.push_back(c.second);
            }
        }
}
```
128. 最长连续序列
```C++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> num_set(nums.begin(), nums.end());
        int longest = 0;

        for (int num : num_set) {
            // 只有当当前数是一个序列的最小值时才进入查找
            if (num_set.find(num - 1) == num_set.end()) {
                int currentNum = num;
                int currentLength = 1;

                // 向后查找连续的数字
                while (num_set.find(currentNum + 1) != num_set.end()) {
                    currentNum++;
                    currentLength++;
                }

                longest = max(longest, currentLength);
            }
        }

        return longest;
    }
};
```

283. 移动零
```C++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int lastNonZeroFoundAt = 0; // 指向下一个非零元素的位置

        // 将非零元素移动到数组的前面
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != 0) {
                nums[lastNonZeroFoundAt] = nums[i];
                lastNonZeroFoundAt++;
            }
        }

        // 将剩余的位置填充为零
        for (int i = lastNonZeroFoundAt; i < nums.size(); i++) {
            nums[i] = 0;
        }
    }
};
```

11. 盛最多水的容器
```C++
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;
        int right = height.size() - 1;
        int maxVol = 0;

        while (left < right) {
            int h1 = height[left];
            int h2 = height[right];
            int vol = (right - left) * min(h1, h2);
            maxVol = max(maxVol, vol);

            // 移动较小的那一侧
            if (h1 < h2)
                left++;
            else
                right--;
        }

        return maxVol;
    }
};


```

15. 三数之和
```C++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res;
        sort(nums.begin(),nums.end());
        for(int i = 0;i < nums.size()-2;i++){
            if(i > 0 && nums[i] == nums[i-1]) continue;
            int left = i + 1;
            int right = nums.size()-1;
            int target = -nums[i];
            while(left<right){
                if(nums[left]+nums[right] == target){
                    res.push_back({nums[i], nums[left], nums[right]});
                    while(left < right && nums[left] == nums[left + 1]) left++;
                    while(left < right && nums[right] == nums[right - 1]) right--;
                    left++;
                    right--;
                }else if(nums[left]+nums[right] < target){
                    left++;
                }else if(nums[left]+nums[right] > target){
                    right--;
                }
            }
        }
        return res;
    }
};
```

注意限定条件

42. 接雨水
**大化小**
比较自然地，如果是坑，一个格子此处可以接水的量是：min(左边最高的柱子，右边最高的柱子) - 当前柱子的高度
如果是墙/平地，也算上自己，就是0

超时：
```C++
class Solution {
public:
    int trap(vector<int>& height) {
        int n = height.size();
        if (n == 0) return 0;
        int vol = 0;
        
        for (int i = 0; i < n; ++i) {
            int leftMax = 0, rightMax = 0;
            
            // 找到左边最高的墙
            for (int j = i; j >= 0; --j) {
                leftMax = max(leftMax, height[j]);
            }
            
            // 找到右边最高的墙
            for (int j = i; j < n; ++j) {
                rightMax = max(rightMax, height[j]);
            }
            
            // 当前位置的积水高度由左右两边的最低墙决定
            int currentWater = min(leftMax, rightMax) - height[i];
            if (currentWater > 0) {
                vol += currentWater;
            }
        }
        
        return vol;
    }
};
```
一言以蔽之，小不能再小（每次从小算起）
4ms
```C++
#include <vector>
using namespace std;

class Solution {
public:
    int trap(vector<int>& height) {
        if (height.empty()) return 0;

        int left = 0, right = height.size() - 1;
        int left_max = 0, right_max = 0;
        int ans = 0;

        while (left < right) {
            if (height[left] < height[right]) {
                // 当前位置是 left
                if (height[left] >= left_max)
                    left_max = height[left];  // 更新左最高墙
                else
                    ans += left_max - height[left];  // 可以接水
                left++;
            } else {
                // 当前位置是 right
                if (height[right] >= right_max)
                    right_max = height[right];  // 更新右最高墙
                else
                    ans += right_max - height[right];  // 可以接水
                right--;
            }
        }

        return ans;
    }
};
```
一样的逻辑
0ms
```C++
class Solution {
public:
    int trap(vector<int>& height) {
        int left = 0, right = height.size() - 1;
        int leftMax = height[0], rightMax = height[right];

        int ans = 0;
        while(left < right){
            leftMax = max(leftMax, height[left]);
            rightMax = max(rightMax, height[right]);

            if(height[left] < height[right]){
                ans += leftMax - height[left];
                left++;
            }else{
                ans += rightMax - height[right];
                right--;
            }
        }

        return ans;
    }
};
```
438. 找到字符串中所有字母异位词
窗口先吃后吐(注意滑动窗口是针对元素都为正数的情况)
```C++


一会再记

560. 和为K的子数组
使用一个叫做前缀和的技巧
```C++
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int cnt = 0;
        int curSum = 0;
        unordered_map<int, int> precount;
        precount[0] = 1;
        
        for(int i = 0; i < nums.size(); i++){
            curSum += nums[i];

            if(precount.find(curSum - k) != precount.end()){
                cnt += precount[curSum - k];
            }
            // 对于k为0的情况
            precount[curSum]++;
        }
        return cnt;
    }
};
```
















