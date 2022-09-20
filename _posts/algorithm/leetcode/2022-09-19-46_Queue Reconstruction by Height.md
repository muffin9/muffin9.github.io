---
permalink: /leetcode/Queue Reconstruction by Height
title: "Queue Reconstruction by Height"
categories: Leetcode
tags: Bruteforce

toc: true
toc_sticky: true

date: 2022-09-20
last_modified_at: 2020-09-20
---

## Queue Reconstruction by Height

https://leetcode.com/problems/queue-reconstruction-by-height/

### 문제

![image](https://user-images.githubusercontent.com/45479309/191151699-5af335f5-9170-4f78-a11c-430e06c49956.png)

### 내풀이

```javascript
/**
 * @param {number[][]} people
 * @return {number[][]}
 */
var reconstructQueue = function(people) {
    people.sort((a, b) => a[0] === b[0] ? a[1] - b[1] : b[0] - a[0]);
    let result = [];

    for(const [key, value] of people) {
       result = [...result.slice(0, value), [key, value], ...result.slice(value)];
    }
    return result;
};
```

지문이 영어로 된건지.. 문제 이해하는데만 시간을 많이 소요했다.. 어떻게 이해를 하고 문제를 풀긴 했는데 메모리 측면에선 그렇게 좋진 않아 보이지만.. 내가 설계한 풀이과정은 다음과 같다.

1. 먼저 키에 대하여 내림차순을 하자. 만약, 키가 동일하다면? => 앞 사람들 중 자신의 키 이상 사람수가 적을 수록 앞쪽에 위치  

**k가 작은 숫자를 먼저 넣어야 같은 숫자를 포함하면서 k조건을 만족할 수 있다. (즉 h 기준으로 내림차순으로 하지만 h값이 같다면 k값을 기준으로 오름차순 정렬.)**

2. 다시 정렬된 2차원 배열 people을 순회하며 value(ki)값을 기준으로 value(ki)보다 작은 녀석들을 먼저 셋팅하고, 자기 자신(hi, ki)을 넣고, value(ki) 보다 큰 녀석들을 뒤에 다시 배치하도록 전개연산자를 이용하여 계속 result를 업데이트 시켜주었다.

