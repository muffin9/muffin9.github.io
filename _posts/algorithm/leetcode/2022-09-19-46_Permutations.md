---
permalink: /leetcode/permutations
title: "Permutations"
last_modified_at: 2022-05-07T17:11-03:02
categories:
    - leetcode
tags:
    - recur
---

## Permutations

https://leetcode.com/problems/permutations/

### 문제

Given an array nums of distinct integers, return all the possible permutations. You can return the answer in any order.

`Input & Output`

```
Input: nums = [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

### 내풀이

```javascript
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permute = function(nums) {
    const len = nums.length;
    const visited = new Array(len);
    const result = [];
    const recur = (arr) => {
        if(arr.length === len) {
            result.push(arr);
            return;
        }
        for(let i = 0; i < len; i++) {
            if(visited[i]) continue;
            visited[i] = true;
            recur([...arr, nums[i]]);
            visited[i] = false;
        }
    }

    recur([]);
    return result;
};
```

문제에서 주어진 고유 정수가 담긴 배열을 input값으로 준다고 했으니 따로 중복처리는 생각할 필요없이 배열에 계속 다음에 올 수 있는 수를 담는 형식으로 재귀를 돌려 문제를 풀었다.

1. recur의 인자로는 빈배열(result에 담을 녀석)만 주면된다.

2. 재귀를 탈출하는 조건 base case는 인풋으로 주어진 nums의 길이와 같아질때 리턴

3. reucr문 내의 for문은 방문했던 숫자를 다시 담지 않기 위해 visited 배열로 처리하면서 계속 recur의 인자 붙여주기.