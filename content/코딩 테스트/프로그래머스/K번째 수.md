---
title: K번째 수
date: 2025-02-21
tags:
  - 알고리즘
  - 코딩테스트
  - Cpp
description: K번째 수
---
[문제 링크](https://school.programmers.co.kr/learn/courses/30/lessons/42748)  
난이도 : Lv. 1

# 문제 내용

배열 array의 i번째 숫자부터 j번째 숫자까지 자르고 정렬했을 때, k번째에 있는 수를 구하려 합니다.

예를 들어 array가 \[1, 5, 2, 6, 3, 7, 4\], i = 2, j = 5, k = 3이라면

1. array의 2번째부터 5번째까지 자르면 \[5, 2, 6, 3\]입니다.
2. 1에서 나온 배열을 정렬하면 \[2, 3, 5, 6\]입니다.
3. 2에서 나온 배열의 3번째 숫자는 5입니다.

배열 array, \[i, j, k\]를 원소로 가진 2차원 배열 commands가 매개변수로 주어질 때, commands의 모든 원소에 대해 앞서 설명한 연산을 적용했을 때 나온 결과를 배열에 담아 return 하도록 solution 함수를 작성해주세요.

## 입력

\[1, 5, 2, 6, 3, 7, 4\], \[\[2, 5, 3], \[4, 4, 1], \[1, 7, 3]]

## 출력

\[5, 6, 3]

# 문제 분석

commands 크기만큼 int array 하나 만들기

commands 크기만큼 밑에 반복

1. j - i + 1 크기만큼 array 하나 만듬
2. 그 array에 값 복사함
3. sort
4. array에서 k - 1번째 있는걸 커맨드 크기의 배열에 넣음

# 작성한 코드

```cpp
#include <bits/stdc++.h>

using namespace std;

vector<int> solution(vector<int> array, vector<vector<int>> commands) {
    vector<int> answer;
    for(vector<int> one : commands) {
        vector<int> new_array;
        for(int i = 0; i < one[1] - one[0] + 1; i++) {
            new_array.push_back(array[one[0] - 1 + i]);
        }
        //for(auto one : new_array) cout << one << " ";
        //cout << endl;
        sort(new_array.begin(), new_array.end());
        answer.push_back(new_array[one[2] - 1]);
    }
    return answer;
}
```

중간에 segmentation fault 뜨길래 봤더니.. vector에 값도 없는데 \[]로 접근해서 그렇다네..

push_back 잊지말자.. 동적할당이라고 \[]로 바로 갈수있는거아니다…

# 우수 코드 분석

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> solution(vector<int> array, vector<vector<int>> commands) {
    vector<int> answer;
    vector<int> temp;

    for(int i = 0; i < commands.size(); i++) {
        temp = array;
        sort(temp.begin() + commands[i][0] - 1, temp.begin() + commands[i][1]);
        answer.push_back(temp[commands[i][0] + commands[i][2]-2]);
    }

    return answer;
}
```

몰랐는데 vector 깊은복사 되는듯?

저기서 좀만 더 손보면 지정한 부분만 깊은복사로 가져오면 될듯

