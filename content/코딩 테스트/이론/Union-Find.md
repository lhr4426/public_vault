---
title: Union-Find
date: 2025-03-09
tags:
  - 자료구조
  - 알고리즘
description: Union-Find의 정의와 예시
---
# 용어 정의

트리를 활용해서 집합을 관리하는 구조  
요소 간 관련이 있는지 없는지 판단할 때 쓰기 좋다.

# 구현


```c++
struct UnionFind {
	// subSize : 본인을 포함한 서브트리의 꼭짓점 갯수
	vector<int> parent, subSize; 
	UnionFind(int N) : parent(N, -1), subSize(N, 1);

	int root(int x) {
		if(parent[x] == -1) return x;
		else return parent[x] = root(parent[x]);
	}

	// x와 y가 같은 트리 내에 존재하는지
	bool IsSame(int x, int y){ 
		return root(x) == root(y);
	}

	// x가 속한 트리와 y가 속한 트리를 합침
	bool Unite(int x, int y){ 
		x = root(x);
		y = root(y);
		if(x == y) return false;
		if(subSize[x] < subSize[y]) swap(x, y);
		parent[y] = x;
		subSize[x] += subSize[y];
		return true;
	}
	int size(int x){
		return subSize[root(x)];
	}
}
```

# 참고자료

[문제 해결력을 높이는 알고리즘과 자료 구조](https://www.yes24.com/Product/Goods/107514892)