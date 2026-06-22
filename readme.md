# GLP-1 Peptide Generation Project

GLP-1 계열 peptide drug를 대상으로 **SMILES 기반 화학 구조 모델링**과 **amino acid sequence 기반 조건부 생성 모델링**을 수행한 프로젝트입니다.

본 프로젝트는 GLP-1 계열 약물의 구조적 특징과 sequence-level 특징을 각각 다른 representation에서 학습하고, latent space에서 약물 간 관계와 transition을 분석하는 것을 목표로 합니다.

---

## Project Overview

GLP-1 계열 peptide drug는 당뇨병 및 비만 치료제 개발에서 중요한 약물군입니다.

Semaglutide, Tirzepatide, Retatrutide와 같은 약물들은 receptor targeting, amino acid sequence, chemical modification 측면에서 서로 다른 특징을 가집니다.

본 프로젝트에서는 이를 다음 두 가지 관점에서 분석합니다.

| Part | Input | Purpose |
|---|---|---|
| SMILES VAE | SMILES / SELFIES / Morgan Fingerprint | 화학 구조 기반 latent representation 학습 |
| Sequence cVAE | Amino acid sequence + receptor condition | receptor condition 기반 peptide sequence 생성 |

추가적으로, 두 latent space 사이의 연결 가능성을 확인하기 위한 bridge mapping 실험도 보조적으로 다룹니다.

---

## Repository Structure

```bash
MLPROJECT/
├── dataset/
│   ├── milestone.csv
│   ├── Sequences.csv
│   ├── Sequences_additional.csv
│   └── SMILES.csv
│
├── sequence/
│   ├── after_data_added/
│   ├── cvae_last.pt
│   ├── metadata.json
│   ├── optimized_sequence_cvae.pt
│   └── sequence_cvae_checkpoint...
│
├── smiles/
│   ├── checkpoints/
│   ├── cvae_gru_best_evod.pt
│   ├── cvae_gru_last.pt
│   ├── distance_matrix_only.png
│   ├── pca_centroid_v4.png
│   ├── umap_camp_pec50.png
│   ├── umap_full.png
│   └── umap_milestone_full_v7.png
│
├── AlphaFold2_Eval.ipynb
├── Dataset.ipynb
├── Sequence_cVAE.ipynb
├── Sequence_generation_msa.ipynb
├── SMILES_VAE.ipynb
├── .gitignore
└── readme.md
```

---

## Dataset

`dataset/` 폴더에는 프로젝트에 사용한 원본 및 추가 데이터가 저장되어 있습니다.

| File | Description |
|---|---|
| `SMILES.csv` | SMILES VAE 학습에 사용한 GLP-1 계열 약물의 SMILES 데이터 |
| `Sequences.csv` | Sequence cVAE 학습에 사용한 amino acid sequence 데이터 |
| `Sequences_additional.csv` | 희소 condition 보강을 위해 추가한 sequence 데이터 |
| `milestone.csv` | 주요 GLP-1 계열 약물 및 milestone 분석용 데이터 |

---

## Notebooks

### `Dataset.ipynb`

데이터셋을 정리하고 모델 학습에 사용할 수 있는 형태로 전처리하는 노트북입니다.

주요 내용은 다음과 같습니다.

- SMILES 데이터 정리
- amino acid sequence 데이터 정리
- receptor condition label 구성
- milestone drug 정보 정리
- 학습 및 분석에 필요한 데이터 저장

---

### `SMILES_VAE.ipynb`

SMILES 기반 VAE 모델을 학습하고 분석하는 노트북입니다.

SMILES는 peptide drug의 화학 구조를 표현하지만, 길이가 길고 복잡하기 때문에 SELFIES 변환과 BPE tokenization을 함께 사용합니다.

주요 내용은 다음과 같습니다.

- SMILES preprocessing
- SELFIES 변환
- BPE tokenization
- Morgan fingerprint 생성
- SMILES VAE 모델 학습
- reconstruction 분석
- latent space visualization
- chemical property correlation 분석
- distance matrix 분석
- drug trajectory 분석
- interpolation 분석

---

### `Sequence_cVAE.ipynb`

Amino acid sequence와 receptor condition을 함께 사용하는 conditional VAE 모델을 학습하는 노트북입니다.

Receptor condition은 다음과 같은 3-bit vector로 표현합니다.

```text
[GLP1R, GIPR, GCGR]
```

| Condition | Meaning |
|---|---|
| `100` | GLP1R only |
| `010` | GIPR only |
| `001` | GCGR only |
| `101` | GLP1R + GCGR |
| `110` | GLP1R + GIPR |
| `111` | GLP1R + GIPR + GCGR |

주요 내용은 다음과 같습니다.

- amino acid sequence tokenization
- receptor condition encoding
- Sequence cVAE 모델 정의
- reconstruction 학습
- conditional generation
- latent space visualization
- ProtParam 기반 sequence property 분석
- drug trajectory 분석
- interpolation 분석

---

### `Sequence_generation_msa.ipynb`

Sequence cVAE의 latent space가 실제 peptide family 간 residue 변화와 연결되는지 확인하기 위한 MSA 기반 분석 노트북입니다.

주요 내용은 다음과 같습니다.

- GLP1 / GIP / GCG 계열 sequence 비교
- MSA 기반 target residue position 설정
- receptor-specific latent direction 계산
- latent steering 수행
- 생성 sequence의 receptor-like mutation 확인

이 분석은 latent vector를 특정 receptor 방향으로 이동시켰을 때, decoder output sequence에서 관련 residue 변화가 나타나는지 확인하기 위한 보조 분석입니다.

---

### `AlphaFold2_Eval.ipynb`

Sequence cVAE가 생성한 peptide 후보에 대해 AlphaFold2 기반 구조 예측 결과를 분석하는 노트북입니다.

주요 내용은 다음과 같습니다.

- 생성 후보 sequence 선택
- AlphaFold2 결과 확인
- pLDDT 및 pTM 분석
- predicted structure visualization
- 생성 sequence의 구조적 plausibility 확인

AlphaFold 분석은 생성 sequence가 구조적으로 완전히 붕괴하지 않았는지 확인하기 위한 sanity check로 사용합니다.

이는 receptor binding이나 biological activity를 직접 검증하는 결과가 아닙니다.

---

## Model Components

### 1. SMILES VAE

SMILES VAE는 peptide drug의 화학 구조 정보를 latent space에 학습하기 위한 모델입니다.

Peptide SMILES는 길이가 길고 복잡하기 때문에, 본 프로젝트에서는 SMILES를 SELFIES로 변환하고 BPE tokenization을 적용하여 reconstruction 난이도를 낮추었습니다.

주요 구성은 다음과 같습니다.

- Morgan fingerprint encoder
- SELFIES CNN encoder
- Conditional latent space
- Morgan fingerprint decoder
- SELFIES GRU decoder
- SELFIES to SMILES conversion

SMILES 기반 모델은 amino acid sequence만으로는 반영하기 어려운 chemical modification, lipidation, linker structure 등을 구조 정보 관점에서 다루기 위해 사용합니다.

---

### 2. Sequence cVAE

Sequence cVAE는 amino acid sequence와 receptor condition을 함께 입력으로 사용하는 conditional VAE 모델입니다.

주요 구성은 다음과 같습니다.

- amino acid embedding
- positional embedding
- sequence encoder
- condition embedding
- latent variable sampling
- conditional decoder
- auxiliary receptor classifier

Sequence cVAE는 GLP-1 계열 peptide의 sequence pattern과 receptor targeting 정보를 함께 학습하여, 조건부 sequence generation과 latent transition 분석을 수행합니다.

---

### 3. Bridge Model

Bridge model은 SMILES VAE와 Sequence cVAE에서 학습된 latent representation 사이의 mapping 가능성을 확인하기 위한 보조 실험입니다.

```text
Sequence latent space  ↔  SMILES latent space
```

이 모델은 sequence-level representation과 chemical structure-level representation이 서로 어느 정도 연결될 수 있는지 확인하기 위한 확장 분석입니다.

본 프로젝트의 핵심 생성 모델은 **SMILES VAE**와 **Sequence cVAE**이며, Bridge model은 두 latent space를 연결하는 보조 모듈로 다룹니다.

---

## Analysis Pipeline

전체 실험 흐름은 다음과 같습니다.

```text
Dataset preprocessing
→ SMILES VAE training
→ Sequence cVAE training
→ Latent space analysis
→ Conditional generation
→ Drug trajectory analysis
→ Interpolation analysis
→ MSA latent steering
→ AlphaFold-based structure check
```

---

## Main Analysis

본 프로젝트에서 수행한 주요 분석은 다음과 같습니다.

### SMILES-based Analysis

- SMILES / SELFIES preprocessing
- Morgan fingerprint generation
- chemical structure reconstruction
- latent space visualization
- chemical property correlation
- distance matrix analysis
- drug trajectory analysis
- interpolation analysis

### Sequence-based Analysis

- amino acid sequence preprocessing
- receptor condition encoding
- sequence reconstruction
- conditional peptide generation
- ProtParam-based property check
- PCA-based latent space visualization
- MSA latent steering
- drug trajectory analysis
- interpolation analysis

### Structure-based Check

- AlphaFold2 prediction
- pLDDT / pTM analysis
- generated sequence structure sanity check

---

## Saved Outputs

학습된 모델과 분석 결과는 `sequence/` 및 `smiles/` 폴더에 저장됩니다.

### `sequence/`

Sequence cVAE 관련 모델 checkpoint와 metadata를 저장합니다.

예시는 다음과 같습니다.

- `cvae_last.pt`
- `optimized_sequence_cvae.pt`
- `metadata.json`
- `after_data_added/`

### `smiles/`

SMILES VAE 관련 모델 checkpoint와 latent space 분석 결과 이미지를 저장합니다.

예시는 다음과 같습니다.

- `cvae_gru_best_evod.pt`
- `cvae_gru_last.pt`
- `distance_matrix_only.png`
- `pca_centroid_v4.png`
- `umap_full.png`
- `umap_milestone_full_v7.png`
- `umap_camp_pec50.png`

---

## Limitations

본 프로젝트는 GLP-1 계열 peptide drug의 representation learning과 latent space 분석을 목표로 하며, 실제 신약 후보를 검증하는 프로젝트는 아닙니다.

주요 한계는 다음과 같습니다.

- Sequence-only 모델은 fatty acid chain, linker, Aib 등 chemical modification을 직접 반영하기 어렵습니다.
- SMILES 기반 모델은 화학 구조를 다루지만 receptor binding이나 biological activity를 직접 학습하지 않습니다.
- AlphaFold 결과는 구조적 sanity check이며, 실제 약효 검증이 아닙니다.
- 희소한 receptor condition에서는 조건부 생성 성능이 제한될 수 있습니다.
- 실제 drug candidate로 평가하기 위해서는 docking, activity assay, toxicity prediction 등 추가 검증이 필요합니다.

---

## Requirements

주요 라이브러리는 다음과 같습니다.

```bash
python
torch
numpy
pandas
scikit-learn
matplotlib
biopython
rdkit
selfies
```

필요한 라이브러리는 환경에 맞게 설치한 뒤 각 notebook을 실행하면 됩니다.

```bash
pip install -r requirements.txt
```

---

## Reproducible Execution (excluding AlphaFold2)

The reproducible workflow targets **Python 3.11**. Run every command and notebook
from the repository root.

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
jupyter lab
```

Use the committed dataset snapshots for reproducible model runs:

- `dataset/SMILES.csv`
- `dataset/Sequences_additional.csv`

Recommended notebook order:

1. `SMILES_VAE.ipynb`
2. `Sequence_cVAE.ipynb`
3. `Sequence_generation_msa.ipynb`

`Dataset.ipynb` is an optional data-collection notebook. It writes to `dataset/`
but calls external ChEMBL, UniProt, Ensembl, NCBI, and RCSB services, so rerunning
it can produce different snapshots when those services change. It is not required
to reproduce the model runs above.

`Sequence_generation_msa.ipynb` reads
`sequence/sequence_cvae_checkpoint.pt`. This checkpoint is produced by the final
save cell of `Sequence_cVAE.ipynb`; a submitted checkpoint is also included so the
generation notebook can be run without retraining first.

`AlphaFold2_Eval.ipynb` is excluded from this workflow because it requires the
Google Colab/ColabFold GPU environment and external model downloads.

---
