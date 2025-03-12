---
title: X보다 작은 수
date: 2025-01-06
tags:
  - 알고리즘
  - 코딩테스트
  - Cpp
description: X보다 작은 수
---
[문제 링크](https://www.acmicpc.net/problem/10871)  
난이도 : 브론즈 5

# 문제 내용

정수 N개로 이루어진 수열 A와 정수 X가 주어진다. 이때, A에서 X보다 작은 수를 모두 출력하는 프로그램을 작성하시오.

## 입력

첫째 줄에 N과 X가 주어진다. (1 ≤ N, X ≤ 10,000)

둘째 줄에 수열 A를 이루는 정수 N개가 주어진다. 주어지는 정수는 모두 1보다 크거나 같고, 10,000보다 작거나 같은 정수이다.

## 출력

X보다 작은 수를 입력받은 순서대로 공백으로 구분해 출력한다. X보다 작은 수는 적어도 하나 존재한다.

# 문제 분석

1. N, X 입력받고 저장
2. 크기가 N인 배열 저장
3. 순회하면서 X보다 작은 데이터 출력

# 작성한 코드

답안 1 : 배열로 저장하고, 그 배열을 순회하면서 검사

```cpp
#include <iostream>

using namespace std;

int main() {
    int n, x; // 배열 크기 n, 작은 수의 기준 x
    cin >> n >> x;

    int *numList = new int[n];
    for (int i = 0; i < n; i++) {
        cin >> numList[i];
    }

    for (int i = 0; i < n; i++) {
        if(numList[i] < x) {
            cout << numList[i] << " ";
        }
    }
}
```

답안 2 : 배열로 저장하지 않고, 바로 검사

```cpp
#include <iostream>

using namespace std;

int main() {
    int n, x; // 배열 크기 n, 작은 수의 기준 x
    cin >> n >> x;

    for (int i = 0; i < n; i++) {
        int m;
        cin >> m;
        if(m < x) {
            cout << m << " ";
        }
    }
}
```

# 발견한 점

놀랍게도 답안 2가 답안 1보다 느림!

그리고 더 빠른 속도를 낸 코드를 보니까 cin을 사용하지 않길래, 왜 그런가 확인해봤음

[https://www.acmicpc.net/blog/view/56](https://www.acmicpc.net/blog/view/56)

결론 : cin, cout보다 scanf, printf가 더 빠름.

그러나? 놀랍게도 또 다른 글을 발견함

[https://jaimemin.tistory.com/1521](https://jaimemin.tistory.com/1521)

결론 : cin, cout을 scanf, printf보다 빠르게 하는 방법이 존재함!

c와 cpp의 버퍼 동기화를 비활성화 시킴으로써 실행 속도에서의 이점을 가져오지만

멀티쓰레드를 사용할 때는 쓰면 안됨. ⇒ 코테용?

그리고 cin, cout 사용하면서 scanf, gets 등등의 C 입출력 사용하면 안된다고 함

그래서 2번 답안을 혹시나 하고 C 스타일로 다시 풀어봄.

```cpp
#include <stdio.h>
int main() {
    int n, x; // 배열 크기 n, 작은 수의 기준 x
    scanf("%d%d", &n, &x);
    for (int i = 0; i < n; i++) {
        int m;
        scanf("%d", &m);
        if(m < x) {
            printf("%d ", m);
        }
    }
}
```

시간이 0ms로 매우 우수하게 나왔음!

