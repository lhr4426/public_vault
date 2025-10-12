---
title: 3. 스파인 FSM 연결 + 자연스럽게 애니메이션 변경하기
date: 2025-10-12
tags:
description:
---
# 자연스럽게 애니메이션 변경하기

[스파인 공식 문서](https://ko.esotericsoftware.com/spine-applying-animations)  

- 애니메이션이 섞일 수 있게끔 설정

```csharp
private void Start()  
{  
    state = skeletonAnimation.AnimationState;  
    stateData = state.Data;  
      
    stateData.SetMix("idle", "run", 0.12f);  
    stateData.SetMix("run", "idle", 0.12f);  
}
```

- 기본적으로 SetMix 는 두 개의 애니메이션 정보를 이름으로 찾아다가 씀
- => AnimationRenferenceAsset으로는 안됨. 그래서 함수를 하나 만듦

```csharp
public void SetMix(AnimationReferenceAsset from, AnimationReferenceAsset to, float duration)  
{  
    // Debug.Log($"[Animation Mix] from name : {from.name}, to name : {to.name}");  
    skeletonAnimation.AnimationState.Data.SetMix(from.name, to.name, duration);  
}
```

- 이러면 AnimationReferenceAsset으로도 쓸 수 있음

# 공격 상태 추가하기

## 스파인에서도 Normalized Time같이 구할 수 있나?

- 진행도를 계산하는 방법은 있음

```csharp
var entry = skeletonAnimation.AnimationState.GetCurrent(0);
if (entry != null)
{
    float currentTime = entry.TrackTime; // 현재 진행된 시간 (초 단위)
    float duration = entry.Animation.Duration; // 애니메이션 전체 길이 (초 단위)
    float normalizedTime = currentTime / duration; // 0 ~ 1 사이의 값
    Debug.Log($"진행도: {normalizedTime}");
}
```

- 이거 한 이유 : 원래는 PlayerAnimator 에서 AttackAnimation()이 이렇게 구성되어 있었음
```csharp
public void AttackAnimation()  
{  
    // 공격 모션 끝나면 Idle로 돌아올 수 있도록 함  
    var entry = skeletonAnimation.AnimationState.SetAnimation(PlayerAnimationKeys.MainAnimationTrack, attack, false);  
    entry.Complete += (trackEntry) => { Player.StateMachine.ChangeState<IdleState>(); };
}
```

- 근데 상식적으로 FSM의 상태 전환을 애니메이터에서 관여하는게 너무 이상함
- 그래서 상태에서 애니메이션의 상태를 확인하고 끝났을 때 바꿔주는걸로 책임을 옮김

```csharp
public class AttackState : BaseState  
{  
    private TrackEntry entry;  
    private float totalDuration;  
    private float currentTime;  
    private float normalizedTime;  
      
    public AttackState(PlayerStateMachine machine) : base(machine) { }  
  
    public override void StartState()  
    {  
        Machine.Player.Animator.AttackAnimation();  
        Machine.Player.Move.rb.velocity = Vector2.zero;  
        entry = Machine.Player.Animator.skeletonAnimation.AnimationState.GetCurrent(  
            PlayerAnimationKeys.MainAnimationTrack  
            );  
  
        currentTime = 0f;  
        totalDuration = entry.Animation.Duration;  
    }
    
    public override void LogicUpdate()  
{  
    currentTime = entry.TrackTime;  
    normalizedTime = currentTime / totalDuration;  
  
    if (normalizedTime >= 1f)  
    {  
        Machine.ChangeState<IdleState>();  
        return;  
    }  
}
...
```

- 그리고 AddEmptyAnimation도 빼버림

![](../image/Pasted%20image%2020251012210631.png)  

- 얼추 잘 때림


# 그래서 나온 클래스 다이어그램



![](../image/Pasted%20image%2020251012211022.png)  

