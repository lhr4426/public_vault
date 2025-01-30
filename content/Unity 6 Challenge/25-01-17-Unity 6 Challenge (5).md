---
title: 김치런 만들기 (5)
date: 2025-01-17
tags:
  - Unity
description: Unity 6 Challenge
---
[유니티 6 챌린지 기록](./index.md)

원래는 이대로 끝내려고 했는데, 튜토리얼만 따라가기엔 너무 심심해서 조금 변형해 보았음

# 기존의 튜토리얼에서 변형시키기

## a, d 또는 왼쪽 방향키, 오른쪽 방향키로 직접 횡이동 가능하게 하기

```csharp
// Player.cs
...
if(Input.GetKeyDown(KeyCode.A) || Input.GetKeyDown(KeyCode.LeftArrow)) {
            transform.position += Vector3.left * moveSpeed * Time.deltaTime;
        }
        if(Input.GetKeyDown(KeyCode.D) || Input.GetKeyDown(KeyCode.RightArrow)) {
            transform.position += Vector3.right * moveSpeed * Time.deltaTime;
        }
```

- 이렇게 했더니 안됨.. 되게 이상하게 움직임
- 그래서 다른 방법을 찾아봄
- [https://torotoblog.tistory.com/81](https://torotoblog.tistory.com/81)
- 트랜스폼을 직접 바꾸는게 아니고 리지드바디의 속력을 바꿔야 하는 듯
- Input Manager에서 Axis에 horizontal이 A, D와 왼쪽/오른쪽 방향키를 인식하도록 하고 따라해봄

```csharp
void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space) && isGrounded){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse);
            isGrounded = false;
            playerAnimator.SetInteger("state", 1);
        }
        Vector2 moveInput = new Vector2(Input.GetAxisRaw("Horizontal"));
        moveVelocity = moveInput.normalized * moveSpeed;
        playerRigidBody.MovePosition(playerRigidBody.position + moveVelocity * Time.deltaTime);
    }
```

- Vector2.normalized가 뭔가 봤더니, 벡터 값을 정규화 하는데 쓰인다고 함. 이게 없으면 대각선 이동에서 속도가 갑자기 증가함
- 근데 이렇게 했을 때 캐릭터가 이상하게 뛰는 애니메이션을 취하고 있음
- 이번에 MovePosition 말고 AddForce를 해보기로 함

```csharp
void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space) && isGrounded){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse);
            isGrounded = false;
            playerAnimator.SetInteger("state", 1);
        }
        playerRigidBody.AddForceX(moveSpeed * Input.GetAxis("Horizontal"), ForceMode2D.Force);
        // force는 지속적으로 힘을 주기 때문에 사용함
        
    }
```

- 되긴 하는데, 오래 눌렀을 때 너무 훅 이동함. 힘을 증가시켜서 그런 것 같음
- 왜 MovePosition은 안되는거고 AddForce가 되는지 알아봤음
- [https://velog.io/@kwontae1313/Unity-Addforce와-velocity의-차이-알아보기-g2lnrewg](https://velog.io/@kwontae1313/Unity-Addforce%EC%99%80-velocity%EC%9D%98-%EC%B0%A8%EC%9D%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-g2lnrewg)
- MovePosition을 쓸거였으면 최종 목적지까지 써야 했음
- 대신 난 최대 속도와 투명 벽을 써서 막으려고 함
- 성공!

```csharp
void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space) && isGrounded){
            playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse);
            isGrounded = false;
            playerAnimator.SetInteger("state", 1);
        }
        playerRigidBody.AddForceX(moveForce * Input.GetAxis("Horizontal"), ForceMode2D.Force);
        // TODO : 최대 속도 제한하기
        if(playerRigidBody.linearVelocityX > maxMoveSpeed)
        {
            playerRigidBody.linearVelocityX = maxMoveSpeed;
        }
        else if(playerRigidBody.linearVelocityX < -maxMoveSpeed)
        {
            playerRigidBody.linearVelocityX = -maxMoveSpeed;
        }
        
    }
```

## 특정 키를 누르면 칼을 휘두르기

- 칼 도트 찍기
- 칼 애니메이션을 드래그&드롭으로 씬에 추가
- 플레이어의 자식 오브젝트로 두고 위치를 플레이어의 우상단으로 조정
- Input Manager에서, Fire Axis 추가하고 left shift와 마우스 왼쪽 클릭으로 가능하게 함
- Player.cs 코드에서, Fire Axis Input이 있을 때 칼 오브젝트를 소환

```csharp
 			 	if(Input.GetAxis("Fire") > 0) {
            knife.SetActive(true); // 위에 public GameObject knife로 받아옴
        }
```

- 애니메이션이 끝나면 다시 칼을 비활성화하고자 함
    - 참고자료 : [https://elena90.tistory.com/entry/애니메이션-종료-시점-이벤트](https://elena90.tistory.com/entry/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98-%EC%A2%85%EB%A3%8C-%EC%8B%9C%EC%A0%90-%EC%9D%B4%EB%B2%A4%ED%8A%B8)
- 애니메이션 윈도우 → 키프레임 바로 아래에 마우스 우클릭 > Add Animation Event
- 이후 knife.cs에 아래와 같은 핸들러 작성

```csharp
	 	public void EndSwing(){
        gameObject.SetActive(false);
    }
```

- knife.cs 상단에 Animator를 public으로 받아오게 하고, 나머지는 연결

![](../image/image%2013.png)

- 성공!

## 칼을 휘둘러야만 배추, 고추, 마늘을 얻을 수 있게 하기

- 가장 먼저, 칼에 박스 콜라이더를 추가하고 설정해 줌
- 원래 player.cs에 있던 충돌 감지 부분을 knife로 옮겨줌 (음식 부분만!)
- 힐 부분을 빼버림 (나중에 힐 감지는 따로 음식 모아서 하게 할거라)
- 성공!

## 배추, 고추, 마늘을 모두 얻어 김치를 만들면 생명력을 회복할 수 있게 하기

- 게임 매니저가 배추, 고추, 마늘의 수집 여부를 파악하게 함
- 현재 어떤 재료를 수집했는지 체력 바 밑에 보이게 하기 위해, 스프라이트를 미리 씬에 추가함
- 이후에 금배추를 포함해서 만들면 5초 무적으로 만들 예정임
- 배추, 고추, 마늘 수집 여부를 bool list로 관리하고, 금배추를 먹은 경우 hasGoldBaechu로 따로 검사할 예정
- Food.cs에 다음을 작성

```csharp
public class Food : MonoBehaviour
{
    public int foodIndex;
}
```

- Hit과 Heal을 게임 매니저가 관리하도록 변경함

```csharp
    private bool[] goldenFoodList; // 혹시나 모를 다른 금 음식 추가를 위해..
    ...
    
    
    public void GetFood(int foodIndex, bool isGolden) {
        foodList[foodIndex] = true;
        if(isGolden){
            goldenFoodList[foodIndex] = true;
            goldenFoodObjects[foodIndex].gameObject.SetActive(true);
        } else{ // 금 음식이 있으면 그게 우선적으로 보이게 하려고 했음
            if(!goldenFoodList[foodIndex]){
                foodObjects[foodIndex].gameObject.SetActive(true);
            }
        }
        CheckFood();
    }

    void CheckFood() {
        foreach (bool item in foodList)
        {
            if (!item) return; // 하나라도 false가 있으면 return 될 것임
        }
        Heal();
        if (goldenFoodList[(int)Foods.Baechu])
        {
            playerScript.StartInvincible();
        }
        // 음식들 비활성화 하기
        Array.Clear(foodList, 0, foodList.Length); // 논리적
        
        foreach (GameObject items in foodObjects){  // 물리적 (Scene 안에서 보여지는)
            items.SetActive(false);
        }
        foreach (GameObject items in goldenFoodObjects)
        {
            items.SetActive(false);
        }
    }
```

- 음식을 얻는 부분과 얻고 나서 체크하는 부분을 나눔
- 일반 음식 오브젝트 리스트와 금 음식 오브젝트 리스트로 나눠서 소환하게 함

## 픽셀 애니메이션으로 무적인거 티내기

![](../image/image%2014.png)

- 칼도 그렇고 이 불도 아래의 사이트에서 직접 찍어봤음

[https://www.piskelapp.com/p/create/sprite](https://www.piskelapp.com/p/create/sprite)

- 하이어라키에 추가하고 PPU 조절 후 player 오브젝트 자식으로 둠. 기본값 비활성화로 해둠
- Player.cs 코드 상에서 invincible을 판단하기 때문에, 코드에서 이펙트 참조하게 함

```csharp
    public void StartInvincible() {
        isInvincible = true;
        invincibleEffect.SetActive(true);
        Invoke("StopInvincible", 5f);
    }

    void StopInvincible() {
        invincibleEffect.SetActive(false);
        isInvincible = false;
    }
```

- 성공!

![](../image/image%2015.png)
## UI 일부 수정하기

- 죽은 뒤에는 HighScore를 화면 중앙에 표시하도록 변경함
- 우상단에 조작법을 추가함

# 배포

- Files → Build Profiles → Web → Switch Platform
- Windows → Package Manager → WebGL Publisher 설치
- 상단에 Publish → WebGL Project → Get Started → Build and Publish → 폴더 선택하면 빌드 끝