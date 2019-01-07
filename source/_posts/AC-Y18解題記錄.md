---
title: AC-Y18解題記錄
date: 2018-02-12 14:05:56
tags:
  - Algorithm
---

演算法解題記錄

<!--more-->

## 00

Write a function:

  `function solution(A);`

that, given an array A of N integers, returns the smallest positive integer (greater than 0) that does not occur in A.

For example, given A = [1, 3, 6, 4, 1, 2], the function should return 5.

Given A = [1, 2, 3], the function should return 4.
Given A = [-1, -3], the function should return 1.

Assume that:

* N is an integer within the range [1..100,000];
* each element of array A is an integer within the range [-1,000,000..1,000,000].

Complexity:

* expected worst-case time complexity is O(N);
* expected worst-case space complexity is O(N), beyond input storage (not counting the storage required for input arguments).

---

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
---

## 01

An interval is a pair (A, B) of integers such that A <= B. Two intervals (A, B) and (C, D) overlap if there exists an integer L such that A <= L <= B and C <= L <= D. Intervals that do not overlap are called disjoint. The union of intervals (A, B) and (C, D) is defined as:

* either a single interval (min(A, C), max(B, D)), when (A, B) and (C, D) overlap, or
* the intervals (A, B) and (C, D) themselves, when they are disjoint.

Taking the union of two intervals is a commutative and associative operation, so it
can be extended to an arbitrary number of intervals.

For example, consider the following eight intervals:

```Text
    ( 1,   5)       (12,   15)        (42,   44)
    (70,  72)       (36,   36)        (-4,    2)
    (43,  69)       (15,   24)
```

Intervals (1, 5) and (-4, 2) overlap and their union is (-4, 5). Intervals (12, 15) and (15, 24) overlap and their union is (12, 24). Intervals (42, 44) and (43, 69) overlap and their union is (42, 69). Intervals (70, 72) and (36, 36) are disjoint and do not overlap with other intervals. The union of all eight intervals consists of the following five pairwise disjoint intervals:

```Text
    (-4,  5)        (12, 24)         (42, 69)
    (70, 72)        (36, 36)
```

Write a function:

  `function solution(A, B);`

that, given two zero-indexed arrays A and B consisting of N elements each, returns the number of pairwise disjoint intervals constituting the union of N intervals described by arrays A and B. The K-th interval, where K is an integer within the range [0..(N - 1)], is defined as (A[K], B[K]).

For example, given the following arrays A and B consisting of eight elements each:

```Text
    A[0] =  1    A[1] = 12    A[2] = 42
    A[3] = 70    A[4] = 36    A[5] = -4
    A[6] = 43    A[7] = 15
```

```Text
    B[0] =  5    B[1] = 15    B[2] = 44
    B[3] = 72    B[4] = 36    B[5] = 2
    B[6] = 69    B[7] = 24
```

the function should return 5, because the intervals described by these arrays correspond to the example above. Assume that:

* N is an integer within the range [0..100,000];
* each element of arrays A, B is an integer within the range [-1,000,000,000..1,000,000,000];
* A[K] <= B[K] for integers K within the range [0..(N - 1)].

Complexity:

* expected worst-case time complexity is O(N*log(N));
* expected worst-case space complexity is O(N), beyond input storage (not counting the storage required for input arguments).

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
---

## 03

A chessboard consisting of N rows and M columns is given. Each square of the board is either empty or blocked.

A knight is a chess piece that perform the following moves in one turn:

* two squares up and one square left;
* two squares up and one square right;
* one square up and two squares left;
* one square up and two squares right;
* one square down and two squares left;
* one square down and two squares right;
* two squares down and one square left;
* two squares down and one square right.

The number of possible moves can be reduced if the knight would fall off the board or if the destination square is blocked.

The board can be described by a zero-indexed matrix consisting of N rows and M columns of integers. A square of the board is empty if its corresponding matrix element has value 0 and blocked if its corresponding matrix element has value 1.

For example, consider the following matrix A consisting of four rows and three columns:

```Text
    A[0][0] = 0    A[0][1] = 0    A[0][2] = 0
    A[1][0] = 0    A[1][1] = 0    A[1][2] = 1
    A[2][0] = 1    A[2][1] = 0    A[2][2] = 0
    A[3][0] = 0    A[3][1] = 0    A[3][2] = 0
```

Consider a knight standing on the upper-left square. It requires seven turns to move to the lower-right square:

* in the first turn the knight moves from square (0, 0) to square (2, 1);
* in the second turn the knight moves from square (2, 1) to square (0, 2);
* in the third turn the knight moves from square (0, 2) to square (1, 0);
* in the fourth turn the knight moves from square (1, 0) to square (2, 2);
* in the fifth turn the knight moves from square (2, 2) to square (3, 0);
* in the sixth turn the knight moves from square (3, 0) to square (1, 1);
* in the seventh turn the knight moves from square (1, 1) to square (3, 2).

Note that a shorter path of length three exists, but the knight cannot take it because some of the squares along this path are blocked.

Write a function:

  `function solution(A);`

that, given a zero-indexed matrix A consisting of N rows and M columns describing chessboard, returns the minimum number of turns that the knight requires to move from the upper-left square to the lower-right square. The function should return -1 if it is impossible for the knight to move from the upper-left square to the lower-right square.

Assume that:

* N and M are integers within the range [1..1,000,000];
* the number of elements in matrix A is within the range [1..1,000,000];
* each element of matrix A is an integer that can have one of the following values: 0, 1;
* A[0][0] = A[N-1][M-1] = 0

For example, given matrix A consisting of four rows and three columns such that:

```Text
    A[0][0] = 0    A[0][1] = 0    A[0][2] = 0
    A[1][0] = 0    A[1][1] = 0    A[1][2] = 1
    A[2][0] = 1    A[2][1] = 0    A[2][2] = 0
    A[3][0] = 0    A[3][1] = 0    A[3][2] = 0
```

the function should return 7, as explained in the example above.

Complexity:

* expected worst-case time complexity is O(N*M);
* expected worst-case space complexity is O(N*M).

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
