---
permalink: /leetcode/combinations
title: "Combinations"
last_modified_at: 2022-05-07T17:13-03:02
categories:
  - leetcode
tags:
  - recur
---

## Combinations

https://leetcode.com/problems/combinations/

### 문제

Given two integers n and k, return all possible combinations of k numbers chosen from the range [1, n].

You may return the answer in any order.


`Input & Output`

```
Input: n = 4, k = 2
Output: [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
Explanation: There are 4 choose 2 = 6 total combinations.
Note that combinations are unordered, i.e., [1,2] and [2,1] are considered to be the same combination.
```

### 내풀이

```javascript
/**
 * @param {number} n
 * @param {number} k
 * @return {number[][]}
 */
var combine = function(n, k) {
    const result = [];
    const recur = (arr, idx) => {
        if(arr.length === k) {
            result.push(arr);
            return;
        }

        for(let i = idx; i <= n; i++) {
            recur([...arr, i], i + 1);
        }
    }

    recur([], 1);
    return result;
};

combine(4, 2);
```

예전에 백준에서 풀어본 문제인것 같다. 주어진 n값과 k값을 이용해서 1 ~ n 을 적절히 섞어 가능한 k 길이만큼의 조합 수를 구하는건데, 주의할점은 앞의 값보다 무조건 뒤의 값이 커야 한다는 점이다.

1. reucr의 인자로는 빈배열(result에 담을 녀석), 인자 arr의 앞의 값보다 뒤 값이 무조건 크게 하기 위해 idx라는 인자 를 넣어주었다.

2. 재귀 탈출 조건으론 인자(arr)의 길이가 주어진 k가 같아질때 리턴

3. recur문 내의 for문은 1씩증가하는 idx를 시작으로 주어진 n 이하일때까지 계속 recur문 순회하며 arr에는 1씩 증가되는 값 붙여주기.