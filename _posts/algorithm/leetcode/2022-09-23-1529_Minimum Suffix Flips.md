---
permalink: /leetcode/MinimumSuffixFlips
title: "Minimum Suffix Flips"
categories: Leetcode
tags: Bruteforce

toc: true
toc_sticky: true


date: 2022-09-23
last_modified_at: 2020-09-23
---

## Combinations

https://leetcode.com/problems/minimum-suffix-flips/

### 문제

![img](https://user-images.githubusercontent.com/45479309/191879358-5ca6513d-6c1b-4419-95f4-aa74e22c50b5.png)

### 내풀이

```javascript
/**
 * @param {string} target
 * @return {number}
 */
var minFlips = function(target) {
    let count = 0;
    let prevValue = '0';
    for(let idx = 0; idx < target.length; idx++) {
        if(target[idx] !== prevValue) {
            count += 1;
            prevValue = target[idx];
        }
    }
    return count;
};
```

"00000" 에서 주어진 스트링형태의 target 값이 되도록 하기 위해선 target 문자를 계속 순회하면서 이전 값이랑 다른지 비교하면서 다르면 뒤집어주는 형식으로 접근하였다.

예를들어, 10111이 target이라고 할 때,  


|   idx|   prevValue|   count|
|---|---|---|
|   0|   0|   1|
|   1|   0|   2|
|   2|   1|   3|
|   3|   1|   3|
|   4|   1|   3|  

위 표와 같이 target 문자가 되기 위해 prevValue를 계속 업데이트 시켜주면서 count값을 증가시켜주면 target 문자가 되기 위한 최솟값을 구할 수가 있다.
