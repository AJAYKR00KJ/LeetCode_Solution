# Sturdy Wall Questions

## Question 1:
### Question:
Find the maximum length of a substring of a string with the first character lexicographically smaller than its last character.

### Sample Test Case:
Input: `"abz"`  
Output: `3`

### Mine Provided Approach:
Use a 26-size array to store the rightmost index of each character. Iterate through the string, and for each index `i`, check all characters greater than `s[i]` to find the max difference.

### Your Approach:
Brute force with optimization using rightmost occurrence tracking.

### Final Solution:
```cpp
int maxSubstringLength(string s) {
    vector<int> rightmost(26, -1);
    int n = s.length(), res = 0;
    for (int i = 0; i < n; i++) rightmost[s[i] - 'a'] = i;
    for (int i = 0; i < n; i++) {
        for (int c = s[i] - 'a' + 1; c < 26; c++) {
            if (rightmost[c] > i) res = max(res, rightmost[c] - i + 1);
        }
    }
    return res;
}
```

## Question 2:
### Question:
Given an origin airport, destination airport, and a series of flights, determine whether it is possible for a package at the origin to reach the destination.  
A flight is represented as departure airport, arrival airport, departure time, and arrival time.  
The package must only take flights such that the **departure time ≥ its arrival time** at that airport.  
The package starts at the origin airport at time 0.

### Sample Test Case 1:
- Origin: `"NYC"`
- Destination: `"SFO"`
- Flights:  
  `NYC → LAX`, Departure: `0`, Arrival: `4`  
  `LAX → SFO`, Departure: `5`, Arrival: `7`  
- Output: `True`

### Sample Test Case 2:
- Origin: `"NYC"`
- Destination: `"SFO"`
- Flights:  
  `NYC → LAX`, Departure: `0`, Arrival: `4`  
  `LAX → SFO`, Departure: `3`, Arrival: `5`  
- Output: `False`

### Mine Provided Approach:
Use an adjacency list to store neighbors, and a separate map to store departure and arrival times.  
Traverse from the origin using current time and only consider flights whose departure time is ≥ current time.  
Update current time as you proceed. If the destination is reachable following these constraints, return true.

### Your Approach:
Use Dijkstra-style BFS with a priority queue to always explore the earliest reachable airport.  
Track the earliest arrival time for each airport using a visited map.

### Final Solution:
```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <queue>
#include <tuple>
#include <string>

using namespace std;

bool canTransfer(string origin, string destination, vector<tuple<string, string, int, int>> flights) {
    // Build adjacency list
    unordered_map<string, vector<tuple<string, int, int>>> adj;
    for (const auto& [from, to, dep, arr] : flights) {
        adj[from].emplace_back(to, dep, arr);
    }

    // Min-heap to prioritize earliest arrival times
    priority_queue<pair<int, string>, vector<pair<int, string>>, greater<>> pq;
    unordered_map<string, int> visitedTime;

    pq.push({0, origin});  // {arrival_time, airport}
    visitedTime[origin] = 0;

    while (!pq.empty()) {
        auto [currTime, airport] = pq.top(); pq.pop();

        if (airport == destination) return true;

        for (const auto& [next, dep, arr] : adj[airport]) {
            if (dep >= currTime) {
                // Only consider if we haven't visited or have a better arrival time
                if (!visitedTime.count(next) || visitedTime[next] > arr) {
                    visitedTime[next] = arr;
                    pq.push({arr, next});
                }
            }
        }
    }

    return false;
}
```

### Time Complexity:
- **O(E × log N)**, where:
  - `E` is the number of flights (edges)
  - `N` is the number of unique airports (nodes)
- We use a min-heap to always explore the airport with the earliest possible arrival time.


## Question 3:
### Question:
You’re given a string which may contain "bad pairs".  
A **bad pair** is defined as a pair of adjacent characters that are the same letter but in **different cases**.  
For example, `"xX"` is a bad pair, but `"xx"` or `"XX"` are not.  
Implement a solution to remove all bad pairs from the string.

### Sample Test Case:
- Input: `"abxXw"`  
- Output: `"abw"`

### Edge Case:
- Input: `""`  
- Output: `""`

---

### Mine Provided Approach:
Use a stack-based approach to process each character.  
If the top of the stack and the current character form a bad pair, pop the stack.  
Otherwise, push the character onto the stack.  

### Your Approach:
Same stack-based strategy using a string as a stack.  
A bad pair is detected by checking `abs(stack.top() - c) == 32`.

---

### Final Solution (With Stack):
```cpp
string removeBadPairs(string s) {
    string stack;
    for (char c : s) {
        if (!stack.empty() && abs(stack.back() - c) == 32) {
            stack.pop_back();  // Remove bad pair
        } else {
            stack.push_back(c);  // Keep valid char
        }
    }
    return stack;
}
```

---

### ✅ Unit Test Cases:
```cpp
void testRemoveBadPairs() {
    assert(removeBadPairs("abxXw") == "abw");
    assert(removeBadPairs("aAa") == "a");         // "aA" removed, left with "a"
    assert(removeBadPairs("aAbBcC") == "");       // All cancel
    assert(removeBadPairs("") == "");             // Empty input
    assert(removeBadPairs("abc") == "abc");       // No bad pairs
    assert(removeBadPairs("aBAbBA") == "A");      // Complex alternating case
    assert(removeBadPairs("aAaAa") == "a");       // Chain cancellations
    assert(removeBadPairs("aBBbAa") == "a");      // Interleaved case
}
```

---

## Follow-Up 1: Unit Test Design
Write tests for:
- Simple cancels
- Chains of bad pairs
- Empty input
- No bad pairs
- Complex interleaved sequences
- Single character strings
- Already clean strings

Consider modularizing your code and testing helper functions like:
- `bool isBadPair(char a, char b)`

---

## Follow-Up 2: Solution Without Using Stack
Use a pointer `i` to simulate stack behavior by writing valid characters in-place.

### Final Solution (No Stack):
```cpp
string removeBadPairsNoStack(string s) {
    int i = -1;  // Acts like the top index of a stack
    for (int j = 0; j < s.size(); ++j) {
        if (i >= 0 && abs(s[i] - s[j]) == 32) {
            --i;  // Remove bad pair
        } else {
            ++i;
            s[i] = s[j];  // Store valid character
        }
    }
    return s.substr(0, i + 1);
}
```

---

### 🔍 Dry Run of `"abxXw"` (No Stack Version)

| j | s[j] | i  | s[0..i] | Action                  |
|---|------|----|---------|-------------------------|
| 0 | a    | 0  | a       | push a                  |
| 1 | b    | 1  | ab      | push b                  |
| 2 | x    | 2  | abx     | push x                  |
| 3 | X    | 1  | ab      | pop bad pair xX         |
| 4 | w    | 2  | abw     | push w                  |

✅ Final Output: `"abw"`

---

### Time Complexity:
- **O(n)** for both approaches
- Space:
  - **O(n)** for the stack version
  - **O(1)** extra space (in-place) for the non-stack version


## Question 4:
### Question:
You are given a fixed-size array of integers and a target sum `S`.  
You can choose any integer `x`, and **truncate all array elements greater than `x` to `x`**.  
Return the integer `x` such that the sum of the resulting array is as **close as possible** (min absolute difference) to the given target `S`.

### Sample Test Case:
- Input: `arr = [2, 3, 5], target = 10`  
- Output: `5`  
  (No element is truncated → sum is 10)

- Input: `arr = [2, 3, 5], target = 6`  
- Output: `2`  
  (All values > 2 become 2 → [2, 2, 2] → sum is 6)

---

### Mine Provided Approach:
Sort the array.  
For every value `i` from 0 to target, calculate the truncated sum using prefix sum.  
Pick the value where `|sum - target|` is minimized.

**Time Complexity:** O(n log n + target × log n)

---

### Your Approach (Optimized with Binary Search):
Use binary search over possible values of `x` from `0` to `max(arr)`.

For each `mid`, calculate the truncated sum:

- Use prefix sums and `lower_bound` to determine how many values are ≤ `mid`
- Replace all values > `mid` with `mid` and compute total sum
- Keep track of the value that minimizes `|sum - target|`

---

### Final Solution:
```cpp
int findBestValue(vector<int>& arr, int target) {
    sort(arr.begin(), arr.end());
    int n = arr.size();
    vector<int> prefix(n, 0);
    prefix[0] = arr[0];

    for (int i = 1; i < n; ++i) {
        prefix[i] = prefix[i-1] + arr[i];
    }

    int left = 0, right = arr[n - 1];
    int best = 0, minDiff = INT_MAX;

    while (left <= right) {
        int mid = (left + right) / 2;
        int idx = lower_bound(arr.begin(), arr.end(), mid) - arr.begin();

        int currSum = (idx > 0 ? prefix[idx - 1] : 0) + (n - idx) * mid;
        int diff = abs(currSum - target);

        if (diff < minDiff || (diff == minDiff && mid < best)) {
            best = mid;
            minDiff = diff;
        }

        if (currSum < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return best;
}
```

---

### Time Complexity:
- **O(n log n)** for sorting and prefix sum
- **O(log max(arr)) × log n** for binary search with `lower_bound`
- **Total:** `O(n log n + log(max_val) × log n)`

---

### Follow-Up: Can We Optimize Further?
- The function that maps `x → sum after truncation` is **monotonic increasing**, so binary search is optimal.
- No closed-form or greedy shortcut exists in general due to array distribution.
- Pointer tricks or caching may help if there are **multiple queries**, but not for a one-shot problem.

✅ Conclusion:  
**Binary search** is the most efficient and clean solution — you've hit the optimal ceiling for this problem.

## Question: Pi Index Matching

### Problem Statement:
You are given a string `pi` representing digits of π (pi), such as:

```
pi = "314159265359..."
```

You need to return a list of **1-based indices `i`** such that:

```
to_string(i) == pi.substr(i - 1, len(to_string(i)))
```

That is, the digits starting at index `i - 1` in `pi` match the string representation of `i`.

---

### Examples:

#### Example 1:
**Input:**  
`pi = "314159265359"`

**Output:**  
`[3]`  
(Explanation: `i = 3`, `to_string(i) = "3"`, and `pi[2] == '3'`)

#### Example 2:
**Input:**  
`pi = "0123456789101112..."`

**Output:**  
`[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, ...]`  
(If pi is the decimal concatenation of integers, all indices match)

---

### Constraints:
- `pi.length <= 10^6`
- Must return all indices `i` such that `pi.substr(i - 1, len(str(i))) == str(i)`

---

### Final C++ Solution:
```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

vector<int> findMatchingIndexes(const string& pi) {
    vector<int> result;
    int n = pi.size();
    
    for (int i = 1; ; ++i) {
        string si = to_string(i);
        int len = si.length();
        
        if (i - 1 + len > n) break; // Stop if substring would go out of bounds
        
        if (pi.substr(i - 1, len) == si) {
            result.push_back(i);
        }
    }
    
    return result;
}

int main() {
    string pi = "314159265359";
    vector<int> matches = findMatchingIndexes(pi);

    for (int i : matches) {
        cout << i << " ";
    }
    cout << endl;

    return 0;
}
```

---

### Time Complexity:
- Let `L = pi.size()`
- Each `i` requires `O(log i)` time
- Loop runs while `i - 1 + len(i) <= L`
- ✅ **Total Complexity:** `O(L)`, efficient for up to 1 million digits

---

### Notes:
- This problem combines digit indexing with pattern detection.
- Ideal for problems involving digit-aligned pattern searching in numerical strings.


# IP Range to Country Lookup

This project implements a solution to look up the country associated with an IP address based on predefined IP ranges.

## Problem Overview

Given a list of IP address ranges, each associated with a country, the task is to determine which country an input IP address belongs to.

### Example

- IP Range: `{1.1.0.1 - 1.1.0.10}` -> Country: `IND`
- IP Range: `{1.1.0.20 - 1.1.0.30}` -> Country: `FR`

**Input:** `IP Address = 1.1.0.5`  
**Output:** `Country = IND`

Explanation: `1.1.0.5` falls within the range `{1.1.0.1 - 1.1.0.10}`, which is mapped to `IND`.

## Approach

We convert IP addresses into 32-bit integers for easier comparison. This allows us to use binary search to efficiently determine the country for a given IP address.

### Steps to Solve:

1. **Convert IP Address to Integer**: Convert the given IP address (e.g., `1.1.0.5`) into a 32-bit integer for easy comparison.
2. **Store IP Ranges**: Store the IP ranges and countries as structs, and ensure the ranges are sorted.
3. **Binary Search**: Use binary search to find the country associated with the input IP address.

## Code Implementation

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <sstream>
#include <algorithm>

using namespace std;

// Function to convert an IP address to a 32-bit integer
uint32_t ipToInt(const string& ip) {
    int a, b, c, d;
    sscanf(ip.c_str(), "%d.%d.%d.%d", &a, &b, &c, &d);
    return ((uint32_t)a << 24) | (b << 16) | (c << 8) | d;
}

// Struct to store the range of IPs and associated country
struct IPRange {
    uint32_t start;
    uint32_t end;
    string country;
};

// Function to find the country for a given IP address
string findCountry(const vector<IPRange>& ranges, const string& ipStr) {
    uint32_t ip = ipToInt(ipStr);

    int low = 0, high = ranges.size() - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (ip >= ranges[mid].start && ip <= ranges[mid].end) {
            return ranges[mid].country;
        } else if (ip < ranges[mid].start) {
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }

    return "Unknown"; // IP not found in any range
}

int main() {
    // Define the ranges with corresponding countries
    vector<IPRange> ranges = {
        {ipToInt("1.1.0.1"), ipToInt("1.1.0.10"), "IND"},
        {ipToInt("1.1.0.20"), ipToInt("1.1.0.30"), "FR"}
    };

    // Query IP addresses
    string ip1 = "1.1.0.5";
    cout << "IP: " << ip1 << " -> Country: " << findCountry(ranges, ip1) << endl; // Output: IND

    string ip2 = "1.1.0.25";
    cout << "IP: " << ip2 << " -> Country: " << findCountry(ranges, ip2) << endl; // Output: FR

    string ip3 = "1.1.0.15";
    cout << "IP: " << ip3 << " -> Country: " << findCountry(ranges, ip3) << endl; // Output: Unknown

    return 0;
}
```


# Cake Cutting Problem

## Question 1:
### Question:
Determine if a valid vertical cut can be made through a rectangular cake such that the cut does not intersect any of the toppings, and both resulting pieces contain at least one topping.

### Sample Test Case:
Input: `Topping a (0,6,0,1); Topping b (7,8,0,4); Topping c (0,1,2,4);`  
Output: `True`

### Mine Provided Approach:
First, model the cake as a 2D grid where each topping is represented as a rectangle.
Iterate over every possible vertical cut between two columns, ensuring that the cut doesn't intersect any topping and that both resulting pieces contain at least one topping.

### Your Approach:
Sweep Line Approach: Use events to represent the start and end of each topping along the x-axis.
Process the events in sorted order, updating active toppings and checking if a valid vertical cut can be made at each position.

### Final Solution:
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <unordered_set>
using namespace std;

struct Topping {
    char id;
    int x1, x2, y1, y2;
};

struct Event {
    int x;
    bool isStart; // true if start, false if end
    char id;

    bool operator<(const Event& other) const {
        return x < other.x || (x == other.x && isStart < other.isStart); // ends processed before starts at same x
    }
};

bool canMakeValidCut(const vector<Topping>& toppings) {
    vector<Event> events;
    unordered_set<char> allToppings;

    for (const auto& t : toppings) {
        events.push_back({t.x1, true, t.id});       // start of a topping
        events.push_back({t.x2 + 1, false, t.id});  // end of a topping (non-inclusive)
        allToppings.insert(t.id);
    }

    sort(events.begin(), events.end());

    unordered_set<char> active;       // toppings currently intersecting sweep line
    unordered_set<char> ended;        // toppings fully to the left of current x

    for (int i = 0; i < events.size(); ++i) {
        int currX = events[i].x;

        // Process all events at currX
        while (i < events.size() && events[i].x == currX) {
            if (events[i].isStart) {
                active.insert(events[i].id);
            } else {
                active.erase(events[i].id);
                ended.insert(events[i].id);
            }
            i++;
        }
        --i; // compensate for outer loop increment

        // At any point *between* events, check if we can cut
        // Valid cut: no active topping intersects this x
        // and there is at least one ended and one upcoming topping
        if (active.empty()) {
            int leftCount = ended.size();
            int rightCount = allToppings.size() - leftCount;
            if (leftCount > 0 && rightCount > 0) {
                return true;
            }
        }
    }

    return false;
}

int main() {
    vector<Topping> toppings = {
        {'a', 0, 6, 0, 1},
        {'b', 7, 8, 0, 4},
        {'c', 0, 1, 2, 4}
    };

    cout << (canMakeValidCut(toppings) ? "True" : "False") << endl;  // Should print: True
    return 0;
}
```

# Pathfinding Problem: Counting Unique Paths in a Grid

## Problem Description

You are given an `n x m` grid, starting from the bottom-left corner `(m, 0)` and aiming to reach the bottom-right corner `(m, n)`. The allowed movement directions are:
- **Right** `(0, 1)`
- **Diagonally Up-Right** `(-1, 1)`
- **Diagonally Down-Right** `(1, 1)`

You are tasked with finding the total number of unique paths from the starting point to the destination following these allowed movement rules.

---

## Solution Approach

### Initial Question: Basic Pathfinding (DFS Solution)

We can solve this problem using a **Depth First Search (DFS)** approach. The general idea is to explore every possible path starting from `(m, 0)` and moving toward the destination `(m, n)` while adhering to the movement constraints.

#### Key Steps:
1. **Start at (m, 0)** and recursively explore all valid moves to the right, diagonally up-right, and diagonally down-right.
2. **Terminate the search** when the destination `(m, n)` is reached, returning `1` as a valid path.
3. **Backtrack** to explore all possible routes by marking the cell as visited and unvisited during the DFS.
4. **Avoid cycles** by keeping track of visited cells in a `visited` array to prevent revisiting the same cell.

#### DFS Code Snippet:

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

class UniquePaths {
public:
    int countPaths(int m, int n) {
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        return dfs(m, n, m-1, 0, visited);
    }

private:
    unordered_map<string, int> memo;

    int dfs(int m, int n, int row, int col, vector<vector<bool>>& visited) {
        if (row < 0 || row >= m || col < 0 || col >= n || visited[row][col])
            return 0;
        if (row == m-1 && col == n-1)
            return 1;

        string key = to_string(row) + "," + to_string(col);
        if (memo.find(key) != memo.end()) return memo[key];

        visited[row][col] = true;
        int totalPaths = 0;

        totalPaths += dfs(m, n, row, col + 1, visited);
        totalPaths += dfs(m, n, row - 1, col + 1, visited);
        totalPaths += dfs(m, n, row + 1, col + 1, visited);

        visited[row][col] = false;
        memo[key] = totalPaths;
        return totalPaths;
    }
};

```

#### DFS Code Snippet With Check Points:

```cpp
#include <map>

class UniquePathsWithCheckpoints {
public:
    int countPaths(int m, int n, vector<pair<int,int>>& checkpoints) {
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        map<pair<int,int>, bool> checkpointVisited;
        for (auto& p : checkpoints)
            checkpointVisited[p] = false;
        
        return dfs(m, n, m-1, 0, visited, checkpointVisited);
    }

private:
    unordered_map<string, int> memo;

    string generateKey(int row, int col, const map<pair<int,int>, bool>& checkpointVisited) {
        string key = to_string(row) + "," + to_string(col);
        for (auto& cp : checkpointVisited)
            key += "|" + to_string(cp.first.first) + "," + to_string(cp.first.second) + "=" + (cp.second ? "1" : "0");
        return key;
    }

    int dfs(int m, int n, int row, int col, vector<vector<bool>>& visited, map<pair<int,int>, bool>& checkpointVisited) {
        if (row < 0 || row >= m || col < 0 || col >= n || visited[row][col])
            return 0;
        if (row == m-1 && col == n-1) {
            for (auto& cp : checkpointVisited) {
                if (!cp.second) return 0;
            }
            return 1;
        }

        string key = generateKey(row, col, checkpointVisited);
        if (memo.find(key) != memo.end()) return memo[key];

        visited[row][col] = true;
        bool wasCheckpoint = false;
        if (checkpointVisited.find({row, col}) != checkpointVisited.end()) {
            checkpointVisited[{row, col}] = true;
            wasCheckpoint = true;
        }

        int totalPaths = 0;
        totalPaths += dfs(m, n, row, col + 1, visited, checkpointVisited);
        totalPaths += dfs(m, n, row - 1, col + 1, visited, checkpointVisited);
        totalPaths += dfs(m, n, row + 1, col + 1, visited, checkpointVisited);

        if (wasCheckpoint) {
            checkpointVisited[{row, col}] = false;
        }
        visited[row][col] = false;
        memo[key] = totalPaths;
        return totalPaths;
    }
};


```

#### DFS Code Snippet With Ordered Check Points:

```cpp
class UniquePathsWithOrderedCheckpoints {
public:
    int countPaths(int m, int n, vector<pair<int,int>>& checkpoints) {
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        return dfs(m, n, m-1, 0, visited, checkpoints, 0);
    }

private:
    unordered_map<string, int> memo;

    string generateKey(int row, int col, int checkpointIndex) {
        return to_string(row) + "," + to_string(col) + "," + to_string(checkpointIndex);
    }

    int dfs(int m, int n, int row, int col, vector<vector<bool>>& visited, vector<pair<int,int>>& checkpoints, int checkpointIndex) {
        if (row < 0 || row >= m || col < 0 || col >= n || visited[row][col])
            return 0;
        if (row == m-1 && col == n-1)
            return (checkpointIndex == checkpoints.size()) ? 1 : 0;

        string key = generateKey(row, col, checkpointIndex);
        if (memo.find(key) != memo.end()) return memo[key];

        visited[row][col] = true;
        bool movedCheckpoint = false;
        if (checkpointIndex < checkpoints.size() && make_pair(row, col) == checkpoints[checkpointIndex]) {
            movedCheckpoint = true;
            checkpointIndex++;
        }

        int totalPaths = 0;
        totalPaths += dfs(m, n, row, col + 1, visited, checkpoints, checkpointIndex);
        totalPaths += dfs(m, n, row - 1, col + 1, visited, checkpoints, checkpointIndex);
        totalPaths += dfs(m, n, row + 1, col + 1, visited, checkpoints, checkpointIndex);

        visited[row][col] = false;
        memo[key] = totalPaths;
        return totalPaths;
    }
};

```

# Pathfinding Problem: Complexity Analysis

## 1. Initial Problem (No Checkpoints)
✅ **Only three types of moves:** right, up-right, down-right.  
✅ **Memoizing** based on `(row, col)`.

- **Number of unique states:** `m * n`
- **At each state:** Constant branching factor of 3 (due to 3 possible moves).

### Final Time Complexity:
- `O(m × n)`  
- **Space complexity** is also `O(m × n)` due to the memoization table.

---

## 2. Follow-up 1 (Unordered Checkpoints)
✅ **Memoizing** based on `(row, col)` and **which checkpoints have been visited**.

- **(row, col):** Can be at `m * n` positions.
- For `k` checkpoints, each checkpoint can either be visited or not, resulting in `2^k` possible visited states.

### Final Time Complexity:
- `O(m × n × 2^k)`  
- **Space complexity** is also `O(m × n × 2^k)` due to memoization.

⚡ **Note:** `2^k` becomes very large if `k` (number of checkpoints) is large, but manageable for small `k` (typically `k ≤ 15-20`).

---

## 3. Follow-up 2 (Ordered Checkpoints)
✅ **Memoizing** based on `(row, col, checkpointIndex)`.

- **(row, col):** Can be at `m * n` positions.
- **checkpointIndex:** Can be from 0 to `k` (number of checkpoints).

### Final Time Complexity:
- `O(m × n × k)`  
- **Space complexity** is also `O(m × n × k)`.

⚡ **Note:** This version is much faster than the unordered version because it doesn't require handling `2^k` combinations.


# Template String Replacement Problem
## Question 1:
### Question:
You are given a map as a template with key-value pairs, and an input string. The task is to replace any placeholders (i.e., `%<key>%`) in the input string with the corresponding values from the template map. The process continues recursively until no more placeholders can be replaced.

### Example

Given Map:

```cpp
{
  "X": "123",
  "Y": "456_%X%"
}
```

### Sample Test Case:
Input: `"%X%_%Y%"`  
Output: `"123_456_123"`


### Final Solution:
```cpp
#include <iostream>
#include <unordered_map>
#include <stack>
#include <string>
using namespace std;

string replaceTemplate(const string &input, const unordered_map<string, string> &templateMap) {
    string result = "";
    stack<char> st;
    int n = input.length();
    int i = 0;
    
    while (i < n) {
        if (input[i] == '%') {
            // Handle the case where we found a '%' start
            int start = i + 1;
            while (i < n && input[i] != '%') {
                i++;
            }
            int end = i; // mark the end of the placeholder
            string placeholder = input.substr(start, end - start);
            
            // Check if placeholder exists in templateMap
            if (templateMap.find(placeholder) != templateMap.end()) {
                string replacement = templateMap.at(placeholder);
                for (char c : replacement) {
                    st.push(c);  // Push the replacement string characters to the stack
                }
            } else {
                // Placeholder not found, just put it as is
                st.push('%');
                for (char c : placeholder) {
                    st.push(c);
                }
                st.push('%');
            }
        } else {
            // Handle regular characters (numbers, underscores)
            st.push(input[i]);
        }
        i++;
    }

    // Reconstruct the result from the stack
    while (!st.empty()) {
        result = st.top() + result;
        st.pop();
    }
    return result;
}

int main() {
    unordered_map<string, string> templateMap = {
        {"X", "123"},
        {"Y", "456_%X%"}
    };

    string input = "%X%_%Y%";

    string output = replaceTemplate(input, templateMap);

    cout << "Output: " << output << endl;

    return 0;
}
```

# Restaurant Waitlist System Design

## Problem Statement

Design a system to manage a restaurant waitlist with the following requirements:

1. **Add Customer Groups**: New customer groups can be added to the waitlist.
2. **Remove Customer Groups**: A customer group can leave the waitlist at any time.
3. **Assign Table**: Given a table size, find the group that best fits the table. A group is eligible for a table if its size is ≤ table size, and among eligible groups, the one that has been waiting the longest should be assigned the table.

## Approach

We use a combination of:
- A **map** (`map<int, priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>>>`) to store groups, where keys represent group sizes and the values are min-heaps ordered by their arrival times.
- An **unordered_map** (`unordered_map<int, bool>`) to track whether a group is still waiting or has left.
- A **timer** variable to track the order of arrival.

### Operations:
1. **Add Group**:
   - For each new group, the group size is used to determine the key in the map. The arrival time (a timer) is used to ensure we prioritize earlier arrivals when two groups have the same size.
   - The group’s waiting status is set to true when it is added.
  
2. **Remove Group**:
   - When a group leaves, we mark its status as not waiting but don't remove it from the heap immediately. This allows us to "lazy delete" invalid groups later.
  
3. **Assign Table**:
   - Given a table size, we find the largest group that can fit (whose size ≤ table size). The group with the smallest arrival time (earliest) is chosen from among the largest groups.

### Optimizations:
- **Map**: We use a `map<int, minHeap<pair<int, int>>>` instead of `unordered_map` because `map` keeps the keys (group sizes) sorted, which allows for efficient searching for the largest group size ≤ table size.
- **Lazy Deletion**: When a group leaves, it is not immediately removed from the heap. Instead, we remove it lazily during the table assignment process when we encounter an invalid (non-waiting) group.

### Updated Design

```cpp
#include <iostream>
#include <unordered_map>
#include <map>
#include <queue>
#include <vector>
using namespace std;

class WaitList {
private:
    // groupSize -> minHeap of (arrivalTime, groupId)
    map<int, priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>>> groupSizeMap;
    unordered_map<int, bool> isWaiting; // groupId -> true/false
    int timer = 0; // to assign arrival order

public:
    // Add new customer group
    void addGroup(int groupId, int groupSize) {
        groupSizeMap[groupSize].push({timer++, groupId});
        isWaiting[groupId] = true;
    }

    // A group leaves
    void removeGroup(int groupId) {
        isWaiting[groupId] = false;
    }

    // Find eligible group for table
    int findGroup(int tableSize) {
        // Find the largest groupSize <= tableSize
        auto it = groupSizeMap.upper_bound(tableSize); // strictly greater
        if (it == groupSizeMap.begin()) {
            return -1; // no group fits
        }
        --it; // step back to <= tableSize

        while (true) {
            auto& heap = it->second;
            while (!heap.empty()) {
                auto [arriveTime, groupId] = heap.top();
                if (isWaiting[groupId]) {
                    isWaiting[groupId] = false; // group assigned
                    heap.pop();
                    return groupId;
                }
                heap.pop(); // lazy removal of invalid groups
            }
            if (it == groupSizeMap.begin()) break;
            --it; // check next smaller groupSize
        }
        return -1; // no valid group found
    }
};
```

### Optimized
```cpp
#include <bits/stdc++.h>
using namespace std;

class WaitList {
private:
    map<int, list<pair<int, int>>> groupSizeToGroups; // groupSize -> list of {arrivalTime, groupId}
    unordered_map<int, list<pair<int, int>>::iterator> groupIdToNode; // groupId -> node in list
    int arrivalTime = 0; // incrementing timer to maintain arrival order

public:
    // Add new group
    void addGroup(int groupId, int groupSize) {
        groupSizeToGroups[groupSize].push_back({arrivalTime++, groupId});
        auto it = groupSizeToGroups[groupSize].end();
        --it; // iterator to the newly added group
        groupIdToNode[groupId] = it;
    }

    // Group leaves
    void removeGroup(int groupId) {
        if (groupIdToNode.count(groupId)) {
            auto it = groupIdToNode[groupId];
            int size = it->first; // but we don't know size directly
            for (auto& [sz, lst] : groupSizeToGroups) {
                if (lst.empty()) continue;
                if (groupId == it->second) {
                    lst.erase(it);
                    break;
                }
            }
            groupIdToNode.erase(groupId);
        }
    }

    // Assign table
    int assignTable(int tableSize) {
        auto it = groupSizeToGroups.upper_bound(tableSize);
        if (it == groupSizeToGroups.begin()) return -1; // no group fits
        --it; // largest groupSize <= tableSize
        if (it->second.empty()) return -1; // no group waiting

        // Pick earliest arrival group
        int groupId = it->second.front().second;
        groupIdToNode.erase(groupId);
        it->second.pop_front();
        return groupId;
    }
};
```

### More Optimization
```cpp
#include <bits/stdc++.h>
using namespace std;

class WaitList {
private:
    map<int, list<int>> groupSizeToGroups; // groupSize -> list of groupId (maintains arrival order)
    unordered_map<int, pair<int, list<int>::iterator>> groupIdToNode; // groupId -> {groupSize, node in list}

public:
    // Add a new group
    void addGroup(int groupId, int groupSize) {
        groupSizeToGroups[groupSize].push_back(groupId);
        auto it = prev(groupSizeToGroups[groupSize].end()); // iterator to the newly added group
        groupIdToNode[groupId] = {groupSize, it};
    }

    // Remove a group
    void removeGroup(int groupId) {
        if (groupIdToNode.count(groupId)) {
            auto [size, it] = groupIdToNode[groupId];
            groupSizeToGroups[size].erase(it);
            if (groupSizeToGroups[size].empty()) {
                groupSizeToGroups.erase(size); // optional: clean up empty sizes
            }
            groupIdToNode.erase(groupId);
        }
    }

    // Assign a table
    int assignTable(int tableSize) {
        auto it = groupSizeToGroups.upper_bound(tableSize);
        if (it == groupSizeToGroups.begin()) return -1; // no group fits
        --it; // largest groupSize <= tableSize
        if (it->second.empty()) return -1;

        int groupId = it->second.front(); // get the earliest arriving group
        it->second.pop_front(); // remove from list
        groupIdToNode.erase(groupId);
        if (it->second.empty()) {
            groupSizeToGroups.erase(it); // optional: clean up
        }
        return groupId;
    }
};
```
### ✨ Time Complexities:

| Operation      | Time Complexity   |
|----------------|-------------------|
| `addGroup`     | O(log M)          |
| `removeGroup`  | O(1)              |
| `assignTable`  | O(log M)          |


### ✨ Problem Statement:
You are given a sorted array of strings and a prefix. The task is to return the number of strings in the array that start with the given prefix.

---

### 🛠️ Approaches:

#### 1. Brute Force Approach:
**Description:**  
Iterate over each element in the array and check if the element starts with the given prefix.

**Time Complexity:** O(n * m), where:
- `n` is the number of strings in the array.
- `m` is the average length of the strings.

**Steps:**
1. Iterate through each string in the array.
2. For each string, check if it starts with the given prefix using string functions like `startswith` or equivalent.
3. Maintain a count and return the result after iterating through all strings.

**Example Code:**

```cpp
int countPrefix(const vector<string>& arr, const string& prefix) {
    int count = 0;
    for (const string& s : arr) {
        if (s.substr(0, prefix.length()) == prefix) {
            count++;
        }
    }
    return count;
}
```
### Trie Approach
```cpp
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    int count = 0; // frequency of words passing through this node
};

void insert(TrieNode* root, const string& word) {
    TrieNode* node = root;
    for (char c : word) {
        if (node->children.find(c) == node->children.end()) {
            node->children[c] = new TrieNode();
        }
        node = node->children[c];
        node->count++;
    }
}

int countPrefix(TrieNode* root, const string& prefix) {
    TrieNode* node = root;
    for (char c : prefix) {
        if (node->children.find(c) == node->children.end()) {
            return 0; // no such prefix exists
        }
        node = node->children[c];
    }
    return node->count;
}
```
### Binary Search Approach
```cpp
int countPrefix(const vector<string>& arr, const string& prefix) {
    int left = lower_bound(arr.begin(), arr.end(), prefix) - arr.begin();
    string prefix_end = prefix;
    prefix_end[prefix_end.size() - 1]++;  // Increment last character to find the end
    int right = lower_bound(arr.begin(), arr.end(), prefix_end) - arr.begin();
    return right - left;
}
```

# Sturdy Wall Builder

This program calculates the number of ways to build a sturdy wall of a specified height and width, using bricks of given sizes. The key condition is that adjacent rows should not align at the same positions where the bricks end, except at the ends of the wall.

### Problem Description

You are given:
- **height**: The height of the wall (number of rows).
- **width**: The width of the wall (length of each row).
- **bricks**: An array of brick sizes, each brick has height 1 and width as described by the array. Each brick can be used an infinite number of times.

The program calculates how many ways you can arrange the bricks to form a sturdy wall, ensuring that adjacent rows do not align their brick ends at the same positions except at the wall edges.

### Key Constraints
- Rows in the wall must be exactly `width` units long.
- Adjacent rows must not align at the same positions except at the start and end of the wall.

### Approach

The solution uses a **recursive depth-first search (DFS)** strategy combined with **memoization** to efficiently compute the number of valid ways to build the wall. Each row's brick positions (cut points) are stored and used to ensure the adjacency condition is met.

1. **Row Configuration Generation**: For a given row width, generate all possible ways to fill the row using the given bricks and store the cut positions.
2. **DFS with Memoization**: Recursively build the wall row by row, making sure that no adjacent rows share the same cut positions (except at the wall edges).
3. **Memoization**: To avoid recalculating the same state (row and cut positions), results are stored using memoization.

### Time Complexity

- **Generating Row Configurations**: This step is exponential with respect to the width of the wall (since we try every combination of brick sizes).
- **DFS**: The recursive depth is proportional to the height of the wall, and for each row, we explore all possible valid row configurations, ensuring that no cuts from adjacent rows align.

### Example

Given the inputs:
- **Height**: 2
- **Width**: 3
- **Bricks**: `[1, 2]`

The program will output the number of ways to build the wall. For this example, the result is `4` because there are 4 valid configurations.

### Code Explanation

#### 1. `buildRows` Function
Generates all possible row configurations using the given brick sizes and stores the cut positions for each configuration.

#### 2. `dfs` Function
Recursively builds the wall row by row, ensuring no cuts from adjacent rows align (except at the wall boundaries). Uses memoization to store previously computed results to optimize the solution.

#### 3. `isCompatible` Function
Checks whether two rows’ cut positions are compatible, meaning they don't align at the same positions except at the wall edges.

#### 4. `toString` Function
Converts the cut positions into a string format for easy use in memoization.

### Running the Program

1. Clone the repository to your local machine.
2. Compile the C++ code using your preferred C++ compiler.

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 1e9 + 7;

vector<vector<int>> possibleRows;   // Stores all possible row configurations (cut positions)
unordered_map<string, int> memo;    // Memoization to avoid recomputation
vector<int> bricks;                 // The list of available brick sizes
int wallHeight, wallWidth;           // Height and Width of the wall

// Function to recursively build all possible row configurations
void buildRows(int pos, vector<int>& cuts) {
    if (pos == wallWidth) {  // If the row is filled exactly to the required width
        possibleRows.push_back(cuts);  // Store the cut positions of this row
        return;
    }
    for (int brick : bricks) {  // Try placing each brick at the current position
        if (pos + brick <= wallWidth) {  // Ensure we do not exceed the width of the wall
            if (pos + brick != wallWidth) cuts.push_back(pos + brick);  // Add cut if not at the end of the row
            buildRows(pos + brick, cuts);  // Recurse to the next position
            if (pos + brick != wallWidth) cuts.pop_back();  // Backtrack by removing the last cut
        }
    }
}

// Function to check if two rows' cuts are compatible
bool isCompatible(const vector<int>& prev, const vector<int>& curr) {
    int i = 0, j = 0;
    while (i < prev.size() && j < curr.size()) {
        if (prev[i] == curr[j]) return false;  // If the cuts align, they are not compatible
        if (prev[i] < curr[j]) i++;  // Move to the next cut in the previous row
        else j++;  // Move to the next cut in the current row
    }
    return true;  // If no cuts overlap, the rows are compatible
}

// Convert the cut vector to a string for memoization
string toString(const vector<int>& cuts) {
    string s;
    for (int x : cuts) {
        s += to_string(x) + ",";
    }
    return s;
}

// Recursive DFS function to build the wall row by row
int dfs(int row, vector<int>& prevCuts) {
    if (row == wallHeight) return 1;  // If we've successfully built all rows, return 1
    
    string key = to_string(row) + "#" + toString(prevCuts);  // Key for memoization
    if (memo.count(key)) return memo[key];  // If this state has been computed before, return it

    int ways = 0;
    for (auto& cuts : possibleRows) {  // Try each possible row configuration
        if (isCompatible(prevCuts, cuts)) {  // Check if the current row is compatible with the previous one
            ways = (ways + dfs(row + 1, cuts)) % MOD;  // Recurse to the next row
        }
    }
    
    return memo[key] = ways;  // Memoize the result for this state
}

// Main function to build the wall and return the number of ways
int buildWall(int height, int width, vector<int>& brickSizes) {
    wallHeight = height;
    wallWidth = width;
    bricks = brickSizes;
    possibleRows.clear();  // Clear any previous row configurations
    memo.clear();  // Clear the memoization cache
    
    vector<int> cuts;
    buildRows(0, cuts);  // Generate all possible row configurations

    vector<int> emptyCuts;  // Starting with an empty previous row
    return dfs(0, emptyCuts);  // Start building the wall from the first row
}

// Main function to test the program
int main() {
    int height = 2, width = 3;
    vector<int> brickSizes = {1, 2};
    cout << buildWall(height, width, brickSizes) << endl;  // Output the number of ways
    return 0;
}
```


## More problems to look
- https://leetcode.com/problems/check-if-grid-can-be-cut-into-sections/description/

