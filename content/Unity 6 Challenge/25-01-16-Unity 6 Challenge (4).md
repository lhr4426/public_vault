---
title: 김치런 만들기 (4)
date: 2025-01-16
tags:
  - Unity
description: Unity 6 Challenge
---
[유니티 6 챌린지 기록](./index.md)

# 플레이어 Hit & Heal

- 플레이어의 체력이라는 개념을 코드로 구현

```csharp
...
		private int lives = 3; // 초기 목숨 3개
    private bool isInvincible = false; // 금배추 먹었을 때 무적임을 확인 
...
		// 일단 대강으로만 만들어 둠
		void OnTriggerEnter2D(Collider2D collider) { 
        if(collider.gameObject.tag == "Enemy") {
            Hit() 
        }
        else if (collider.gameObject.tag == "Food") {
            Heal()
        }
        else if (collider.gameObject.tag == "Golden") {
            StartInvincible()
        }
    }
```

- 적, 아이템은 플레이어와 부딪혔을 때 삭제되게 하고싶음

```csharp
		 	void OnTriggerEnter2D(Collider2D collider) { 
        if(collider.gameObject.tag == "Enemy") {
            if(!isInvincible){ // 무적이 아닐 때만 삭제
                Destroy(collider.gameObject);
            }
            Hit()
        }
        else if (collider.gameObject.tag == "Food") {
            Destroy(collider.gameObject);
            Heal()
        }
        else if (collider.gameObject.tag == "Golden") {
            Destroy(collider.gameObject);
            StartInvincible()
        }
    }
```

- Hit & Heal 매커니즘 작성

```csharp
    void Hit() {
        lives -= 1;
    }

    void Heal() {
        lives = Mathf.Min(3, lives + 1); // 두 값 중에 작은 값 리턴
    }
```

- 무적 매커니즘 작성

```csharp
    void StartInvincible() {
        isInvincible = true;
        Invoke("StopInvincible", 5f); // 한 번 무적이 켜지면 5초 뒤에 끔
    }

    void StopInvincible() {
        isInvincible = false;
    }
```

- 플레이어 삭제 모션 작성

```csharp
...
		public BoxCollider2D playerCollider;
...
    void KillPlayer() {
        playerCollider.enabled = false; // 콜라이더 비활성화 => 플랫폼 무시하고 떨어짐
        playerAnimator.enabled = false; 
        playerRigidBody.AddForceY(jumpForce, ForceMode2D.Impulse); // 한 번 점프하고 떨어지게
    }

    void Hit() {
        lives -= 1;
        if(lives == 0){ // 체력이 0이 되면 죽음
            KillPlayer();
        }
    }
```

# 게임 매니저

## 싱글톤 타입

- 게임 매니저는 씬에 하나만 존재해야 하고, 모든 객체가 접근할 수 있어야 하기 때문에 싱글톤으로 작성함

```csharp
    public static GameManager instance;

    void Awake() {
        if(instance == null) {
            instance = this;
        }
    }
```

## 게임 상태 관리

- 게임의 상태를 enum 객체로 조정

```csharp
public enum GameState {
    Intro,
    Playing,
    Dead
}

...
		public GameState state = GameState.Intro;
```

- IntroUI GameObject를 만들어서 타이틀과 스타트 버튼을 만듬
    - 기본적으로는 비활성화 상태로 두고, 시작할 때 켜지게 만듬
    - 스포너들도 비활성화시킴. 나중에 시작할 때 켜지도록!

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f1734619-ae9d-4c0c-a2b9-7c7618d30d9b/ac436d59-4fb3-420a-9a3e-8e3fc4f5b6d1/image.png)

```csharp
...
    [Header("References")]
    public GameObject introUI;
    public GameObject enemySpawner;
    public GameObject foodSpawner;
    public GameObject goldenSpawner;
...
       void Start()
    {
        introUI.SetActive(true); // 시작할 때 보이게 하기
    }

    void Update()
    {
		    // 현재 상태가 인트로이고, 스페이스 바를 눌렀다면 
        if (state == GameState.Intro && Input.GetKeyDown(KeyCode.Space)) {
            state = GameState.Playing; // 상태를 바꾸고
            introUI.SetActive(false); // 인트로 UI를 보이지 않게 함
            // 그리고 모든 스포너를 활성화 하기
            enemySpawner.SetActive(true);
            foodSpawner.SetActive(true);
            goldenSpawner.SetActive(true); 
        }
    }
```

- 대신에, Spawner에 start 부분을 비우고, OnEnable 메소드를 추가해주어야 함
    - 왜냐면 위에서 처음에 비활성화 하고 나중에 활성화시키게 바꾸었기 때문임. 활성화 될 때 OnEnable이 호출됨

## 플레이어 생명 관리

- 기존에 player.cs 스크립트에서 만들어 둔 내용을 gameManager로 이동시키는 작업이 필요함
    - 왜냐면 플레이어가 죽은걸 알고 그 이후의 작업을 해야하기 때문

```csharp
// GameManager.cs
...
		public int lives = 3;
...
void Update()
    {
        ...

        if (state == GameState.Playing && lives == 0) {
            playerScript.KillPlayer();
            state = GameState.Dead;
            enemySpawner.SetActive(false);
            foodSpawner.SetActive(false);
            goldenSpawner.SetActive(false);
        }

        if (state == GameState.Dead && Input.GetKeyDown(KeyCode.Space)){
            SceneManager.LoadScene("main");
        }
    }
```

```csharp
// Player.cs

...
    void Hit() {
        GameManager.instance.lives -= 1;
    }

    void Heal() {
        GameManager.instance.lives = Mathf.Min(3, GameManager.instance.lives + 1);
    }
...
```

- 더이상 플레이어가 직접 자기 자신을 죽이지 않고, 게임 매니저를 통해서 실행함
- 스포너를 비활성화 할 때, 이미 Invoke를 위해 대기 중이던 spawn 작업을 종료해야 함

```csharp

// Spawner.cs

		void OnDisable() {
        CancelInvoke(); // Invoke 하기로 대기한 메소드를 모두 취소함
    }
```

# 하이 스코어

## 죽었을 때의 UI 생성

- 빈 게임 오브젝트 만들어서 DeadUI라고 지은 후 자식에 Tap To Start 두기
- IntroUI랑 비슷하게 GameManager에서 참조하고, State == GameState.Dead 일 때 활성화 하게 함

## 체력바 생성

- Heart.cs 생성해서 두 개의 스프라이트를 참조하게 함 (채워진 하트, 빈 하트)
- 각 heart는 본인의 liveNumber를 가지고있음

```csharp
    void Update()
    {
        if (GameManager.instance.lives >= liveNumber) { // ex. 현재 목숨이 2 있는데, 이 객체의 liveNumber는 2임
            spriteRenderer.sprite = onHeart; // => 그러면 체력이 채워진 것으로 보이게 함
        } else { // ex. 현재 목숨이 2 있는데, 이 객체의 liveNumber는 3임
            spriteRenderer.sprite = offHeart; // => 그러면 체력이 빠진 것으로 보이게 함
        }
    }
```

## 점수 계산 및 UI 설정

- 점수는 생존한 시간 기준

```csharp
// Game Manager.cs
...
public float playStartTime;
...
if (state == GameState.Intro && Input.GetKeyDown(KeyCode.Space)) {
...
		playStartTime = Time.time; // 스페이스바로 시작했을 때의 시간을 playStartTime으로 기록
}

    float CalculateScore() {
        return Time.time - playStartTime;
    }

    void SaveHighScore() {
        int score = Mathf.FloorToInt(CalculateScore());
        int currentHighScore = PlayerPrefs.GetInt("highScore"); 
        if(score > currentHighScore) {
            PlayerPrefs.SetInt("highScore", score);
            PlayerPrefs.Save();
        }

    }
```

- PlayerPrefs : [https://docs.unity3d.com/6000.0/Documentation/ScriptReference/PlayerPrefs.html](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/PlayerPrefs.html)
    - 암호화 없이 로컬에 데이터를 저장할 수 있게 해줌
    - 암호화 없으니까 중요한건 저장하면 안됨
    - 여기서는 개인의 하이스코어를 저장하기 위해 쓰였음
- 하이어라키에서 Canvas 생성 → Render Mode : Screen Space - Camera → Render Camera : Main Camera 설정
- Canvas 아래에 Text 생성 후 위치 조정 → Score : 0으로 설정

```csharp
// Game Manager.cs
		int GetHighScore() {
        return PlayerPrefs.GetInt("highScore"); 
    }

    // Update is called once per frame
    void Update()
    {
        if(state == GameState.Playing) {
            scoreText.text = "Score : " + Mathf.FloorToInt(CalculateScore()); // 게임 진행 중일땐 현재 점수를
        }

        if(state == GameState.Dead) {
            scoreText.text = "High Score : " + GetHighScore(); // 게임이 종료되었을 때는 하이 스코어를 표시
        }
```

## 게임 속도 증가시키기

- 시간이 지날 수록 더 빠르게 움직이도록 하고자 함
- 10초마다 게임 속도가 0.5씩 증가하도록 함

```csharp
public float CalculateGameSpeed() {
        if (state != GameState.Playing) {
            return 3f;
        }
        float speed = 3f + (0.5f * Mathf.Floor(CalculateScore() / 10));
        // CalculateScore는 게임을 진행한 시간을 반환함
        // 이를 10으로 나누면 10초마다 Mathf.Floor(CalculateScore()/ 10)) = 1이 됨
        // 따라서, 1초마다 0.5f씩 속도가 증가됨
        return Mathf.Min(speed, 20f);
        // 최대 속도를 20f로 설정
    }
```

- 이후, 해당 함수를 Mover와 BackgroundScroll에서 사용하도록 변경하면 끝