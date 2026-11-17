# BioAI_KU_2026

고려대학교 바이오인공지능 (2026-2) 프로젝트.
종양 침윤 골수계 세포(myeloid) single-cell RNA-seq 데이터의 군집화 재현성과
배치 보정 방법 비교.

## 데이터

- **GSE154763** — 다수 암종의 종양 침윤 골수계 세포 scRNA-seq 데이터
- 현재 분석: MYE count matrix (`GSE154763_MYE_count.h5`)
- 확장 예정: ESCA, KIDNEY, LYM, MYE, OV-FTC, PAAD, THCA, UCEC 8개 암종 병합

## 분석 질문

C1QC+ 대식세포 내부에 구분 가능한 하위 상태가 존재하는지, 그리고 그 구성이
암종에 따라 달라지는지를 확인한다.

## 분석 흐름

1. **QC** — mitochondrial read 비율, 검출 유전자 수, total count 분포 점검
2. **Doublet 검출** — Scrublet, 샘플별 자동 임계값 (제거하지 않고 기록만)
3. **정규화** — `normalize_total(1e4)` + `log1p`, raw count는 `layers['counts']`에 보존
4. **HVG 선정** — 2,000개, seurat flavor
5. **차원 축소 및 군집화** — PCA → neighbors → Leiden
6. **하이퍼파라미터 민감도 분석**
   - PC 30 vs 50, resolution 0.6 vs 1.0, k = 10/15/30 조합별 ARI 비교
   - PC30 → PC50 최빈 대응(modal correspondence) 기반 분할·병합 점검
7. **마커 기반 세포 유형 주석** — Mono_CD14/CD16, Macro_C1QC, Macro_NLRP3,
   cDC1/cDC2/cDC3_LAMP3, pDC, Mast
   - t-test와 Wilcoxon 순위 비교, 리보솜·미토콘드리아·Ig·스트레스 유전자 필터링
8. **배치 보정 방법 비교** — 보정 전 / BBKNN / Harmony / scVI
   - 평가 지표: 환자 silhouette, 군집 silhouette, 단일 환자 편중 군집 수,
     희귀 세포(pDC, cDC1) 보존 여부
9. **대식세포 하위 상태 분석** — C1QC+ 군집만 재군집화 (res 0.4 / 0.6)
10. **분류 모델** — PCA 30성분 + Logistic Regression,
    leave-one-patient-out 방식으로 조직(향후 암종) 예측

## 사용 도구

Scanpy 1.11.5, Scrublet, BBKNN, Harmony (harmonypy), scvi-tools, scikit-learn

seed는 0으로 고정했고, 실행 조건과 패키지 버전은 `run_manifest.json`,
`versions.json`에 함께 저장한다.

## 주요 중간 결과

- 배치 보정 전 단일 환자 편중 군집 9개 → BBKNN·scVI 0개, Harmony 1개
- BBKNN과 Harmony 군집 결과의 ARI 0.508 — 큰 세포 유형은 일치하나
  세부 경계는 방법에 따라 달라짐
- scVI는 환자 섞임이 가장 강했으나 pDC 군집 순도가 떨어져 과보정 경향을 보임
- 대식세포 하위 상태는 resolution에 따라 ARI 0.44 수준으로 변동이 커,
  단일 데이터셋만으로는 확정하지 않고 8개 암종 병합 후 재검토 예정

## 파일 안내

| 파일 | 내용 |
| --- | --- |
| `1115_백수현.ipynb` | **가장 최근 버전.** 배치 보정 비교, 대식세포 하위 상태, 분류 모델 |
| `1106.ipynb` | 군집화 민감도 분석 및 세포 유형 주석 |
| `바이오인공지능중간(10 12).ipynb` | 중간 과제 |
| `바이오인공지능_10_29(피드백_수정본).ipynb` | 피드백 반영본 |
| `바이오인공지능_10_13(화).ipynb` | 주차별 실습 |
| `260916_PCA_HVG_2024250170_백수현.ipynb` | PCA 및 HVG 선정 |
| `바이오인공지능_w3_QC_수정본(논문_R_code_참고).ipynb` | QC (논문 R 코드 참고) |
| `바이오인공지능_week2.ipynb`, `바이오인공지능_week3.ipynb` | 초기 실습 |

## 현재 진행 상황

MYE 단일 데이터셋에 대한 QC부터 배치 보정 비교까지 완료
8개 암종 병합 데이터가 준비되는 대로 동일한 파이프라인을 재실행하여
암종별 대식세포 하위 상태 구성을 비교할 예정

## 실행 환경

Google Colab에서 작성

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/psh135230-design/BioAI_KU_2026/blob/main/1115_%EB%B0%B1%EC%88%98%ED%98%84.ipynb)
