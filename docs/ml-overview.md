---
title: 機械学習・AI 開発ガイド（実務の最短ルート）
---

<!-- LP Toast: ランディングに戻る -->
<a href="./index.html" class="lp-toast">← LPに戻る</a>
<style>
.lp-toast{position:fixed;right:16px;bottom:16px;background:#111;color:#fff;padding:10px 14px;border-radius:8px;box-shadow:0 8px 24px rgba(0,0,0,.28);text-decoration:none;font-weight:600;z-index:9999;opacity:.95}
.lp-toast:hover{opacity:1;transform:translateY(-1px)}
</style>

# 機械学習・AI 開発ガイド（実務の最短ルート）

本ガイドは、Python を用いた機械学習/深層学習の開発をプロジェクト内で円滑に進めるための要点をまとめたものです。データ前処理から学習、評価、埋め込み+ベクタDB、サービング、再現性までをひと通り俯瞰できます。

---

## 目的と前提
- 目的: 再現性のある ML 開発を素早く実装・共有する。
- 前提: Docker ベース開発（本リポジトリの compose を利用）、バックエンドは Python。
- 推奨: `poetry` または `pip + requirements.txt`、型ヒント、Lint/Format（ruff/black）。

## 主要ライブラリ（用途別）
- 数値/前処理: `numpy`, `pandas`, `scikit-learn`
- 深層学習: `torch`（PyTorch）, （必要に応じて `torchvision`, `pytorch-lightning`）
- NLP/画像モデル: `transformers`, `sentence-transformers`, `diffusers`（必要に応じて）
- ベクタDB: `qdrant-client`（Qdrant を利用）
- API/サービング: `fastapi`, `uvicorn`
- ユーティリティ: `joblib`（モデル永続化）, `pydantic`（設定）, `hydra-core`（構成管理, 任意）

---

## 開発フロー（基本）
1) データ用意: 取得、クレンジング、特徴量設計、分割（train/valid/test）
2) 学習: ベースライン → 改善（特徴量、モデル、ハイパラ）
3) 評価: 適切な指標・検証手法（KFold/StratifiedKFold 等）
4) 保存: 学習済み成果物（モデル、前処理器、メタ）をバージョン管理（ファイル名に日付/ハッシュ）
5) サービング: FastAPI などで推論 API 化（Docker で配布）
6) モニタリング: 入力分布のずれ、精度低下、レイテンシを監視（必要に応じて）

---

## スプリットと評価の基本
- データ漏洩を避けるため、スケーリング等の前処理は「学習データで fit → 予測時に transform」。
- 分類: `accuracy`, `precision/recall`, `f1`, `roc_auc`。不均衡なら ROC/AUC や PR-AUC を重視。
- 回帰: `rmse`, `mae`, `r2`。

```python
# scikit-learn: ベースライン学習例（分類）
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
import joblib

X, y = load_iris(return_X_y=True)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

clf = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(max_iter=1000))
])
clf.fit(X_tr, y_tr)
print("acc=", accuracy_score(y_te, clf.predict(X_te)))
joblib.dump(clf, "artifacts/iris_logreg.joblib")
```

---

## PyTorch（最小トレーニングループ）
```python
import torch
from torch import nn
from torch.utils.data import DataLoader, TensorDataset

# ダミーデータ（y = 2x の回帰）
X = torch.randn(1024, 1)
y = 2 * X + 0.1 * torch.randn_like(X)

ds = TensorDataset(X, y)
loader = DataLoader(ds, batch_size=32, shuffle=True)

model = nn.Sequential(nn.Linear(1, 16), nn.ReLU(), nn.Linear(16, 1))
opt = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()

for epoch in range(10):
    total = 0.0
    for xb, yb in loader:
        opt.zero_grad()
        pred = model(xb)
        loss = loss_fn(pred, yb)
        loss.backward()
        opt.step()
        total += loss.item() * xb.size(0)
    print(epoch, total / len(ds))

torch.save(model.state_dict(), "artifacts/regressor.pt")
```

- GPU を使う場合は `device = torch.device("cuda" if torch.cuda.is_available() else "cpu")` を用意し、テンソルとモデルを `.to(device)` で移動。

---

## 埋め込みとベクタ検索（Qdrant）
- テキスト/画像を高次元ベクトルに変換（埋め込み）し、Qdrant に格納・検索。
- ユースケース: 検索、レコメンド、RAG（LLMと組み合わせる検索拡張生成）。

```python
# 例: sentence-transformers で埋め込み → Qdrant へ upsert
# pip install sentence-transformers qdrant-client
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

texts = ["Hello world", "Quick brown fox", "Deep learning"]
model = SentenceTransformer("all-MiniLM-L6-v2")
vecs = model.encode(texts).tolist()

client = QdrantClient(host="localhost", port=6333)
COL = "docs"

client.recreate_collection(
    collection_name=COL,
    vectors_config=VectorParams(size=len(vecs[0]), distance=Distance.COSINE),
)

points = [PointStruct(id=i, vector=vecs[i], payload={"text": texts[i]}) for i in range(len(texts))]
client.upsert(collection_name=COL, points=points)

query = model.encode(["hello"])[0].tolist()
res = client.search(collection_name=COL, query_vector=query, limit=3)
for r in res:
    print(r.payload["text"], r.score)
```

- 本プロジェクトの compose では `qdrant` が起動します（`.env` の `QDRANT_PORT`）。

---

## モデルのサービング（FastAPI）
- 目的: 学習済みモデルを API として提供。
- 手順: 起動時にモデルをロード → 入力を受け、前処理→推論→後処理→レスポンス。

```python
# 例: scikit-learn パイプラインを FastAPI でサーブ
# pip install fastapi uvicorn joblib
from fastapi import FastAPI
import joblib
import numpy as np

app = FastAPI()
clf = joblib.load("artifacts/iris_logreg.joblib")

@app.post("/predict")
def predict(x: list[float]):
    X = np.array([x])
    y = int(clf.predict(X)[0])
    return {"pred": y}

# 起動: uvicorn app:app --host 0.0.0.0 --port 8000
```

- Docker 化: API コンテナにモデルファイル（`artifacts/`）を同梱 or ボリュームマウント。
- スキーマ: `pydantic` で入力検証を行うと安全。

---

## 再現性と実験管理
- 乱数固定: `numpy`, `torch`, `random` の seed 固定。再現性オプション（`torch.backends.cudnn.deterministic = True` 等）を検討。
- 記録: 依存バージョン（`pip freeze`）、ハイパラ、データバージョン、スコア、モデルのハッシュ/パス。
- フォルダ例: `artifacts/`（モデル/前処理器）, `logs/`, `data/`（raw/interim/processed）。
- 共有: MinIO（S3互換）へ成果物をアップロードし、メンバーで共有可能。
- 追加ツール（任意）: MLflow/W&B を導入すると実験トラッキングとモデル登録が容易。

---

## GPU と性能
- NVIDIA GPU（Linux/WSL2）: `nvidia-container-toolkit` を導入し、compose の `api` サービスに GPU を予約。
- Apple Silicon: 多くの学習は CPU/Metal で可能だが、コンテナ内の GPU は非推奨。必要に応じてリモート GPU を利用。
- 最適化の基本: バッチサイズ調整、データローダの `num_workers`、Mixed Precision（`torch.autocast`）。

---

## セキュリティ/ライセンス/倫理
- PII/機微データ: 匿名化・マスキング・アクセス制御。データは暗号化して保存。
- ライセンス: 事前学習モデル/データセットのライセンスを確認（商用可否、クレジット表記）。
- 倫理: バイアス、説明責任、誤用の防止。評価データは代表性に配慮。

---

## チェックリスト（着手時に）
- [ ] データの所在と利用許諾を確認
- [ ] ベースライン実装（単純モデルで可）
- [ ] 指標と検証方法を合意
- [ ] 成果物の保存先（artifacts/MinIO）を決定
- [ ] サービング要求（レイテンシ/スループット/SLA）を確認

---

## 参考リンク
- scikit-learn: https://scikit-learn.org/
- PyTorch: https://pytorch.org/
- Transformers: https://huggingface.co/docs/transformers
- Qdrant: https://qdrant.tech/
- FastAPI: https://fastapi.tiangolo.com/
- 実験再現性の話: https://pytorch.org/docs/stable/notes/randomness.html

このガイドを起点に、必要に応じて「データ管理」「モデルサービング設計」「RAG 設計」などの詳細ドキュメントを追加していきましょう。
