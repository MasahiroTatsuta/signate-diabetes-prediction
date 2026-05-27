# 🏥 糖尿病発症予測 — SIGNATE 初心者向けコンペティション

> 医療診断データから糖尿病の発症を予測する**二値分類**。
> このコンペティションは、SIGNATEで**Intermediateランク**を達成するための足がかりとして活用されました。

> **注：** このリポジトリは実験の過程を記録したものです。
> 記載されているスコアは、コンペティション期間中の最高成績を示しています。
> 環境やシードの違いにより、正確な再現性は異なる場合があります。

---

## コンテスト情報

| 項目 | 詳細 |
|---|---|
| プラットフォーム | [SIGNATE – 診断データを使った糖尿病発症予測](https://signate.jp/competitions/204) |
| タスク | 二値分類（糖尿病：あり / なし） |
| 評価指標 | AUC-ROC |
| 成績 | 🏅 **Intermediateランクに昇格** |

---

## 技術的ハイライト

### 問題の概要
8つの医療特徴量（血糖値、BMI、血圧など）から糖尿病の発症を予測する。  
クラス不均衡とゼロ値が偏った特徴量を含む、古典的な表形式の二値分類。

### ソリューションパイプライン

```
生データ
   ↓
[前処理]
   ├── 外れ値の除去 (血圧 < 20, BMI < 10)
   └── 値がゼロの医療特徴量に対するKNN補完
   ↓
[特徴量エンジニアリング]
   ├── AutoFeatによる多項式クロス特徴量 (feateng_steps=2)
   ├── 相関に基づく特徴量選択 (ターゲットとの |r| > 0.01)
   └── ドメインフラグ (High_Glucose, Young_Pregnant, Old_Obese など)
   ↓
[モデリング]
   ├── LightGBM + Optuna (メイン)
   └── PyCaret AutoML (比較用ベースライン)
   ↓
[閾値最適化]
   └── AUC-ROC に基づく最適閾値の探索
```

### 主要な実験

| 手法 | CV AUC |
|---|---|
| LightGBM ベースライン | 0.768 |
| + KNN補完 | 0.779 |
| + AutoFeat特徴量 | 0.791 |
| + Optunaチューニング + 閾値最適化 | 0.797 |
| PyCaret AutoML | 0.782 |

---

## リポジトリ構造

```
signate-diabetes-prediction/
├── notebooks/
│   ├── 01_pipeline.ipynb           ← メイン: EDA + 特徴量エンジニアリング + LightGBM + Optuna
│   └── 02_pipeline_pycaret.ipynb   ← 比較: PyCaret AutoML ベースライン
├── data/
│   ├── train.csv          ← 生トレーニングデータ (SIGNATE より)
│   ├── test.csv           ← テスト用生データ
│   └── sample_submit.csv  ← 提出用フォーマット
├── outputs/
│   ├── submission_final.csv    ← 最終提出データ
│   └── feature_importance.csv  ← LightGBMの特徴量重要度
├── requirements.txt
└── .gitignore
```

---

## クイックスタート

```bash
git clone https://github.com/MasahiroTatsuta/signate-diabetes-prediction
cd signate-diabetes-prediction
pip install -r requirements.txt

# コンテストデータを data/ ディレクトリに配置（SIGNATE からダウンロード）
# その後、Jupyter で notebooks/01_pipeline.ipynb を開く
jupyter notebook notebooks/01_pipeline.ipynb
```

---

## 環境

| ライブラリ | バージョン |
|---|---|
| Python | 3.10+ |
| LightGBM | 4.0 以上 |
| Optuna | 3.5 以上 |
| scikit-learn | 1.3 以上 |
| autofeat | 2.1 以上 |
| pycaret | 3.3 以上 |

完全なリストについては、[`requirements.txt`](requirements.txt)を参照してください。

---

## 学んだこと

- 医療データにおける**ゼロインフレーション**（例：`Insulin=0`はゼロではなく欠損を意味する）には、ドメインを意識した補完が必要である — ゼロ以外のサブセットに対するKNN補完により、AUCが約0.008向上した
- **AutoFeat**は約300個の多項式特徴量を生成したが、相関フィルタリングにより有用な特徴量は約40個に絞り込まれた
- PyCaretは高速なベースラインを提供したが、LightGBMとOptunaによるチューニングが常にそれを上回った
- AUC-ROCに基づく閾値の最適化により、わずかではあるが一貫した改善（約0.003）が見られた


DeepL.com（無料版）で翻訳しました。
