---
permalink: /leetcode/Number of Provinces
title: "Number of Provinces"
categories: Leetcode
tags: DFS

toc: true
toc_sticky: true


date: 2022-10-16
last_modified_at: 2020-10-16
---

## Number of Provinces

[Number of Provinces 풀기](https://leetcode.com/problems/number-of-provinces/)

### 문제

<img width="1247" alt="547_Number of Provinces" src="https://user-images.githubusercontent.com/45479309/196018413-5ee005e4-2fb3-40a8-aba7-0b747b26f4af.png">

### 내풀이


```javascript
/**
 * @param {number[][]} isConnected
 * @return {number}
 */

var findCircleNum = function(isConnected) {
    const len = isConnected.length;
    const graph = Array.from({length: len + 1}, () => []);
    const visited = new Array(len + 1).fill(false);
    let provinces = 0;
    
    // 그래프 연결 하기.
    isConnected.forEach((edge, index) => {
        for(let i = 0; i < edge.length; i++) {
            if(i === index) continue; // 자기 자신 정점을 (셀프) 의미하므로 pass.
            
            if(edge[i]) graph[index + 1].push(i + 1);
        }
    });

    const dfs = (node) => {
        visited[node] = true;

        for(let i = 0; i < graph[node].length; i++) {
            if(!visited[graph[node][i]]) {
                dfs(graph[node][i]);
            }
        }
    }

    for(let i = 1; i <= len; i++) {
        if(!visited[i]) {
            provinces += 1;
            dfs(i);
        }
    }
    return provinces;
};

findCircleNum([[1,1,0],[1,1,0],[0,0,1]]);
```

**`주어진 Input 값들`의 정점과 간선을 그래프로 표현하는게 다른 문제와달라 약간 까다로웠다.**

1. 우선 주어진 2차원배열 안에 있는 정점을 연결하는 그래프를 만들자.

>> 위 예제에서 저장 이후에 찍어보면 => [ [], [ 2 ], [ 1 ], [] ]와 같은 형태로 저장된다.

2. 방문 체크를 위해 정점의 개수만큼 visited 배열 선언

3. 1번 정점노드부터 `각 정점에 연결된 노드들`은 count 1로 처리해야 하기 때문에 DFS 탐색.

4. !visited[node] 조건으로 방문하지 않는 정점일때만 dfs 재귀 호출