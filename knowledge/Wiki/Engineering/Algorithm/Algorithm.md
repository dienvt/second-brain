---
title: "Algorithm"
date: 2026-01-14
tags:
  - engineering
  - algorithm
---

[[BigO notation]]

[[Sort algorithm]]

## **1886. Determine Whether Matrix Can Be Obtained By Rotation**

try to list then convert to int

```
// 90 degree
mat[i][j] = mat[j][n-i-1]

// 180 degree
mat[i][j] = mat[n-j-1][i]

// 270 degree
mat[i][j] = mat[n-i-1][n-j-1]
```