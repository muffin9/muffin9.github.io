---
permalink: /leetcode/Maximum Product of Two Elements in an Array
title: "Maximum Product of Two Elements in an Array"
categories: Leetcode
tags: Heap Sort

toc: true
toc_sticky: true


date: 2022-10-06
last_modified_at: 2020-10-06
---

## Maximum Product of Two Elements in an Array

https://leetcode.com/problems/maximum-product-of-two-elements-in-an-array/

### 문제

![img](https://user-images.githubusercontent.com/45479309/194117326-b5e22441-f7cf-430d-a14d-15d6556f9e88.png)

### 내풀이 (주어진 nums를 바로 내림차순 정렬해서 풀이)

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var maxProduct = function(nums) {
    nums.sort((a, b) => b - a);
    return (nums[0] - 1) * (nums[1] - 1);
};
```

---

### 내풀이 (최대힙 사용)

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
 class MaxHeap {
    constructor() {
        this.heap = [];
    }

    isEmpty() {
        return this.heap.length ? false : true;
    }

    swap(idx1, idx2) {
        return [this.heap[idx1], this.heap[idx2]] = [this.heap[idx2], this.heap[idx1]];
    }

    insert(value) {
        this.heap.push(value);
        let currentIdx = this.heap.length - 1;

        while(currentIdx > 0) {
            const parentIdx = Math.floor((currentIdx) / 2);
            if(this.heap[parentIdx] >= this.heap[currentIdx]) break;
            this.swap(parentIdx, currentIdx);
            currentIdx = parentIdx;
        }
    }

    getMax() {
        if(this.heap.length === 1) {
            return this.heap.pop();
        }

        const maxValue = this.heap[0];
        this.heap[0] = this.heap.pop();
        this.heapify(0);
        return maxValue;
    }

    heapify(idx) {
        const leftIdx = idx * 2;
        const rightIdx = idx * 2 + 1;
        let biggestIdx = idx;

        if(this.heap[biggestIdx] <= this.heap[leftIdx]) biggestIdx = leftIdx;

        if(this.heap[biggestIdx] <= this.heap[rightIdx]) biggestIdx = rightIdx;

        if(biggestIdx !== idx) {
            this.swap(biggestIdx, idx);
            this.heapify(biggestIdx);
        }
    }
}

var maxProduct = function(nums) {
    const maxHeap = new MaxHeap();
    nums.forEach((num) => {
        maxHeap.insert(num);
    });
    return (maxHeap.getMax() - 1) * (maxHeap.getMax() - 1);
};
```

**내림차순으로 정렬해서 답을 도출해냈을때의 시간복잡도와 최대힙을 사용했을때의 시간복잡도 차이가 거의 없다.**

이전에 만들어논 maxHeap 보일러플레이트를 가져와서 적용했을때의 시간복잡도와 공간복잡도 비교 : 거의 차이 없다.  

제출할때마다 악간씩 다르게 나오는데.. 어차피 최대힙을 사용할때도 처음에 주어진 배열을 힙에 넣어줄때 반복문을 사용해야해서 간단한 첫 번째 방법으로 푸는게 시간절약이 될 듯 싶다.

