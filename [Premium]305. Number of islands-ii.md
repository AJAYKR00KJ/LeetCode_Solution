**Question**:

You are given an empty 2D binary grid grid of size m x n. The grid represents a map where 0's represent water and 1's represent land. Initially, all the cells of grid are water cells (i.e., all the cells are 0's).

We may perform an add land operation which turns the water at position into a land. You are given an array positions where positions[i] = [ri, ci] is the position (ri, ci) at which we should operate the ith operation.

Return an array of integers answer where answer[i] is the number of islands after turning the cell (ri, ci) into a land.

An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

**Example**.

![image](https://github.com/AJAYKR00KJ/LeetCode_Solution/assets/48375037/744cbfdf-da48-470b-9c37-9ef1e4b56fda)

Input: m = 3, n = 3, positions = [[0,0],[0,1],[1,2],[2,1]]
Output: [1,1,2,3]
Explanation:
Initially, the 2d grid is filled with water.
- Operation #1: addLand(0, 0) turns the water at grid[0][0] into a land. We have 1 island.
- Operation #2: addLand(0, 1) turns the water at grid[0][1] into a land. We still have 1 island.
- Operation #3: addLand(1, 2) turns the water at grid[1][2] into a land. We have 2 islands.
- Operation #4: addLand(2, 1) turns the water at grid[2][1] into a land. We have 3 islands.


**Solution: 1**:

Time Complexity: O(k log(mn)), where k == positions.length
Space Complexity: O(m*n)

```
class Solution {
public:
    int getParent(vector<int> &par, int val) {
        if(par[val]==val) return par[val];
        return par[val] = getParent(par, par[val]);
    }
    vector<int> numIslands2(int m, int n, vector<vector<int>>& positions) {
        vector<int> par(m*n, -1);
        vector<int> ans;
        vector<vector<int>> directions = {{-1,0},{0,1}, {1,0}, {0,-1}};
        int islands = 0;
        for(vector<int> position: positions) {
            int x = position[0], y = position[1], index = x*n + y;
            if(par[index]!=-1) {
                ans.push_back(islands);
                continue;
            }
            islands++;
            par[index] = index;
            for(vector<int> direction: directions) {
                int nx = x + direction[0], ny = y + direction[1];
                int adjIndex = nx * n + ny;
                if(nx>=0 && nx<m && ny>=0 && ny<n && par[adjIndex]!=-1) {
                    int pAdjIndex = getParent(par, adjIndex);
                    int pNewIndex = getParent(par, index);
                    if(pAdjIndex != pNewIndex) {
                        islands--;
                        par[pAdjIndex] = pNewIndex;
                    }
                }
            }
            ans.push_back(islands);
        }
        return ans;
    }
};
```

**Solution:2** **[**union by rank**]**
Time Complexity: O(k log(mn)), where k == positions.length
Space Complexity: O(m*n)

```
class Solution {
public:
    int getParent(vector<int> &par, int val) {
        if(par[val]==val) return par[val];
        return par[val] = getParent(par, par[val]);
    }
    void _union(int a, int b, vector<int> &par, vector<int> &rank) {
        if(rank[a]<rank[b]) {
            par[a] = b;
        } else if(rank[a]>rank[b]) {
            par[b] = a;
        } else {
            par[a] = b;
            rank[a]++;
        }
    }
    vector<int> numIslands2(int m, int n, vector<vector<int>>& positions) {
        vector<int> par(m*n, -1), rank(m*n, 1);
        vector<int> ans;
        vector<vector<int>> directions = {{-1,0},{0,1}, {1,0}, {0,-1}};
        int islands = 0;
        for(vector<int> position: positions) {
            int x = position[0], y = position[1], index = x*n + y;
            if(par[index]!=-1) {
                ans.push_back(islands);
                continue;
            }
            islands++;
            par[index] = index;
            for(vector<int> direction: directions) {
                int nx = x + direction[0], ny = y + direction[1];
                int adjIndex = nx * n + ny;
                if(nx>=0 && nx<m && ny>=0 && ny<n && par[adjIndex]!=-1) {
                    int pAdjIndex = getParent(par, adjIndex);
                    int pNewIndex = getParent(par, index);
                    if(pAdjIndex != pNewIndex) {
                        islands--;
                        _union(pAdjIndex, pNewIndex, par, rank);
                    }
                }
            }
            ans.push_back(islands);
        }
        return ans;
    }
};
```
