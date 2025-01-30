---
title: 김치런 만들기 (3)
date: 2025-01-15
tags:
  - Unity
description: Unity 6 Challenge
---
[유니티 6 챌린지 기록](./index.md)

# 장애물 파트 1

- 카메라 오른쪽 끝에 빌딩을 만들고, 카메라 왼쪽 끝으로 스크롤이 끝나면 해당 빌딩을 삭제하고자 함 ⇒ 보이지 않는 객체가 메모리를 차지하지 않게 하기 위함!!
    - 적, 음식들도 똑같이!
- 빌딩은 높이가 제각각이라, 기준점(앵커)을 바닥으로 통일해야 함
    - 5개의 빌딩 모두 선택
    - Sprite Mode : Single로 변경
    - Pivot : Bottom으로 변경
- 총 3 개의 스크립트 제작 예정
    - 오브젝트를 생성하는 스크립트 : Spawner
    - 오브젝트를 오른쪽 끝에서 왼쪽 끝으로 이동시키는 스크립트 : Mover
    - 오브젝트를 없애는 스크립트 : Destroyer

## Mover : 오브젝트 이동

- 움직이는 속도를 위한 변수 moveSpeed 생성
- Transform은 모든 유니티 게임오브젝트에 기본적으로 있기 때문에, 코드에서 빠르게 부를 수 있음. transform으로!
- 그리고 new Vector3() 할 필요 없이, 좌우상하 기본적인 벡터는 제공됨. ex. Vector3.left
- 이를 이용하면 한 줄로 이동 코드를 작성할 수 있음

```csharp
public class Mover : MonoBehaviour
{
    [Header("Settings")]
    public float moveSpeed;
    
    void Update()
    {
        transform.position += Vector3.left * moveSpeed * Time.deltaTime;
    }
}
```

- 이대로 바로 실행하면 건물이 플레이어를 가리기 때문에, Z축 값을 설정함으로써 이를 해결함
    - 플레이어 Z : -10
    - 메인 카메라 Z : -50
- 추후에 모든 오브젝트의 이동 속도를 동기화 할 예정

## Destroyer : 오브젝트 삭제

- 적당히 조절해 보면서 카메라의 왼쪽 끝에 가서 사라지는 x값을 찾음
- 나같은 경우에는 -11.5 정도면 이미 밖에 있어서, 대강 -12를 기준값으로 함
- transform을 바로 부를 수 있는 것처럼, 해당 스크립트가 붙어있는 게임 오브젝트를 바로 부를 수 있음 ⇒ gameObject!
- Destroy()함수는 지정한 게임 오브젝트를 파괴함
- 이를 이용해서 삭제 코드를 작성함

```csharp
void Update()
    {
        if(transform.position.x < -12){ // 이 오브젝트의 transform 중 위치의 x가 -12보다 작으면
            Destroy(gameObject); // 이 오브젝트를 파괴한다
        }
    }
```


![](../image/image%205.png)

- 제대로 삭제된 것을 확인할 수 있음!

## 프리팹화 하기

- Asset → Prefabs 폴더 생성
- Prefab : 미리 조립된 게임 객체
- 위에서 만든 Mover와 Destroyer가 포함된 빌딩 게임 오브젝트를 Prefabs 폴더에 드래그&드롭하면 프리팹이 됨
- 그러면 동일한 설정값을 가진 복사본을 여럿 만들 수 있음
- ctrl+D 사용해서 복사본을 총 4개 만든 다음, 빌딩 스프라이트만 변경하기

## Spawner : 오브젝트 생성

- 위에서 미리 만들어 둔 5개의 프리팹을 10초마다 랜덤으로 하나씩 소환하게 할 것임
- 이를 나중에 적, 음식에도 적용할 것임
- 소환할 오브젝트를 미리 참조하도록 리스트로 만듬

```csharp
...
		[Header("References")]
    public GameObject[] gameObjects;
...
```

- 기존에 씬에 있던 빌딩을 지우고, Building Spawner라는 빈 게임 오브젝트 객체를 생성함
    
    - 위치는 빌딩이 생겼으면 하는 위치로 이동함
    - 여기에 Spawner 스크립트를 붙임
    - 그리고 아래처럼, gameObjects에 미리 만들어둔 프리팹을 하나씩 추가함

	![](../image/image%206.png)

    - 주의 : 모든 프리팹의 포지션을 0, 0, 0으로 해야 함. 왜냐면 Building Spawner가 빌딩을 하나씩 만들 때, 원점 기준이 아닌 Spawner의 위치를 기준으로 하기 때문임
    - 만약에 어떤 프리팹이 2, 0, 0이면 생성될 때 스포너의 위치가 아니고 스포너의 위치 + (2, 0, 0)인 상태로 생성될 것임
- 객체 리스트 중 랜덤으로 선택하고 생성하는 코드를 작성할 것
    

```csharp
...
		void Start()
    {
        Invoke("Spawn", 1f); // 게임이 시작 된 후 1초 뒤에 Spawn 함수를 실행함 
    }

    void Spawn()
    {
        var randomObject = gameObjects[Random.Range(0, gameObjects.Length)]; // Random.Range를 통해 0부터 gameObjects 개수 - 1까지의 수 중 하나를 뽑음 => 랜덤으로 하나를 고름!
        Instantiate(randomObject, transform.position, Quaternion.identity); // Instantiate는 인자로 생성 할 오브젝트, 위치, 회전값을 받음. 이 때 Quaternion.identity는 회전하지 않은 상태를 의미
        Invoke("Spawn", 2f); // 2초 후에 Spawn 함수를 실행함 => 무한 반복!
    }
```


![](../image/image%207.png)

- 성공!
- 다만 Spawner는 건물 말고도 적과 아이템과 같은 다른 카테고리의 오브젝트를 생성하기 때문에, 각 카테고리에 맞는 생성 속도를 커스텀 할 수 있도록 조정해 주어야 함.

```csharp
...
		[Header("Settings")]
    public float minSpawnDelay; // 오브젝트가 스폰되기 위해 최소한으로 기다려야 하는 시간
    public float maxSpawnDelay; // 오브젝트가 스폰되기 위해 최대한으로 기다려야 하는 시간
...
		void Start()
    {
        Invoke("Spawn", Random.Range(minSpawnDelay, maxSpawnDelay)); // 랜덤하게!
    }
    
    void Spawn()
    {
    ...
        Invoke("Spawn", Random.Range(minSpawnDelay, maxSpawnDelay)); // 랜덤하게! 22
		}
```

![](../image/image%208.png)

- 성공!

# 장애물 파트 2

## 적 만들기 : 배달기사

- Asset → Sprites → delivery 스프라이트 내의 애니메이션 세 개 선택 후 하이어라키 창으로 드래그&드롭 → Asset → Animation → Motorbike로 저장
- 새로 만들어진 게임 오브젝트 명을 Delivery Enemy로 바꾸고, Mover와 Destroyer 컴포넌트를 더해줌
- 플레이어와 배달기사가 부딪혔는지 알아야 하기 때문에 콜라이더를 추가해야함
- 박스 콜라이더를 써버리면 배달아저씨가 없는 부분에도 닿아버림
- 캡슐 콜라이더를 쓰면 수직/수평 방향으로 조절할 수 있음
- 컴포짓 콜라이더를 쓰면 콜라이더를 합치거나, 겹치는 부분만 사용하거나 등 다양하게 변형할 수 있음!
- 폴리곤 콜라이더를 쓰면 기존의 박스와 캡슐보다 자세하게 설정할 수 있음

![](../image/image%209.png)

- 이대로 실행해 보면 배달기사가 플레이어를 밀어버림

![](../image/image%2010.png)

- 우리가 원하는 건 충돌만 감지하되, 플레이어를 밀어내지 않았으면 함
    - 콜라이더가 플레이어의 rigidbody에 물리적 영향을 주지 않길 바람
    - ⇒ 배달기사의 콜라이더에서 Is Trigger를 체크하면 됨!
    - 이에 대한 추가 자료 검색해봄 [https://ariel1910.tistory.com/entry/유니티-4-콜라이더collider-트리거](https://ariel1910.tistory.com/entry/%EC%9C%A0%EB%8B%88%ED%8B%B0-4-%EC%BD%9C%EB%9D%BC%EC%9D%B4%EB%8D%94collider-%ED%8A%B8%EB%A6%AC%EA%B1%B0)
- 배달기사를 prefab으로 만들면 배달기사는 끝

## 적 만들기 : 할머니

- 할머니도 같은 방법으로 만들면 됨
- 나는 할머니의 지팡이를 건드는 것도 용서할 수 없기 때문에 폴리곤 콜라이더로 섬세하게 설정할 것임

![](../image/image%2011.png)

- 이후에는 똑같이 콜라이더의 Is Trigger를 설정하고, mover / destroyer를 추가한 뒤 트랜스폼을 0, 0, 0으로 하고 프리팹화 하면 끝

## 적 스포너 만들기

- Building Spawner를 복제해 Enemy Spawner로 이름을 변경
    
- Enemy Spawner를 조금 뒤로 이동시키고, building spawner와 enemy spawner의 z축 값을 조금 변경해 줌
    
    - building spawner : z = 50
    - enemy spawner : z = 20
- Enemy Spawner의 gameObjects 리스트에 빌딩들을 제외하고 배달기사와 할머니를 추가함
    
- Spawner로 인해 소환되는 오브젝트는 이름 뒤에 클론이 붙기 때문에, 이름으로 충돌을 판단하기는 어려움
    
    - ⇒ 따라서 할머니와 배달기사 프리팹의 태그를 Enemy로 추가해줌
    - Tag → Add Tag… → Enemy 추가하고 할머니/배달기사 태그 변경해주기
    

![](../image/image%2012.png)

- 플레이어 코드 들어가서 수정
    

```csharp
void OnTriggerEnter2D(Collider2D collider) { 
				if(collider.gameObject.tag == "Enemy") {
				// 태그가 Enemy인 트리거와 부딪혔을 때
        }
        else if (collider.gameObject.tag == "Food") {
				// 태그가 Food인 트리거와 부딪혔을 때
        }
        else if (collider.gameObject.tag == "Golden") {
				// 태그가 Golden인 트리거와 부딪혔을 때
        }
    }
```

## 음식 아이템 만들기

- 위의 적 만들기와 동일한 방식으로 진행하면 됨
- 스프라이트 추가, mover / destroyer 추가, 콜라이더 추가, 프리팹화, 복사 붙이기!
- Food 태그 추가 및 설정
- 추가한 음식들을 Trigger로 설정!!
- 스포너 추가하고 gameObject List를 음식들로 채우기
- 같은 방식으로 금배추도 만들기
