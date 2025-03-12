---
title: SQL(Oracle) 공부 내용
date: 2025-03-12
tags:
  - 데이터베이스
  - Oracle
description: 코테 공부하면서 알게된거
---
코테 공부하면서 SQL(Oracle) 관련으로 알게된 내용들 

---

- 정규식 : regexp_like(컬럼, ‘조건’)  
- 두 인수 중 null이 아닌 값 출력 : nvl(값 1, 값 2)  
- from문에 as 쓰지마셈  
- datetime 형식의 컬럼에서 일부 값 추출 : extract(year(아니면 month, day…등등) from datetime컬럼)  
- 뭔가 답이 아니다 싶으면 date 컬럼 처음 표기 형식 잘보기. to_char(date컬럼, ‘YYYY-MM-DD’)로 통일해야 할 수 있음  
- 상위 n개의 레코드만 조회 : fetch first n row only < 맨 밑에 쓰기  
- 컬럼 뒤에 문자열 : concat(컬럼1, 문자열\[or 컬럼\]) 쓰거나 || 쓰기 (||가 더 쓰기 편함)  