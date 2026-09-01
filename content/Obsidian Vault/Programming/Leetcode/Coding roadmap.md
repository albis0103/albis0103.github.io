grind75
https://www.techinterviewhandbook.org/grind75/
**必備topic** 
hash table  
linked-list  
binary tree and recursion and dfs  
stack and dfs  
queue and bfs  
用到 prefix sum的題目  
簡單的 binary search 題目  
經典的 greedy 題目  
經典的 two pointer 題目 （雙指針或是滑窗或是快慢指針）  
經典的 DP 題目  
簡單的 heap/ordered set  
簡單的 string

blink75
https://leetcode.com/discuss/post/460599/blind-75-leetcode-questions/

## 一、LeetCode 官方 Hot 100（依高頻分類）

**陣列 / 雜湊表**

- Two Sum
- 3Sum
- Majority element
- Group Anagrams
- Longest Consecutive Sequence
- Product of Array Except Self

**雙指標 / 滑動視窗**

- Container With Most Water
- Trapping Rain Water
- Longest Substring Without Repeating Characters
- Minimum Window Substring
- Sliding Window Maximum

record
[[No.209 Minimum size Subarray sum]]

**鏈結串列**

- Reverse Linked List
- Merge Two Sorted Lists
- Linked List Cycle
- Add Two Numbers
- LRU Cache（設計題，考頻極高）

**二元樹 / 二元搜尋樹**

- Maximum Depth of Binary Tree
- Validate Binary Search Tree
- Binary Tree Level Order Traversal
- Lowest Common Ancestor of a Binary Tree[[No.236  Lowest Common Ancestor of a Binary Tree]]
- 
- Serialize and Deserialize Binary Tree

**動態規劃**

- Climbing Stairs
- House Robber
- Longest Increasing Subsequence
- Coin Change
- Longest Common Subsequence
- Word Break
- Edit Distance

**回溯法**

- Subsets
- Permutations
- Combination Sum
- Word Search
- N-Queens

**圖論**

- Number of Islands[[N0.200 Number of Islands]]
- Course Schedule（拓撲排序）[[No.207 Course Schedule]]
- Clone Graph
- Pacific Atlantic Water Flow

**堆疊 / 佇列**

- Valid Parentheses [[No.125 Valid Palindrome]]
- Min Stack
- Daily Temperatures（單調堆疊）

**二元搜尋**

- Search in Rotated Sorted Array
- Find Minimum in Rotated Sorted Array

**堆積（Heap）**

- Top K Frequent Elements
- Find Median from Data Stream
- Merge K Sorted Lists

---

## 二、NeetCode 150（依官方分類架構）

NeetCode 150 是把 Blind 75 擴充後，依照下列 18 個分類系統化排列，重疊題目不重複列出，只補充 Hot 100 沒特別強調、但 NeetCode 150 額外強化的部分：

**Trie（字典樹）**— Hot 100 較少涉及但 NeetCode 特別列一章

- Implement Trie
- Design Add and Search Words
- Word Search II

**Union-Find（並查集）**

- Number of Provinces
- Redundant Connection
- Graph Valid Tree

**進階圖論**

- Alien Dictionary（拓撲排序變化題）
- Network Delay Time（Dijkstra）
- Swim in Rising Water（Binary Search + BFS/Dijkstra）
- Reconstruct Itinerary（歐拉路徑）

**進階動態規劃（二維 DP / 區間 DP）**

- Unique Paths
- Longest Palindromic Substring
- Palindromic Substrings
- Decode Ways
- Burst Balloons（區間 DP 代表題，難度高）
- Regular Expression Matching

**背包問題類**

- Partition Equal Subset Sum
- Target Sum

**位元運算**

- Single Number
- Number of 1 Bits
- Counting Bits
- Missing Number

**區間問題**

- Insert Interval
- Merge Intervals
- Non-overlapping Intervals
- Meeting Rooms II

**進階堆疊**

- Largest Rectangle in Histogram
- Car Fleet

---

## 三、兩份清單的選題差異重點

|面向|Hot 100|NeetCode 150|
|---|---|---|
|選題依據|平台實際出現頻率統計|依資料結構/演算法分類教學設計|
|涵蓋廣度|較集中在常考題型|額外涵蓋 Trie、Union-Find、位元運算等冷門但偶爾必考的類型|
|適合階段|面試前衝刺、抓考古題感|系統性建立解題模式庫|

## 建議刷題優先序

1. **陣列/雙指標/雜湊表**、**動態規劃**、**圖論（BFS/DFS）** 這三大類是兩份清單重疊度最高、企業考核最重的部分，優先刷到熟練。
2. **區間 DP（如 Burst Balloons）**、**Trie**、**Union-Find** 屬於進階但特定公司（如做搜尋、圖數據庫相關業務）會重點考，若時間有限可放後段。
3. **LRU Cache**、**Merge K Sorted Lists** 這類「資料結構設計題」在兩份清單都被視為必考，因為同時考驗資料結構理解與程式碼組織能力。

若你想針對特定目標（例如準備台灣科技業 vs 美國 FAANG、或想加強某一類型如動態規劃），我可以再幫你抓更聚焦的子清單。