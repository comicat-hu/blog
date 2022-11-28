---
title: AC-Y18解題記錄
date: 2018-02-12 14:05:56
categories:
  - 資訊技術
  - Algorithm
tags:
  - Algorithm
---

演算法解題記錄

<!--more-->

## 00

### 00 JS參考解

* time: O(N), space: O(1)
* 使用原輸入陣列來標記(index = value - 1)

```JS
function solution(A) {
    for (let i = 0; i < A.length; i++) {
        if (A[i] > 0 && A[i] <= A.length && A[i] !== A[A[i]-1]) {
            // swap
            let temp = A[A[i]-1];
            A[A[i]-1] = A[i];
            A[i] = temp;
            i--;
        }
    }
    for(let i = 0; i < A.length; i++) {
        if(A[i] != i + 1) {
            return i + 1;
        }
    }
    return A.length + 1;
}

console.log(solution([1, 3, 6, 4, -1, 2])) // 5
console.log(solution([1, 2, 3])) // 4
console.log(solution([-1, -3])) // 1

```

* Ref: <https://leetcode.com/problems/first-missing-positive/discuss/17071/My-short-c++-solution-O(1)-space-and-O(n)-time>

---

### 01 JS參考解

* 依區間起始值，由小而大排序
* 排完之後會有個特性: 若第n的區間和n-1的區間不相交，那麼n+1的區間和第n-1的區間也不相交
* 若目前stack top的區間和當前區間不相交，就直接push
* 若目前stack top的區間和當前區間相交(只需考慮區間結束值)，就合併區間

```JS
function Interval(x, y) {
    this.x = x;
    this.y = y;
}

function solution(A, B) {
    var intervals = [];
    for (let i = 0; i < A.length; i++) {
        intervals.push(new Interval(A[i], B[i]));
    }
    intervals.sort((intervalA, intervalB) => intervalA.x - intervalB.x);

    var stack = [];
    stack.push(intervals[0]);

    for (let i = 1, j = 0; i < intervals.length; i++) {
        if (intervals[i].x > stack[j].y) {
            stack.push(intervals[i]);
            j++;
        } else if (intervals[i].y > stack[j].y) {
            stack[j].y = intervals[i].y;
        }
    }

    return stack.length;
}

var A = [1, 12, 42, 70, 36, -4, 43, 15];
var B = [5, 15, 44, 72, 36, 2, 69, 24];

console.log(solution(A, B)); // 5
```

Ref: <https://www.geeksforgeeks.org/merging-intervals/>

---

## 03 JS參考解

* BFS，優先走到終點的就是最短路徑了
* 用queue存每步的所有情況

```text
function initRecord(N, M) {
    let arr = [];
    for (let i = 0; i < N; i++) {
        arr.push([]);
        for (let j = 0; j < M; j++) {
            arr[i].push(false);
        }
    }
    arr[0][0] = true;
    return arr;
}

function Move(x, y, step) {
    this.x = x;
    this.y = y;
    this.step = step;
}

function solution(A) {
    const N = A.length;
    const M = A[0].length;
    let record = initRecord(N, M);
    let dir = [
        [2, -1],
        [2, 1],
        [1, -2],
        [1, 2],
        [-1, -2],
        [-1, 2],
        [-2, -1],
        [-2, 1]
    ];
    let queue = [new Move(0, 0, 0)];
    while (queue.length > 0) {
        let now = queue.shift();
        if (now.x == N - 1 && now.y == M - 1) {
            return now.step;
        }
        for (let i = 0; i < dir.length; i++) {
            let mx = dir[i][0] + now.x;
            let my = dir[i][1] + now.y;
            if (mx >= 0 && mx < N && my >= 0 && my < M && (A[mx][my] != 1) && (record[mx][my] == false)) {
                record[mx][my] = true;
                queue.push(new Move(mx, my, now.step + 1));
            }
        }
    }
    return -1;
}

var A = [
    [0, 0, 0],
    [0, 0, 1],
    [1, 0, 0],
    [0, 0, 0],
];
console.log(solution(A)); // 7
```

Ref: <https://www.geeksforgeeks.org/minimum-steps-reach-target-knight/>
