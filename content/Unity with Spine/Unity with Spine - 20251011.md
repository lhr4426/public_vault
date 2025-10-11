---
title: 2. 스파인과  FSM 연결하기
date: 2025-10-11
tags:
description:
---
# 직접 FSM을 만들어보자.

## 1. 모든 상태의 부모가 될 추상 클래스 만들기

```csharp
public abstract class BaseState  
{  
    protected PlayerStateMachine Machine;  
  
    public BaseState(PlayerStateMachine machine)  
    {  
        this.Machine = machine;  
    }  
      
    public abstract void StartState();  
    public abstract void HandleInput();  
    public abstract void LogicUpdate();  
    public abstract void EndState();  
}
```

## 2. FSM 만들기

```csharp
public class PlayerStateMachine : MonoBehaviour  
{  
    private Dictionary<Type, BaseState> states;  
    private BaseState currentState;  
  
    public Player Player;  
  
  
    public void Init(Player player)  
    {  
        Player = player;  
    }  
  
    // Start is called before the first frame update  
    void Start()  
    {  
        states = new Dictionary<Type, BaseState>()  
        {  
            { typeof(IdleState), new IdleState(this) },  
            { typeof(MoveState), new MoveState(this) },  
        };  
        ChangeState<IdleState>();  
    }  
  
    // Update is called once per frame  
    void Update()  
    {  
        currentState.HandleInput();  
        currentState.LogicUpdate();  
    }  
  
  // 호출 예시 : ChangeState<IdleState>();
    public void ChangeState<T>() where T : BaseState  
    {  
        currentState?.EndState();  
  
        currentState = states[typeof(T)];  
        currentState.StartState();  
    }  
  
}
```

## 3. 실제 상태 만들기

1. Idle 상태
```csharp
public class IdleState : BaseState  
{  
    public IdleState(PlayerStateMachine machine) : base(machine) { }  
  
    public override void StartState()  
    {  
        Machine.Player.Animator.IdleAnimation();  
        Machine.Player.Move.rb.velocity = Vector2.zero;  
    }  
  
    public override void HandleInput()  
    {  
        var moveInput = Machine.Player.Input.Player.Move.ReadValue<Vector2>();  
  
        if (moveInput.x != 0)  
        {  
            Machine.ChangeState<MoveState>();  
            return;  
        }  
    }  
  
    public override void LogicUpdate()  
    {  
   
    }  
  
    public override void EndState()  
    {  
        Machine.Player.Animator.AddEmptyAnimation();  
    }  
}
```



2. Move 상태
```csharp
public class MoveState : BaseState  
{  
    public MoveState(PlayerStateMachine machine) : base(machine) { }  
  
    public override void StartState()  
    {  
        Machine.Player.Animator.MoveAnimation();  
    }  
  
    public override void HandleInput()  
    {  
        var moveInput = Machine.Player.Input.Player.Move.ReadValue<Vector2>();  
        if (moveInput.x == 0)  
        {  
            Machine.ChangeState<IdleState>();  
            return;  
        }  
    }  
  
    public override void LogicUpdate()  
    {  
        Machine.Player.Move.Move();  
    }  
  
    public override void EndState()  
    {  
        Machine.Player.Animator.AddEmptyAnimation();  
    }  
}
```

# 애니메이션을 붙이자

```csharp
public class PlayerAnimator : MonoBehaviour  
{  
    #region 애니메이션들  
    [Header("Animation References")]  
    public AnimationReferenceAsset idle;  
    public AnimationReferenceAsset move;  
    public AnimationReferenceAsset attack;  
  
    [Header("Animation Mix Settings")]   
    public float mixEmptyDuration;  
    public float delayEmpty;  
      
    #endregion  
  
  
    private const int MainAnimationTrack = 0;   
    [HideInInspector] public SkeletonAnimation skeletonAnimation;  
  
    public Player Player;  
  
  
    public void Init(Player player)  
    {  
        Player = player;  
    }  
  
    private void Awake()  
    {  
        skeletonAnimation = GetComponent<SkeletonAnimation>();  
          
        if (skeletonAnimation == null)  
        {  
            Debug.LogError("SkeletonAnimation is null");  
            return;  
        }  
    }  
  
    public void IdleAnimation()  
    {  
        skeletonAnimation.AnimationState.SetAnimation(MainAnimationTrack, idle, true);  
    }  
  
    public void MoveAnimation()  
    {  
        skeletonAnimation.AnimationState.SetAnimation(MainAnimationTrack, move, true);  
    }  
  
    public void AttackAnimation()  
    {  
        // 공격 모션 끝나면 Idle로 돌아올 수 있도록 함  
        var entry = skeletonAnimation.AnimationState.SetAnimation(MainAnimationTrack, attack, false);  
        entry.Complete += (trackEntry) => { Player.StateMachine.ChangeState<IdleState>(); };  
    }  
  
    public void AddEmptyAnimation()  
    {  
        skeletonAnimation.AnimationState.AddEmptyAnimation(MainAnimationTrack, mixEmptyDuration, delayEmpty);  
    }  
      
}
```


# 좌우로 움직일 수 있게 해보자

```csharp
public class PlayerMove : MonoBehaviour  
{  
    #region 이동 관련 변수  
  
    [Header("Move Settings")]   
    public float moveSpeed;  
  
    public bool isLeft = false;  
  
    #endregion  
  
  
  
    public Rigidbody2D rb;  
    private Player player;  
  
    public void Init(Player player)  
    {  
        this.player = player;  
    }  
      
    public void Move()  
    {  
        var moveInput = player.Input.Player.Move.ReadValue<Vector2>();  
        Turn(moveInput.x < 0);  
        rb.velocity = new Vector2(moveInput.x * moveSpeed, rb.velocity.y);  
    }  
      
      
  
    public void Turn(bool isLeft)  
    {  
        this.isLeft = isLeft;  
        player.Animator.skeletonAnimation.Skeleton.ScaleX = isLeft ? -1 : 1;  
    }  
  
}
```