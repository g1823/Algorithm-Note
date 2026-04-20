# 前缀树 (Trie)

## 一、作用与适用场景

### 核心作用
前缀树（也叫字典树、单词查找树）是一种**树形数据结构**，用于高效地存储和检索字符串集合中的键。

### 解决的问题
- **前缀匹配**：查找所有以某前缀开头的字符串
- **字符串检索**：判断某字符串是否在集合中
- **IP路由表**（Trie 的另一个名字来源）
- **自动补全**：搜索引擎、输入法的提示功能

### 时间复杂度
| 操作 | 时间复杂度 | 说明 |
|------|-----------|------|
| 插入 | O(m) | m 为字符串长度 |
| 查找 | O(m) | m 为字符串长度 |
| 前缀匹配 | O(m + n) | n 为匹配数量 |

### 空间复杂度
- 空间消耗较大，每个节点可能有多个子节点
- 可用数组（26字母）或 HashMap 优化

---

## 二、基本结构

### 节点定义
```java
class TrieNode {
    // 26个小写字母的孩子节点
    TrieNode[] children = new TrieNode[26];
    // 标记从根到该节点的路径是否构成一个完整的单词
    boolean isEnd = false;
}
```

### 图示
```
插入 "apple", "app", "apply" 后的前缀树结构：

root
├── a
│   └── p
│       ├── p
│       │   ├── l
│       │   │   └── e (isEnd=true)     → "apple"
│       │   └── y (isEnd=true)        → "appy"
│       │                             (无此单词，仅示例)
│       └── l
│           └── y (isEnd=true)        → "app" + "apply"
```

---

## 三、完整代码实现

### Java 实现（基于 leetCode/moderately/Trie.java）

```java
class Trie {
    /**
     * 只有26个小写字母
     */
    Trie[] children = new Trie[26];
    boolean isEnd = false;

    public Trie() {
    }

    /**
     * 插入一个单词
     * 时间复杂度：O(m)，m 为单词长度
     */
    public void insert(String word) {
        Trie node = this;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                node.children[index] = new Trie();
            }
            node = node.children[index];
        }
        node.isEnd = true;
    }

    /**
     * 搜索一个完整的单词
     * 时间复杂度：O(m)
     */
    public boolean search(String word) {
        Trie node = searchPrefix(word);
        return node != null && node.isEnd;
    }

    /**
     * 判断是否存在以 prefix 为前缀的单词
     * 时间复杂度：O(m)
     */
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }

    /**
     * 辅助方法：查找前缀对应的最后一个节点
     */
    private Trie searchPrefix(String prefix) {
        Trie node = this;
        for (char c : prefix.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                return null;
            }
            node = node.children[index];
        }
        return node;
    }
}
```

---

## 四、应用场景详解

### 1. LeetCode 212. 单词搜索 II

**问题**：在二维网格中找到所有可以组成单词的路径

**思路**：
- 构建前缀树存储所有目标单词
- DFS 遍历网格
- 利用前缀树剪枝，避免无效搜索

```java
class Solution {
    private Set<String> result = new HashSet<>();
    private Trie trie = new Trie();

    public List<String> findWords(char[][] board, String[] words) {
        // 1. 构建前缀树
        for (String word : words) {
            trie.insert(word);
        }

        // 2. DFS 遍历网格
        int m = board.length, n = board[0].length;
        boolean[][] visited = new boolean[m][n];
        
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                dfs(board, visited, "", i, j, trie);
            }
        }

        return new ArrayList<>(result);
    }

    private void dfs(char[][] board, boolean[][] visited, 
                    String path, int i, int j, Trie node) {
        // 边界检查
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length) {
            return;
        }
        if (visited[i][j]) return;

        char c = board[i][j];
        Trie child = node.children[c - 'a'];
        if (child == null) return;  // 剪枝

        visited[i][j] = true;
        String newPath = path + c;
        
        if (child.isEnd) {
            result.add(newPath);  // 找到一个单词
        }

        visited[i][j] = false;  // 回溯
    }
}
```

### 2. 前缀和统计

```java
/**
 * 统计以某前缀开头的单词数量
 */
class TrieWithCount {
    TrieNode root = new TrieNode();
    
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        int prefixCount = 0;  // 记录以该节点为前缀的单词数
        boolean isEnd = false;
    }

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                node.children[index] = new TrieNode();
            }
            node = node.children[index];
            node.prefixCount++;
        }
        node.isEnd = true;
    }

    public int countPrefix(String prefix) {
        TrieNode node = root;
        for (char c : prefix.toCharArray()) {
            int index = c - 'a';
            if (node.children[index] == null) {
                return 0;
            }
            node = node.children[index];
        }
        return node.prefixCount;
    }
}
```

### 3. 字符串替换

```java
/**
 * 最短替换词：在词典中找可以替换原词的最短替换词
 */
class ReplaceWords {
    public String replaceWords(List<String> dict, String sentence) {
        Trie trie = new Trie();
        for (String word : dict) {
            trie.insert(word);
        }

        StringBuilder result = new StringBuilder();
        String[] words = sentence.split(" ");
        
        for (int i = 0; i < words.length; i++) {
            String word = words[i];
            TrieNode node = trie.root;
            StringBuilder replacement = new StringBuilder();
            
            for (char c : word.toCharArray()) {
                int index = c - 'a';
                if (node.children[index] == null || node.isEnd) {
                    break;
                }
                replacement.append(c);
                node = node.children[index];
            }
            
            if (node.isEnd) {
                result.append(replacement);
            } else {
                result.append(word);
            }
            
            if (i < words.length - 1) {
                result.append(" ");
            }
        }
        
        return result.toString();
    }
}
```

---

## 五、前缀树的变体

### 1. 压缩前缀树（Trie 的空间优化）

将只有一个子节点的节点合并：

```java
class CompressedTrie {
    static class Node {
        String prefix;      // 存储公共前缀
        Map<String, Node> children;
        boolean isEnd;
    }
}
```

### 2. 后缀树
用于处理字符串的后缀问题，如最长重复子串。

### 3. 可持久化前缀树
支持版本历史查询。

---

## 六、性能优化技巧

### 1. 使用数组 vs HashMap

```java
// 小字符集（26字母）：用数组，O(1) 查找
TrieNode[] children = new TrieNode[26];

// 大字符集：用 HashMap，节省空间
Map<Character, TrieNode> children = new HashMap<>();
```

### 2. 删除操作

```java
public void delete(String word) {
    delete(root, word, 0);
}

private boolean delete(TrieNode node, String word, int depth) {
    if (node == null) return false;
    
    if (depth == word.length()) {
        if (!node.isEnd) return false;
        node.isEnd = false;
        // 如果没有子节点，可以删除该节点（递归向上）
        return hasNoChildren(node);
    }
    
    char c = word.charAt(depth);
    TrieNode child = node.children[c - 'a'];
    
    if (child == null) return false;
    
    boolean shouldDeleteChild = delete(child, word, depth + 1);
    
    if (shouldDeleteChild) {
        node.children[c - 'a'] = null;
        return !node.isEnd && hasNoChildren(node);
    }
    
    return false;
}

private boolean hasNoChildren(TrieNode node) {
    for (TrieNode child : node.children) {
        if (child != null) return false;
    }
    return true;
}
```

---

## 七、复杂度对比

### 与其他数据结构的对比

| 数据结构 | 插入 | 查找 | 前缀匹配 | 空间 |
|---------|------|------|---------|------|
| 数组/Set | O(n) | O(1) | O(n) | O(n) |
| 哈希表 | O(n) | O(1) | O(1)* | O(n) |
| 平衡树 | O(n log n) | O(log n) | O(log n) | O(n) |
| **前缀树** | **O(n)** | **O(n)** | **O(n)** | O(Σn × Σ字符) |
| **压缩前缀树** | O(n) | O(n) | O(n) | 较小 |

*需要遍历所有字符串

---

## 八、注意事项

1. **空间开销**：如果字符集大且字符串无公共前缀，空间消耗很大
2. **内存优化**：考虑使用数组池或对象池
3. **字符集**：小字符集用数组，大字符集用 HashMap
4. **边界处理**：注意处理空字符串和空树

---

## 九、相关题目

| 题号 | 名称 | 核心考点 |
|------|------|----------|
| 208 | 实现 Trie | 前缀树基础 |
| 212 | 单词搜索 II | 前缀树 + DFS |
| 648 | 单词替换 | 前缀树 |
| 677 | 键值映射 | 前缀树 + 前缀和 |
| 720 | 词典中最长的单词 | 前缀树 |
| 820 | 单词的压缩编码 | 前缀树 / 后缀处理 |
| 1032 | 字符流 | 前缀树 + 逆向思维 |
