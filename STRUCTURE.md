# 지율이, 해인이, 한글나라 — 코드 구조

## 파일 구성

```
baby_game/
├── index.html      ← 게임 전체 (HTML + CSS + JS 단일 파일)
├── 지율이2.png     ← 지율이 스프라이트 시트 (2×2 그리드)
└── 해인이2.png     ← 해인이 스프라이트 시트 (2×2 그리드)
```

---

## 게임 개요

- **캔버스 해상도**: 900×500 (반응형 스케일링)
- **언어**: Vanilla JavaScript (ES6), HTML5 Canvas 2D
- **장르**: 횡스크롤 플랫포머 + 한글 단어 학습
- **스테이지 수**: 10개

---

## JS 코드 구조 (index.html 내부)

### 1. 오디오 (line 25–48)
- `tone(freq, dur, type, vol, freqEnd)` — Web Audio API로 효과음 생성
- 효과음 함수: `sfxJump`, `sfxDie`, `sfxStomp`, `sfxCoin`, `sfxClear`, `sfxTitle`, `sfxStar`, `sfxBoost`, `sfxKey`, `sfxBoss`

### 2. 스프라이트 시스템 (line 50–102)
- `sprImg` / `sprImg2` — 지율이·해인이 PNG 로드
- **스프라이트 시트 구조** (2×2 그리드):
  ```
  [0,0] idle  [1,0] jump
  [0,1] run   [1,1] dead
  ```
- `CHARS[]` — 캐릭터별 이미지와 포즈별 픽셀 오프셋 정의
- `drawSprite()` — 현재 선택 캐릭터 렌더링 (상태에 따라 포즈 선택)
- `drawSpriteChar()` — UI 미리보기용 렌더링

### 3. 입력 처리 (line 104–153)
- **키보드**: `K{}` (현재 상태), `JP{}` (방금 눌림), `jp()` 함수
- **모바일 터치**: `TOUCH{}`, `BTN{}` (좌/우/점프 버튼 좌표)
  - `getTouchBtn()` — 터치 좌표→버튼 매핑
  - `updateTouchState()` — 멀티터치 처리
- `jpTouch()` / `jpTap()` — 터치 one-shot 이벤트

### 4. 상수 (line 155–194)
| 상수 | 설명 |
|------|------|
| `W=900, H=500` | 캔버스 크기 |
| `GY=418` | 지면 Y좌표 |
| `PW=38, PH=52` | 플레이어 히트박스 |
| `EW=52, EH=46` | 적 히트박스 |
| `GRAV=0.58` | 중력 |
| `JVY=-13.5` | 점프 초속 |
| `SPD=4.8` | 이동 속도 |
| `WORDS[]` | 한글 단어 20개 (적에게 표시) |
| `ECLR[]` | 적 색상 10가지 |

### 5. 스테이지 테마 (line 168–180)
10개 테마, 각각 하늘색·지면색·발판색·배경 장식 타입(`decor`) 포함:

| # | 이름 | decor |
|---|------|-------|
| 1 | 버섯나라 | mushroom |
| 2 | 구름위의산책 | cloud |
| 3 | 별이빛나는밤에 | star |
| 4 | 사탕숲 | candy |
| 5 | 용암동굴 | lava |
| 6 | 수중왕국 | bubble |
| 7 | 벚꽃마을 | petal |
| 8 | 무지개다리 | rainbow |
| 9 | 얼음왕국 | snow |
| 10 | 황금궁전 | gold |

### 6. 스테이지 데이터 (line 197–208)
`STAGES[]` — 10개 스테이지, 각각 포함:
- `w` — 맵 전체 너비
- `flag` — 골 깃발 X 좌표
- `plats` — 발판 목록 `[x, y, width]`
- `enems` — 지면 적 목록 `[x, 단어인덱스, 이동범위]`
- `platEnems` — 발판 위 적 `[x, 발판Y, 단어인덱스, 이동범위]`
- `coins` — 코인 X 좌표 목록

### 7. 게임 상태 변수 (line 210–216)
```js
state = 'title' | 'playing' | 'stageclear' | 'gameover' | 'win'
player, enemies, coins, sd(현재스테이지), th(현재테마)
cam.x, timer, gt(글로벌틱), mech(스테이지별 특수 상태)
```

---

## 핵심 함수

### 물리 엔진 (line 220–248)
- `overlaps(ax,ay,aw,ah, bx,by,bw,bh)` — AABB 충돌 감지
- `getSolids()` — 현재 프레임의 모든 충돌체 반환 (발판+지면+거품발판)
- `moveEntity(e, solids)` — X→Y 순서로 이동 및 충돌 해소

### 스테이지 메카닉 초기화 (line 251–302) — `initMechanic(n)`

| 스테이지 | 메카닉 | 참고 게임 |
|----------|--------|-----------|
| 3 | ⭐ 별 먹으면 무적 (8초) | 팩맨 |
| 4 | 👷 레밍즈 3마리 따라옴 | 레밍즈 |
| 5 | 🔥 용암이 서서히 차오름 | 동키콩 |
| 6 | 🫧 떠다니는 거품 발판 + 약한 중력 | 버블보블 |
| 7 | ⚔️ 보스 (HP 20, 2페이즈) | 파이널판타지 |
| 8 | 💨 로켓 부스트 패드 (속도 2.6배) | 소닉 |
| 9 | 🧊 얼음 — 미끄러운 가속/감속 | 아이스클라이머 |
| 10 | 🔑 열쇠 먹어야 문 열림 | 젤다의 전설 |

### 스테이지 초기화 (line 304–321) — `initStage(n)`
플레이어·적·코인 생성, 카메라 리셋, 메카닉 초기화

### 게임 업데이트 (line 440–509) — `updatePlaying()`
1. 플레이어 입력 처리 (키보드/터치 통합)
2. 스테이지별 중력/속도 보정 (stage 6: 약한 중력, stage 9: 아이스)
3. `moveEntity()` 호출
4. 적 순찰 이동
5. 코인 수집 판정
6. 적 밟기/충돌 판정
7. `updateMechanic()` 호출
8. 깃발 도달 판정

---

## 렌더링 파이프라인

`playing` 상태의 드로우 순서:
```
drawBG()          ← 하늘 그라디언트 + 파티클 + 시차 실루엣 + 구름
drawGround()      ← 지면 + 잔디 + 테마별 특수 효과
drawPlatforms()   ← 발판 (테마별 디자인)
drawCoins()       ← 코인 (스핀 애니메이션)
drawFlag()        ← 골 깃발
drawMechanic()    ← 스테이지 특수 오브젝트 (별·용암·보스·열쇠 등)
drawEnemies()     ← 적 (한글 단어 표시)
drawPlayer()      ← 플레이어 스프라이트
drawHUD()         ← 상단 UI (목숨·점수·스테이지명·메카닉 힌트)
drawMobileControls() ← 모바일 터치 버튼
```

---

## 게임 상태 흐름

```
title ──[스타트]──> playing ──[깃발]──> stageclear ──[다음 스테이지]──> playing
                      │                                                    │
                   [목숨 0]                                          [10스테이지 완료]
                      │                                                    │
                   gameover ──[재시작]──> playing                        win ──[재시작]──> playing
```

---

## 캐릭터 선택

- 타이틀 화면에서 ←/→ 또는 숫자키 1/2로 선택
- `selectedChar` 변수가 0(지율이) 또는 1(해인이)
- 선택된 캐릭터로 `drawSprite()` 호출

---

## 폴백 캐릭터

`drawCat()` (line 1312–1345) — 스프라이트 PNG 로드 실패 시 Canvas로 직접 그린 고양이 캐릭터
