---
title: 김치런 만들기 (2)
date: 2025-01-14
tags:
  - Unity
description: Unity 6 Challenge
---
[2025 Unity 6 Challenge](2025%20Unity%206%20Challenge.md)


# 플레이어 점프

- 유니티가 스프라이트의 애니메이션 프레임을 제대로 분리하지 못할 때 ⇒ 스프라이트 클릭 → Open Sprite Editor 에서 수정 가능
    
- 분리한 애니메이션을 모두 선택해서 씬에 드래그&드롭 하면 애니메이션 저장 창이 나타남
    
- Assets 폴더에 Animations 폴더를 만들어서 저장
    
- 캐릭터의 크기를 조정하는 방법에는 Rect Tool 말고, Pixels Per Unit을 변경하는 방법이 있음
    
    - Pixels Per Unit : 1 스케일 단위에 몇 개의 픽셀을 넣어야 하는지를 의미함
    - 스케일 단위 : 씬 창에서 보이는 네모 한 칸
    - PPU가 작을 수록 이미지 자체는 크게 보임. 왜냐하면 네모 한 칸에 적은 수의 픽셀을 넣는다면 그 픽셀 하나하나가 커지기 때문임
    - 반대로, PPU가 클 수록 이미지 자체는 작게 보임. 같은 네모 한 칸에 많은 수의 픽셀을 넣어야 하기 때문에 픽셀이 작아짐
- 이대로 씬에 추가하기만 해도 알아서 달리는 것처럼 보임!
    
    - 그러나 점프, 충돌 등을 구현하기 위해서는 물리법칙이 작용하도록 만들어야 함
- 추가한 애니메이션을 Player로 바꾸고, Rigidbody 2D 컴포넌트를 추가함 ⇒ 물리법칙이 적용됨!
    
- 그대로 그냥 실행하면 끝없이 추락함. 따라서 플랫폼에서 멈출 수 있도록 해야함 : Collider!
    
    - Collider : 물체가 서로 부딪히게 만들고, 충돌을 감지하는 역할을 수행
- Collider 2D를 플레이어와 플랫폼에 추가함으로써 바닥이 없어 떨어지는 일을 막아줌
    
- Edit Collider를 사용해 Collider의 위치나 크기를 조정할 수 있음
    
- 플레이어가 점프할 수 있도록 하기 위해, Player.cs를 제작
    
- 점프 힘 수치를 조절해 가면서 얼마가 적당한지 테스트할 예정이기 때문에, public으로 변수를 설정함
    

```csharp
[Header("Settings")]
public float jumpForce;
```

- 점프를 구현하기 위해서는 Rigidbody 2D를 참조해야 함. 선속도를 수정해야 하기 때문!

```csharp
[Header("References")]
public Rigidbody2D playerRigidBody;
```

- 유저가 스페이스를 누르면 점프하도록 구현하기 위해, 매 프레임마다 점프가 눌렸는지 확인하는 작업이 필요함.

```csharp
void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space)){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse); // public으로 지정한 jumpForce만큼 Y축에 순간적인 힘을 가함
        }
    }
```

- ForceMode2D 안에 있는 Impulse와 Force의 차이를 확인하고자 검색해봄

[Impulse와 Force의 차이 참고자료](https://blog.naver.com/gold_metal/221486016593)

- 우리는 바닥이 이미 달리고있다는 착시효과를 받고있기 때문에 실질적으로 캐릭터는 정지상태임. 그리고 무게를 적용하기 때문에 Impulse를 사용한 것으로 보임
- 이대로 진행하면 중력이 너무 약해서 공중에 너무 오래 떠있음. Rigidbody 2D 컴포넌트 → Gravity Scale의 숫자를 키워가며 적절한 무게를 조정함
- 그리고 점프를 하는 도중에 또 점프를 할 수 있어서 무한 점프가 가능하다는 문제점이 발견됨
- 따라서 점프 중에는 점프할 수 없도록 플레이어가 땅에 붙어있는지 확인하는 변수를 둘 것임

```csharp
private bool isGrounded = true; // 플레이어가 땅에 붙어있나요? == 예
...
void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space) && isGrounded){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse);
            isGrounded = false; // 땅에 이제 안붙어있음
        }
    }
```

- OnCollisionEnter2D() 함수(자동 호출됨!)를 이용해 플레이어의 콜라이더가 땅의 콜라이더와 부딪쳤을 때를 확인하여 다시 점프할 수 있도록 변수를 조작할 것

```csharp
void OnCollisionEnter2D(Collision2D collision) {
        if(collision.gameObject.name == "Platform") { // 부딪친 콜라이더의 게임 오브젝트 명(하이어라키에서 볼 수 있는 그 이름!)이 Platform이면
            isGrounded = true; // 땅에 붙어있다고 판단
        }
    }
```

# 애니메이션

- 하이어라키 창에서 Player 클릭 → 상단 바에 window 클릭 → Animation → Animation 하면 플레이어가 가지고 있는 애니메이션이 나옴
    
- 여기서 애니메이션 이름 클릭 후 Create New Clip… 선택하고 Asset > Animation 폴더에 Player Jump 저장
    
- plyaer_jump에서 앞의 3개의 프레임을 선택 후 애니메이션 윈도우에 드래그&드롭 하면 프레임이 불러와짐
    
- 이 때 속도가 너무 빠르면 Samples 변수 값을 Player Run의 Samples 변수 값과 동일하게 해줌
    
    - 만약에 Samples가 안보이면 아래와 같이 설정
        
        ![](../image/image%202.png)
        
- 같은 방식으로 뒤의 3개 프레임 선택 후 Player Land 애니메이션 제작하기
    
- 제작한 애니메이션 총 3개 (Run, Jump, Land)를 자연스럽게 연결하기 위해 Animator를 사용
    
- 하이어라키 창에서 Player 클릭 → 상단 바에 window 클릭 → Animation → Animator 클릭
    
- Player Run 우클릭 → Make Transition → Player Jump 클릭
    
- Player Jump 우클릭 → Make Transition → Player Land 클릭
    
- Player Land 우클릭 → Make Transition → Player Run 클릭


![](../image/image%203.png)

- 위 사진같은 삼각관계가 나오면 성공
- 이대로 실행하면 run → jump → land가 무한정 반복됨 ⇒ 애니메이션이 나오는 시기를 조절해야 함
    - run → jump 클릭 후 Has Exit Time 비활성화
    - jump → land 클릭 후 Has Exit Time 비활성화
    - Exit Time : 애니메이션이 끝나면 바로 다음 애니메이션을 실행한다는 뜻
- run → jump 클릭 후 아래와 같이 설정 (jump → land에도 동일하게 설정)

![](../image/image%204.png)

- Transition Duration : 이전 애니메이션이 특정 퍼센트 진행되었을 때 이후 애니메이션을 재생하도록 설정하는 것. 부드럽게 보일 수 있으나 지금은 사용하지 않을 것이기 때문에 0으로 설정
- Player Run에서 Jump로 언제 이동해야 하는지 유니티가 모름. Has Exit Time을 비활성화 했기 때문에 직접 설정해주어야 함
- Animator 창에서 Parameters로 이동
    - 해당 창에서 조건이나 트리거를 생성할 수 있음
- 새 파라미터로 int state를 생성한 후, run → jump conditions에 state equals 1, jump → land conditions에 state equals 2 설정
    - 다만 land → run의 경우, Has Exit Time을 활성화 하였기 때문에 직접 파라미터 조건을 설정하지 않아도 됨
- Player Jump와 Land는 끝났을 때 반복되지 않기를 원함.
    - 두 애니메이션 선택 후 Loop Time을 비활성화
    - Loop Time을 비활성화 하면 애니메이션 재생이 끝났을 때 마지막 키프레임에 머물러 있음
- 위에서 지정한 state를 조정하기 위해 player.cs 코드를 수정해야 함

```csharp
...
		[Header("References")]
    public Rigidbody2D playerRigidBody;
    public Animator playerAnimator;
...
	void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space) && isGrounded){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse);
            isGrounded = false;
            playerAnimator.SetInteger("state", 1); // 플레이어가 점프했을 때, 점프 애니메이션을 재생하기 위해 state를 1로 변경함
        }
    }
    
   void OnCollisionEnter2D(Collision2D collision) {
        if(collision.gameObject.name == "Platform") {
            if (!isGrounded){ // 게임이 처음 시작되면 땅이랑 한 번 부딛치기 때문에, 그 때 착지 애니메이션을 재생하지 않도록 점프 유무를 확인함
                playerAnimator.SetInteger("state", 2); // 착지 애니메이션 재생을 위해 state를 2로 변경함
            }
            isGrounded = true;
        }
    }
    
```

- 애니메이션의 속도를 조정하려면 Animator window → 조정하려는 애니메이션 클릭 → Speed 변수 조절
- 전반적인 게임 진행 속도 자체를 조정하려면 Edit → Project Settings → Time → Time Scale 조절 (기본값 1)
- Player Land 애니메이션 2프레임에서 유독 바닥에 빈 공간이 있음
    - Sprites → player_jump → Open Sprite Editor → 착지 프레임 높이를 다른 프레임과 동일하게 해서 아래 자동 생성되는 여백을 지움