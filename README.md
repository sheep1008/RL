# Deep RL for Minesweeper Efficiency Optimization

본 프로젝트는 심층 강화학습 알고리즘인 PPO를 활용하여 지뢰찾기 게임의 플레이 효율성을 극대화하는 AI 에이전트 개발 연구입니다.

## 1. 연구 개요

지뢰찾기는 불완전한 정보를 바탕으로 추론을 수행해야 하는 대표적인 환경입니다. 본 연구는 단순한 게임 클리어를 넘어, 인간 고수들의 플레이 방식인 화음 기술을 에이전트가 학습하게 하여 행동의 효율성을 최적화하는 것을 목표로 합니다.
이를 위해 'Net Profit' 기반의 보상 체계를 새롭게 제안하고, 이를 통해 에이전트가 불필요한 탐색을 줄이고 정보 획득량을 극대화하도록 유도합니다.

## 2. 프로젝트 목표

### 2.1. 정량적 목표
* **Efficiency > 1.0 달성:** 1회의 클릭 당 1개 이상의 셀을 오픈하는 고효율 플레이 구현.

### 2.2. 정성적 목표
* **화음 전략 습득:** 깃발 설치 후 주변 셀을 한 번에 여는 '화음' 기능을 활용하는 정책 학습.
* **행동 최적화:** 불필요한 깃발 설치 및 의미 없는 클릭 최소화.

## 3. 방법론

### 3.1. 핵심 알고리즘
* **Algorithm:** PPO (Proximal Policy Optimization)

### 3.2. 신경망 구조 (Network Architecture)
단순한 CNN이 아닌, **잔차 연결(Residual Connection)**을 적용하여 깊은 신경망에서도 효율적인 학습이 가능하도록 설계했습니다.
* **Input:** 9x9x1 (정규화된 그리드 정보)
* **Backbone:** Initial Conv2d + 4 Residual Blocks (3x3 Conv, ReLU)
* **Head:** Actor (Action Probabilities), Critic (State Value)

### 3.3. 환경 설정 (Environment Specs)
* **Grid Size:** 9x9 (81 Cells)
* **Mines:** 10 (Beginner Mode)
* **Action Space:** 162개 (0~80: 클릭/화음, 81~161: 깃발 설치)
* **Observation:** -1.0 ~ 1.0으로 정규화된 1채널 텐서

### 3.4. 보상 함수 설계
본 연구에서는 에이전트가 투입 대비 산출을 고려하도록 Net Profit 보상 함수를 설계했습니다.

#### 상태 보상 (Terminal Rewards)
* **Win:** +10.0 (모든 안전한 셀 오픈)
* **Lose:** -10.0 (지뢰 클릭 또는 안전한 곳에 깃발 설치 시)
* **Invalid Action:** -1.0 (이미 열린 곳 클릭 등)

#### 효율성 보상 (Efficiency Rewards)
모든 행동에 비용을 부과하고, 화음 성공 시 순수익을 계산합니다.

1.  **Step Penalty:** -0.3 (모든 행동 기본 비용)
2.  **Chording Gain (화음 이득):**
    $$R_{chording} = 0.3 \times (N_{opened} - N_{flags}) - 0.3$$
    * $N_{opened}$: 화음으로 인해 열린 셀의 개수
    * $N_{flags}$: 화음을 위해 투자한 깃발의 개수

    해석: 깃발을 설치하는 비용을 지불하고도 남는 이득이 있을 때만 긍정적인 보상을 제공하여, 확실하지 않은 상황에서의 무분별한 깃발 설치를 억제합니다.

## 4. 파일 구조

```text
.
├── train_ppo_v9.py          # PPO 학습 (ResNet 적용) 및 미세조정 실행 스크립트
├── minesweeper_env_v9.py    # Gymnasium 기반 커스텀 지뢰찾기 환경 (Net Profit Reward)
├── saved_models/            # 모델 체크포인트 저장소
│   ├── Minesweeper-v8-Efficiency__train_ppo_v8__1__1764778213.pth  # 사전 학습된 8차 모델
│   └── Minesweeper-v9-NetProfit__train_ppo_v9__1__1764858933.pth   # 최종 모델
└── README.md                # 프로젝트 명세서
```
## 5. 설치 및 실행 가이드

### 5.1. 환경 설정
Python 3.x 환경에서 다음의 종속성 라이브러리를 설치합니다.

```bash
pip install torch gymnasium numpy tyro tensorboard wandb pillow
```
### 5.2. 학습 실행
```bash
python3 train_ppo_v9.py \
    --seed 1 \
    --track \
    --capture-video \
    --load_model_path "saved_models/Minesweeper-v8-Efficiency__train_ppo_v8__1__1764778213.pth" \
    --learning-rate 1e-4 \
    --env_id "Minesweeper-v9-NetProfit"
```
