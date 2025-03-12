---
title: 정수 N개의 합
date: 2025-01-08
tags:
  - 알고리즘
  - 코딩테스트
  - Cpp
description: 정수 N개의 합
---
[문제 링크]()  
난이도 : 브론즈 2

# 문제 내용

정수 n개가 주어졌을 때, n개의 합을 구하는 함수를 작성하시오.

작성해야 하는 함수는 다음과 같다.

`a`: 합을 구해야 하는 정수 `n`개가 저장되어 있는 배열 (0 ≤ a\[i\] ≤ 1,000,000, 1 ≤ n ≤ 3,000,000)

리턴값: `a`에 포함되어 있는 정수 `n`개의 합


# 문제 분석

C++11로 작성하기 때문에,

long long sum(vector\<int\> &a)를 구현하면 됨.

list 말고 벡터로 받아오니까 그걸 주의해야 할 듯.

참고자료 : [https://hwan-shell.tistory.com/119](https://hwan-shell.tistory.com/119)

벡터 : 힙에 생성됨. 동적할당 ← 이점이 일반 배열과 다른 듯!

벡터의 요소에 접근하려면 at(i), \[i\]가 있지만 범위 검사가 없어 속도가 빠른 \[i\]의 사용이 더 많다고 함

size ≠ capacity!!

size : 벡터 안에 데이터가 들어간 공간 개수.

capacity : 힙에 할당된 벡터의 최대 크기!!

+내가 아는 자료형은 long인데, long long은 대체 무엇인가 찾아봄.

놀랍게도 C++에서 long이랑 int는 자료형 크기와 범위가 같다?????

내가 아는 long이 C++에서는 long long이었던 것….

# 작성한 코드

```cpp
#include <vector>
long long sum(std::vector<int> &a)
{
    long long ans = 0;
    // 코드 작성 시작
    for(int i = 0; i < a.size(); i++) {
        ans += a[i];
    }
    // 코드 작성 끝
    return ans;
}
```

# 우수 코드 분석

```cpp
#include <vector>
long long sum(std::vector<int> &a) {
	long long ans = 0;
    for (auto v : a) {
      ans += v;
    }
	return ans;
}
```

우수 코드를 보니, auto 라는 접두사를 붙였다. 이게 뭐지?

참고자료 : [https://dydtjr1128.github.io/cpp/2019/06/04/Cpp-auto.html](https://dydtjr1128.github.io/cpp/2019/06/04/Cpp-auto.html)

C++11 버전부터 적용됨!!

형식이 추론되는 변수를 선언!!

auto를 사용할 수 없는 경우

1. 함수의 매개변수 ← 그렇지만 반환형으로는 가능함!!! 대신 후행 반환 형식을 지정해줘야함.
    - ex. auto A()→double{ return 3.2; } 근데 이대로 쓰면 그냥 double A() 와 같음.
    - 따라서, 템플릿과 함께 사용하면 유용함.
2. 구조, 클래스의 멤버변수

그래서 for문에서 auto를 어떻게 쓰는건가?

참고자료 : [https://boycoding.tistory.com/210](https://boycoding.tistory.com/210)

마침 이 범위 기반 for문도 C++11 버전에서 추가된 내용!

for (자료형 변수명 : 배열) ← 이렇게 작성하면, 배열을 순회하면서 변수명에 하나씩 들어감.

파이썬과 비슷한듯?

대신 배열의 자료형과 for문에서 선언한 변수의 자료형이 일치해야 함. 그러지 않으면 형 변환이 일어난다고 함.

따라서, 위의 우수 코드는 auto와 범위 기반 for문을 함께 사용한 예시인 것으로 보임.

그럼 왜 그게 내 코드보다 빨랐을까?

참고자료 : [https://blog.naver.com/remocon33/221701119330](https://blog.naver.com/remocon33/221701119330)

인덱스 접근 for문보다 범위 기반 for문이 벡터에서 더 빠르다는 내용을 보임.

내 코드는 인덱스 접근 for문이어서 아마 4ms가 나온 것 같음.

