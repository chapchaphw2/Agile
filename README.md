# Agile
뉴스 및 소셜 데이터를 활용한 AI 입법 수요 분석 서비스 모델 개발

# 입법 수요 분석 서비스 모델 (AI_LawDemand)

본 저장소는 **뉴스·소셜 데이터 기반 입법 수요 분석** 파이프라인의 대회 제출용 구조를 제공합니다.
- 파이프라인 핵심: 전처리(특히 소셜) → 임베딩 학습(TSDAE+STS/NLI) → 토픽 모델링(UMAP+HDBSCAN+KeyBERT) → 토픽 프로필 통합 → **입법수요 지수** 산출
- 구현은 **노트북 중심**이며, 원본 코드는 보존하고 실행 순서와 경로만 정돈했습니다. (무리한 리팩토링 지양)

## 📁 폴더 구조
```
AI_LawDemand/
├── README.md
├── requirements.txt
├── environment.yml
│
├── dataset/
│   ├── raw/          # 원본 뉴스/소셜
│   ├── processed/    # 전처리·분할 결과
│   └── labels/       # 수작업 라벨 (예: 2,000건)
│
├── model/
│   ├── finetuned_tsdae_news/
│   └── finetuned_tsdae_social/
│
├── src/              # (필수 아님) 공용 유틸/설정이 필요할 때 사용
│
├── notebooks/
│   ├── 1_data_preprocessing.ipynb         # (원본) Social_Data_KcBRT_Finetuning
│   ├── 2_model_training.ipynb             # (원본) Custom_Embedding_Model_Train_2(10_10)
│   ├── 3_clustering_visualization.ipynb   # (원본) 채훈_STM_NTM
│   ├── 4_demand_index_calculation.ipynb   # (원본) 입법수요_지수계산_Colab
│   └── 5_evaluation.ipynb                 # (원본) Parallel_Topic_Labeling_ANN_Leiden
│
├── scripts/
│   ├── setup_environment.sh
│   └── run_all.sh
│
├── figures/
└── demo/
```

## 🚀 실행 순서 (노트북 중심)
1. **1_data_preprocessing.ipynb**: 소셜 KcBRT 분류기로 노이즈 제거 및 유효 샘플 정제. (뉴스는 최소 전처리)
2. **2_model_training.ipynb**: TSDAE 비지도 사전학습 → STS/NLI 지도 미세조정으로 **뉴스/소셜 전용 임베더** 학습.
3. **3_clustering_visualization.ipynb**: 임베딩 → UMAP 차원축소 → HDBSCAN 클러스터링 → KeyBERT 키워드 → **토픽 프로필(Profile) 생성**.
   - ⚠️ 히트맵 등 **중간 산출물이 프로필 생성에 필요**합니다. 단순 시각화로 보이더라도 **삭제 금지**.
4. **4_demand_index_calculation.ipynb**: 통일 스키마의 프로필들을 통계적으로 통합하여 **입법수요 지수** 산출.
5. **5_evaluation.ipynb**: 병렬 토픽 라벨링(ANN/Leiden)과 휴먼-인더-루프 검증/요약.

## 🧩 경로/환경 가이드
- 모든 노트북 상단에 다음 **공통 프리앰블**을 추가하면 경로 혼선을 줄일 수 있습니다. (기존 변수 덮어쓰기 주의)
```python
# ==== Project paths (edit as needed) ====
import os
from pathlib import Path
PROJECT_ROOT = Path(os.getenv("AI_LAWDEMAND_ROOT", "."))
DATA_ROOT = PROJECT_ROOT / "dataset"
MODEL_ROOT = PROJECT_ROOT / "model"
RESULT_ROOT = PROJECT_ROOT / "results"  # 필요 시
for d in [DATA_ROOT/"raw", DATA_ROOT/"processed", DATA_ROOT/"labels", MODEL_ROOT]:
    d.mkdir(parents=True, exist_ok=True)
print("PROJECT_ROOT:", PROJECT_ROOT.resolve())
```
- 대회 환경에서는 `AI_LAWDEMAND_ROOT` 환경변수를 프로젝트 루트로 지정하세요.

## 🛠️ 환경 구성
### Conda
```bash
bash scripts/setup_environment.sh   # conda env 생성
conda activate ai-lawdemand
```

### Pip (대안)
```bash
pip install -r requirements.txt
```

## ▶️ 일괄 실행 (선택)
`run_all.sh`는 papermill로 각 노트북을 순서대로 실행합니다.
```bash
bash scripts/run_all.sh
```

## 📑 감사/인용
- Sentence-Transformers, UMAP-learn, HDBSCAN, KeyBERT 등 오픈소스 생태계에 감사드립니다.
- 데이터는 내부 수집·정제 절차를 따르며, 배포 불가 소스는 업로드하지 않습니다.
