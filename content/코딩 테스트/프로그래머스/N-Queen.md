---
title: N-Queen
date: 2025-11-26
tags:
  - 알고리즘
  - 코딩테스트
description: N-Queen
---
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/12952)  
난이도 : Lv. 2

# 문제 내용

###### 문제 설명

가로, 세로 길이가 n인 정사각형으로된 체스판이 있습니다. 체스판 위의 n개의 퀸이 서로를 공격할 수 없도록 배치하고 싶습니다.

예를 들어서 n이 4인경우 다음과 같이 퀸을 배치하면 n개의 퀸은 서로를 한번에 공격 할 수 없습니다.

![Imgur](https://i.imgur.com/lt2zdK6.png)  
![Imgur](https://i.imgur.com/5c5EUrq.png)

체스판의 가로 세로의 세로의 길이 n이 매개변수로 주어질 때, n개의 퀸이 조건에 만족 하도록 배치할 수 있는 방법의 수를 return하는 solution함수를 완성해주세요.

##### 제한사항

- 퀸(Queen)은 가로, 세로, 대각선으로 이동할 수 있습니다.
- n은 12이하의 자연수 입니다.

---

##### 입출력 예

|n|result|
|---|---|
|4|2|

##### 입출력 예 설명

입출력 예 #1  
문제의 예시와 같습니다.

# 문제 분석

퀸과 퀸이 가로/세로/대각선 에 만나면 안됨.
# 작성한 코드

```cpp
#include <string>
#include <vector>
#include <cmath>

using namespace std;

bool isSameRow(int newRow, int currentRow)
{
    return newRow == currentRow;
}

bool isDiagonal(int newRow, int newCol, int currentRow, int currentCol)
{
    return abs(newRow - currentRow) == abs(newCol - currentCol);
}

bool isSafe(vector<int> &queens, int newRow, int newCol)
{
    for(int i = 0; i < newCol; i++)
    {
        if(isSameRow(newRow, queens[i]) || isDiagonal(newRow, newCol, queens[i], i))
            return false;
    }
    return true;
}

int placeQueens(vector<int> &queens, int col)
{
    if(queens.size() == col)
        return 1;
    
    int count = 0;
    for(int row = 0; row < queens.size(); row++)
    {
        if(isSafe(queens, row, col))
        {
            queens[col] = row;
            count += placeQueens(queens, col+1);
            queens[col] = -1;
        }
    }
    
    return count;
}

int solution(int n) {
    vector<int> queens(n, -1);
    
    return placeQueens(queens, 0);
}
```

- 매 행마다 어디 열에 둘지 검사함