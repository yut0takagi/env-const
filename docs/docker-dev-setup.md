# Dockerベース開発環境ガイド（Web/デスクトップ/ネイティブ + Python/AI バックエンド）

本ドキュメントは、Docker を用いて Web フロントエンド、デスクトップアプリ、ネイティブ（モバイル）アプリの開発を行い、AI を多用する Python バックエンドを支えるための、ローカル開発環境の標準構築手順をまとめたものです。

## 目的と範囲
- 目的: コンテナ化された統一開発環境の提供（オンボーディングの高速化、差分吸収、再現性の確保）
- 範囲: ローカル開発（dev）。本番（prod）/ステージング（stg）は将来の拡張を前提にプロファイルで分離します。

## 前提条件（Prerequisites）
- OS: macOS 13+/Windows 11（WSL2）/Ubuntu 22.04+
- Docker: Docker Desktop（macOS/Windows）または Docker Engine（Linux）
- Docker Compose v2（`docker compose` コマンド）
- Git 2.40+
- 推奨: Make, direnv, VS Code + Dev Containers 拡張
- GPU（任意/AI推論高速化）:
  - NVIDIA（Linux/WSL2）: NVIDIA Driver + CUDA + `nvidia-container-toolkit`
  - Apple Silicon（M1/M2/M3）: コンテナ内GPUは非推奨。CPU実行またはリモートGPU利用を推奨

## リポジトリ構成（推奨）
```
repo-root/
  backend/           # Python (FastAPIなど) + AIコード
  web/               # Webフロント（Next.js/Vite等）
  desktop/           # デスクトップ（Electron等）
  native/            # モバイル（React Native等: エミュレータはホスト依存）
  infra/
    docker/          # Dockerfile郡（api, web, tools ...）
    compose/         # compose.*.yml（dev/stg/prodプロファイル）
  doc/               # 本ドキュメント他
  .env               # 共有環境変数（機微情報は含めない）
  .env.local         # 個人用上書き（gitignore）
```

## 環境変数（.env）
- 原則 `.env` に共通・非機微値、個人差分は `.env.local` に記載。
- 機微情報は「ローカルのみ」もしくは「Docker secrets/1Password/Vault」等で管理し、Gitに含めない。

`.env.example`（プロジェクト直下に配置推奨）:
```
# 共通
PROJECT_NAME=env-const
COMPOSE_PROJECT_NAME=${PROJECT_NAME}
TZ=Asia/Tokyo

# ポート
API_PORT=8080
WEB_PORT=3000
REDIS_PORT=6379
POSTGRES_PORT=5432
MINIO_CONSOLE_PORT=9001
MINIO_API_PORT=9000
QDRANT_PORT=6333

# DB
POSTGRES_DB=app
POSTGRES_USER=app
POSTGRES_PASSWORD=app_password # 本番NG

# AI/推論
PYTORCH_DEVICE=cpu  # linux + nvidia: cuda

# ストレージ
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin # 本番NG

# 開発向け
UVICORN_RELOAD=true
PYTHONUNBUFFERED=1
NODE_ENV=development
```

初回は以下でコピーして調整してください:
```
cp .env.example .env
cp .env.example .env.local  # 個人差分があればこちらで上書き
```

## Compose 構成（例）
以下は開発用のサンプルです。実ファイルは `infra/compose/compose.dev.yml` などに配置することを推奨します。

```yaml
# infra/compose/compose.dev.yml
name: ${COMPOSE_PROJECT_NAME}
services:
  api:
    build:
      context: ../..
      dockerfile: infra/docker/api.Dockerfile
    image: ${COMPOSE_PROJECT_NAME}-api:dev
    container_name: ${COMPOSE_PROJECT_NAME}-api
    env_file:
      - ../../.env
      - ../../.env.local
    environment:
      - PYTORCH_DEVICE=${PYTORCH_DEVICE}
    ports:
      - "${API_PORT}:8000"
    volumes:
      - ../../backend:/app:cached
    command: bash -lc "uvicorn app.main:app --host 0.0.0.0 --port 8000 ${UVICORN_RELOAD:+--reload}"
    depends_on:
      - db
      - redis
      - qdrant
      - minio
    profiles: [dev, api]

  web:
    build:
      context: ../..
      dockerfile: infra/docker/web.Dockerfile
    image: ${COMPOSE_PROJECT_NAME}-web:dev
    container_name: ${COMPOSE_PROJECT_NAME}-web
    env_file:
      - ../../.env
      - ../../.env.local
    ports:
      - "${WEB_PORT}:3000"
    volumes:
      - ../../web:/workspace:cached
    working_dir: /workspace
    command: bash -lc "npm ci && npm run dev"
    depends_on:
      - api
    profiles: [dev, web]

  # デスクトップ/Electron: バンドラ/ビルド用（GUIはホストで実行）
  desktop:
    build:
      context: ../..
      dockerfile: infra/docker/desktop.Dockerfile
    image: ${COMPOSE_PROJECT_NAME}-desktop:dev
    container_name: ${COMPOSE_PROJECT_NAME}-desktop
    env_file:
      - ../../.env
      - ../../.env.local
    volumes:
      - ../../desktop:/workspace:cached
    working_dir: /workspace
    command: bash -lc "npm ci && npm run build:watch"
    profiles: [dev, desktop]

  # ネイティブ/React Native: Metroバンドラのみ（エミュレータはホスト依存）
  native:
    build:
      context: ../..
      dockerfile: infra/docker/native.Dockerfile
    image: ${COMPOSE_PROJECT_NAME}-native:dev
    container_name: ${COMPOSE_PROJECT_NAME}-native
    env_file:
      - ../../.env
      - ../../.env.local
    volumes:
      - ../../native:/workspace:cached
    working_dir: /workspace
    command: bash -lc "npm ci && npm run start"
    profiles: [dev, native]

  db:
    image: postgres:16
    container_name: ${COMPOSE_PROJECT_NAME}-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    profiles: [dev, infra]

  redis:
    image: redis:7
    container_name: ${COMPOSE_PROJECT_NAME}-redis
    ports:
      - "${REDIS_PORT}:6379"
    volumes:
      - redis_data:/data
    profiles: [dev, infra]

  qdrant:
    image: qdrant/qdrant:latest
    container_name: ${COMPOSE_PROJECT_NAME}-qdrant
    ports:
      - "${QDRANT_PORT}:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    profiles: [dev, infra]

  minio:
    image: minio/minio:latest
    container_name: ${COMPOSE_PROJECT_NAME}-minio
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    command: server /data --console-address ":9001"
    ports:
      - "${MINIO_API_PORT}:9000"
      - "${MINIO_CONSOLE_PORT}:9001"
    volumes:
      - minio_data:/data
    profiles: [dev, infra]

volumes:
  db_data:
  redis_data:
  qdrant_data:
  minio_data:
```

### 参考 Dockerfile（例）
`infra/docker/api.Dockerfile`（Python + FastAPI + Uvicorn + dev便利ツール）
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11-slim

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
 && apt-get install -y --no-install-recommends build-essential git \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app
# 依存レイヤを先に固める
COPY backend/requirements.txt /tmp/requirements.txt
RUN pip install -U pip \
 && pip install -r /tmp/requirements.txt

# アプリ本体はマウント前提（開発）
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
```

`infra/docker/web.Dockerfile`（Nodeベース）
```dockerfile
FROM node:20-bullseye
WORKDIR /workspace
EXPOSE 3000
```

`infra/docker/desktop.Dockerfile` / `infra/docker/native.Dockerfile` は web と同様の Node ベースで開始し、必要に応じて SDK/CLI を追加してください。

## 初回セットアップ手順
1) 依存インストール
- Docker Desktop/Engine と Docker Compose v2 をインストール
- Windows は WSL2 を有効化。NVIDIA GPU を使う場合は `nvidia-container-toolkit` をセットアップ（Linux/WSL2）

2) .env 用意
```
cp .env.example .env
cp .env.example .env.local  # 個人差分は .env.local に
```

3) 開発スタック起動（dev プロファイル）
```
# 例: infra/compose/ 配下で
docker compose -f compose.dev.yml --profile dev up -d --build
```

4) 動作確認
- API: http://localhost:${API_PORT}/docs （FastAPI の Swagger UI 想定）
- Web: http://localhost:${WEB_PORT}
- MinIO Console: http://localhost:${MINIO_CONSOLE_PORT}
- Qdrant: http://localhost:${QDRANT_PORT}
- Postgres: `localhost:${POSTGRES_PORT}`

## よく使うコマンド
- 全サービス起動: `docker compose --profile dev up -d`
- 再ビルド: `docker compose build api web`
- ログ表示: `docker compose logs -f api`
- 任意サービスに入る: `docker compose exec api bash`
- 片付け: `docker compose down`（ボリューム保持） / `docker compose down -v`（データ削除）

## GPU 利用（オプション）
- Linux/WSL2 + NVIDIA：`nvidia-container-toolkit` を導入後、`compose.dev.yml` の `api` に以下を追加
```yaml
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
```
- Apple Silicon: コンテナ内GPU加速は基本不可。CPU実行またはリモートGPU（SSH/VPN + 共有DB/MinIO）を推奨。

## 開発ワークフローのヒント
- バックエンド: `uvicorn --reload` でホットリロード、テストは `pytest`。モデル/重量物はボリュームやローカルキャッシュを活用。
- Web: `npm run dev`。API ベースURLは `.env` 管理。
- デスクトップ: バンドラ/ビルドはコンテナ、実行確認はホストで（Electron の GUI はホスト側）。
- ネイティブ: Metro はコンテナで、エミュレータ/デバイスはホストで操作。
- Lint/Format: pre-commit や ESLint/Black/isort を導入し、コンテナ内で共通実行。

## データ永続化とリセット
- データは Docker ボリューム（`db_data` など）に永続化。
- 全削除: `docker compose down -v`（注意: ローカルデータが消えます）
- 個別削除: `docker volume rm <volume_name>`

## トラブルシュート
- ポート競合: `.env` のポートを変更、または競合プロセスを停止。
- ファイル権限: Windows/WSL2 や Mac のボリュームマウントで権限差。`git config core.autocrlf false` や 実行権限を確認。
- 速度問題: ボリュームに `:cached`/`:delegated`、不要なサービスを未起動（プロファイル活用）。
- 画像サイズ: 依存レイヤ固定・ビルドキャッシュ・`pip install --no-cache-dir` 等で最適化。

## セキュリティ注意
- `.env` に機微情報を置かない（`.env.local` でもリスクを理解し最小限に）。
- 本番は `compose.prod.yml` を別途用意し、Secrets/Key 管理を徹底。
- 依存イメージの定期更新と脆弱性スキャンを実施。

## 次のアクション
- `infra/docker/` に各 Dockerfile を追加
- `infra/compose/compose.dev.yml` を作成し、上記サンプルを反映
- 各アプリ（backend/web/desktop/native）の最小テンプレートを用意
- VS Code Dev Containers を使う場合は `.devcontainer/` を追加

---
このガイドをプロジェクト標準として配布し、オンボーディング時は本ドキュメントのみで環境が再現できる状態を維持してください。
