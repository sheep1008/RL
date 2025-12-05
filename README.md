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

### 3.2. 보상 함수 설계
본 연구에서는 에이전트가 투입 대비 산출을 고려하도록 Net Profit 보상 함수를 설계했습니다.

#### 기본 비용
모든 행동에 대해 페널티를 부여하여, 에이전트가 최소한의 행동으로 게임을 진행하도록 유도합니다.
* **Step Penalty:** -0.3 (일반 클릭 및 깃발 설치)

#### 화음 이득
화음 시도 시, **(획득한 정보량 - 투자한 비용)**이 양수일 때만 보상을 부여합니다.

$$R_{chording} = 0.3 \times (N_{opened} - N_{flags}) - 0.3$$

* $N_{opened}$: 화음으로 인해 열린 셀의 개수
* $N_{flags}$: 화음을 위해 투자한 깃발의 개수

해석: 깃발을 설치하는 비용을 지불하고도 남는 이득이 있을 때만 긍정적인 보상을 제공하여, 확실하지 않은 상황에서의 무분별한 깃발 설치를 억제합니다.

## 4. 파일 구조

```text
.
├── train_ppo_v9.py          # PPO 학습 및 미세조정 실행 스크립트
├── minesweeper_env_v9.py    # Gymnasium 기반 커스텀 지뢰찾기 환경 (Reward Shaping 적용)
├── saved_models/            # 모델 체크포인트 저장소
│   ├── Minesweeper-v8-Efficiency__train_ppo_v8__1__1764778213.pth  # 사전 학습된 8차 모델
│   └── Minesweeper-v9-NetProfit__train_ppo_v9__1__1764858933.pth   # 최종 모델
└── README.md                # 프로젝트 명세서

## 5. 설치 및 실행 가이드

### 5.1. 환경 설정
Python 3.x 환경에서 다음의 종속성 라이브러리를 설치합니다.

```bash
pip install torch gymnasium numpy tyro tensorboard wandb
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
