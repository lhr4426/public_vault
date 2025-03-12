---
title: C++ 공부 내용
date: 2025-03-12
tags:
  - Cpp
description: 코테 공부하면서 알게된거
---
코테 공부하면서 C++ 관련으로 알게된 내용들

---

- scanf, printf가 cin, cout보다 빠르다. (c/c++ 동기화 풀지 않는다는 전제 하에)  
- 한 줄째로 읽어와야하는 경우에는 cin 쓰면서 아래 내용 추가하기. c/c++ 동기화 해제하면서 빨라짐      
    ```cpp
    ios_base::sync_with_stdio(false);
    cin.tie(0);
    
    getline(cin, string, '구분자');
    ```

- getline 쓰기 전에 따로 또 입력받는거 있으면 cin.ignore()로 버퍼 한 번 비워주기  
- strtok는 cstring에 있음  
- atoi는 stdlib.h에 있음  
- 범위 기반 for문과 auto를 사용하면 반복을 더 빠르게 할 수 있다. → for(auto data : array)  
- 수 * bool 가능함. 이걸로 특정 조건에 맞을 때만 더하게 한다던가 할 수 있음!  
- printf(”\n”) 보다 puts(””)가 더 빠르다! (puts는 무조건 문자열만 출력하기 때문에 형식 생각 안해도 돼서 그런 듯.)  
- scanf()의 반환값은 데이터를 저장하는데 성공한 변수의 수이다. 이걸 반복문의 조건문으로 쓰면 입력값이 없을 때 까지 반복하게 할 수 있음.  
    - ex. 크냐? 문제 우수 코드 : for(;scanf(”%d%d”, &a, &b);puts(a>b?”Yes”:”No”))  
- 대문자의 소문자 화 : +32, 소문자의 대문자 화 : -32  
- 알파벳 대문자 : 65에서 90까지  
- 알파벳 소문자 : 97에서 122까지  
- string 헤더파일에 stoi, to_string 있으니까 굳이 stdlib.h 찾지말고 string에서 쓰자  
- 홀짝 연산으로 &1 써먹으면 좋음  
- **#include <bits/stdc++.h> ← 이 헤더파일 쓰면 표준 라이브러리 대부분 쓸 수 있음**  
- accumulate 함수 잘 쓰면 유용함  
```cpp
accumulate(배열_컨테이너의_시작지점, 배열_컨테이너의_끝지점, 초기값);
accumulate(배열_컨테이너의_시작지점, 배열_컨테이너의_끝지점, 초기값, 커스텀함수);

// 커스텀함수는 첫 번째 인자로 지금까지의 누적 합, 두 번째 인자로 이번에 더할 값을 받는다

// 사용 예시 1. 모든 수를 더하기
accumulate(nums.begin(), nums.end(), 0);

// 사용 예시 2. 모든 수를 빼기
accumulate(nums.begin(), nums.end(), 0, [](int x, int y){return x - y;});
// 여기서는 커스텀함수를 사용해 누적 값인 x에서 이번 값인 y를 빼는 작업을 반복함

// 사용 예시 3. 홀수 인덱스에 있는 값만 더하기
accumulate(nums.begin(), nums.end(), 0, [&, index = 0](int x, int y) mutable {
	return (index++%2==1)? x + y : x;}
```

- 반복자에서 end()는 값이 없단 뜻이다!!! 반복자는 \[ \) 형식이다!!! begin은 포함되지만 end는 포함되지 않음을 잊지 말기!!!  
- 반복자에서 * 써도 포인터 \*처럼 값 찾아낼 수 있음  
- max_element(vector.begin(), vector.end()) : 최대값의 반복자가 나옴  
- string.substr(pos, count) : 부분 문자열 추출. pos부터 count 개 사용  
- string.push_back은 char 한 글자밖에 못들어감  
- 문자열끼리 이으려면 string.append() 하던가 += 사용  
- find(탐색 문자열, 인덱스 = 0) : 탐색 문자열을 앞에서부터 검색함  
- rfind(탐색 문자열, 인덱스 =npos) : 탐색 문자열을 뒤에서부터 검색함  